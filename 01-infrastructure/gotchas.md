# 01 — Infrastructure & DevOps — Gotchas

Real production failures, with the symptom, root cause, and fix. Most of
these come from the Grabit (`bookolj`) project's setup logs, post-mortems,
and Slack history. Some are from the wider playbook author network — credited
where the source is public.

## G-01 Wildcard SSL silently falls back to per-host issuance

**Symptom.** TLS works fine on the apex domain. Tenant subdomain
`tenant.example.com` says `ERR_SSL_PROTOCOL_ERROR` in Chrome. `caddy
list-modules` shows the cloudflare DNS provider is loaded.

**Root cause.** The Caddyfile has the `tls { dns cloudflare ... }` block
**inside the apex `example.com {}` block**, but the `*.example.com {}` block
inherits no TLS configuration and Caddy quietly tries HTTP-01 — which fails
because each subdomain isn't reachable individually.

**Fix.** Repeat the `tls { dns cloudflare ... }` block in **every** site
block that needs it. There is no inheritance.

```caddy
example.com {
  tls { dns cloudflare {env.CLOUDFLARE_API_TOKEN} }
  # ...
}

*.example.com {
  tls { dns cloudflare {env.CLOUDFLARE_API_TOKEN} }   # ← this is the fix
  reverse_proxy localhost:3000
}
```

## G-02 Cloudflare proxy ON breaks cert issuance

**Symptom.** Caddy logs `obtaining certificate` repeatedly, then "challenge
failed: timeout". Site is unreachable on HTTPS.

**Root cause.** Cloudflare orange-cloud (proxy ON) intercepts the HTTP-01
challenge before it reaches your origin. For DNS-01, the proxy doesn't
matter, but if the API token is missing/wrong Caddy falls back to HTTP-01
silently.

**Fix.** Either turn the orange cloud OFF on `@` and `*` records and let
Caddy handle TLS, or leave it ON and let Cloudflare terminate TLS (Universal
SSL) with origin running on HTTP. Don't try to do both. The Grabit setup
chose proxy OFF because Caddy gives finer control over headers.

## G-03 fail2ban does nothing on Ubuntu 24

**Symptom.** Installed fail2ban, default config; `journalctl` still shows
thousands of failed SSH attempts daily. `fail2ban-client status sshd`
shows `Currently failed: 0` even though attacks are visible in
`journalctl -u ssh`.

**Root cause.** Ubuntu 24's sshd logs to `journald`, not `/var/log/auth.log`
(which doesn't exist anymore). Default fail2ban `backend = auto` looks for
the log file, doesn't find it, and silently does nothing.

**Fix.** Add to `/etc/fail2ban/jail.local`:

```ini
[sshd]
enabled  = true
backend  = systemd       # ← this is the fix
port     = ssh
maxretry = 5
bantime  = 30m
```

Reload `fail2ban-client reload`. Verify with `fail2ban-client status sshd`
— within minutes you should see `Currently banned: N`. The Grabit log
recorded ~150 unique IPs banned within the first hour after this fix.

## G-04 Astro `try_files` with index.html fallback returns 200 for unknown URLs

**Symptom.** Customer reports: `https://example.com/blablabla` shows the
home page instead of a 404. SEO impact: Google indexes hundreds of "soft
404" duplicate pages, organic traffic drops.

**Root cause.** Astro built with `build.format: 'file'` produces
`dist/v2.html`, not `dist/v2/index.html`. The Caddyfile had
`try_files {path} {path}.html {path}/index.html /index.html` — the final
`/index.html` fallback meant any nonsense URL served the home page with
HTTP 200.

**Fix.** Drop the `/index.html` fallback. Use `handle_errors` to return a
proper 404:

```caddy
handle {
  root * /var/www/example/apps/landing/dist
  try_files {path} {path}.html {path}/index.html
  file_server
}

handle_errors {
  @404 expression {err.status_code} == 404
  handle @404 {
    root * /var/www/example/apps/landing/dist
    rewrite * /404.html
    file_server
  }
}
```

Test after deploy:

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://example.com/blablabla
# → 404, not 200
```

## G-05 GitHub Actions pnpm action conflicts with `packageManager` pin

**Symptom.** CI fails with `ERR_PNPM_BAD_PM_VERSION  Currently active pnpm
version: 8.x.x, but package.json field 'packageManager' specifies 10.33.0`.

**Root cause.** The `pnpm/action-setup@v4` action with an explicit `version:`
field installs that version, but `corepack` later activates the version from
`package.json`, and they fight.

**Fix.** Don't specify `version:` in the action config. With Node 20+ and
corepack, the action picks the version from `package.json` automatically.

```yaml
- uses: pnpm/action-setup@v4
  # No 'with: version:' — let package.json drive
```

## G-06 PM2 reloads stale env when `.env` changes

**Symptom.** Updated `.env`, ran `pm2 reload grabit-app`, but the process
still uses the old `RESEND_API_KEY`.

**Root cause.** `pm2 reload` only re-execs the JS process; it doesn't
re-read environment files. PM2 caches the env from the last `start`.

**Fix.** Use `pm2 startOrReload ecosystem.config.cjs --update-env`. The
`--update-env` flag forces PM2 to re-source the env. The Grabit
`deploy.yml` does this on every deploy.

```yaml
- run: pm2 startOrReload apps/app/ecosystem.config.cjs --update-env
```

## G-07 `git pull` on the server creates merge conflicts forever

**Symptom.** Deploy fails with `Your local changes to the following files
would be overwritten by merge`. Someone has `git commit`-ed a hot-fix on
the server.

**Root cause.** The deploy script used `git pull`. Any divergence (e.g. a
manual `chmod`, a stray `pm2 logs > /var/log/...` redirect logged into a
tracked file, or a deliberate hotfix) blocks the merge.

**Fix.** Replace with `git fetch origin main && git reset --hard
origin/main`. This is destructive on purpose: the deploy is the source of
truth; the server has no committable state.

Document the hotfix process: SSH on, fix the file, *commit and push* via a
real branch, then re-deploy. Never edit-and-leave on a deployed server.

## G-08 `apt upgrade` hangs the deploy because of an interactive prompt

**Symptom.** The hourly cron-driven `apt upgrade` step on the server hung
overnight because a service's config file changed and dpkg asked
"keep the local version or use the package maintainer's?"

**Root cause.** Default `apt upgrade` is interactive on conffile changes.

**Fix.** Use the non-interactive form for any scripted/cron upgrade:

```bash
export DEBIAN_FRONTEND=noninteractive
apt-get update -qq
apt-get upgrade -y -qq \
  -o Dpkg::Options::='--force-confdef' \
  -o Dpkg::Options::='--force-confold'
```

`--force-confold` keeps your edits. `--force-confdef` accepts the package
default if there's no conflict. After upgrade:

```bash
test -f /var/run/reboot-required && echo 'REBOOT NEEDED'
needrestart -b   # which services need restart?
```

Plan a maintenance window once a month for kernel reboots.

## G-09 Cloudflare DNS API token scoped too broadly

**Symptom.** Security review flag: the Cloudflare API token in
`/etc/caddy/env` has `Edit zone DNS` for *all* zones in the account.

**Root cause.** Created the token via the "Edit zone DNS" template without
restricting Zone Resources.

**Fix.** Recreate the token with Zone Resources `Include → Specific zone →
yourdomain.com` only. Replace `/etc/caddy/env`. Restart Caddy. Revoke the
old token.

## G-10 Vercel bandwidth bill explodes after a marketing campaign

**Symptom.** Vercel monthly bill jumps from $20 to $1,400 the month after
a viral Reddit post.

**Root cause.** Vercel's bandwidth pricing on Hobby/Pro tiers is generous
for normal traffic but punitive for spikes. Each MB of static asset served
counts.

**Fix in advance:**
- Set Vercel's [Spend Cap](https://vercel.com/docs/pricing/manage-and-optimize-spend).
- Put Cloudflare (free tier) in front of Vercel as a CDN. Cloudflare's
  egress is free; Vercel sees only origin pulls.
- Move large assets (images, video) to Cloudflare R2 or BackBlaze B2 with
  Cloudflare CDN — egress is free or near-free.

**Postmortem fix:** open a Vercel support ticket; in 2024–25 Vercel has
discretion to refund obvious surprises. Then implement the prevention.

This is the most common solo-founder cost incident; protect against it on
day 1.

## G-11 `fallocate` fails on Hetzner with "Text file busy"

**Symptom.** Following a generic "set up swap" guide, you run
`fallocate -l 2G /swapfile` on a fresh Hetzner CX22 and get
`fallocate: cannot open /swapfile: Text file busy`.

**Root cause.** Hetzner CX22 already has a 2GB swap configured by default.
You can't `fallocate` over an active swap file.

**Fix.** Verify with `free -h` first. If swap exists, skip the step. The
Grabit VPS doc has this annotated; it's a 5-minute confusion many founders hit.

## G-12 Reverse proxy serves stale build because Caddy + symlinks

**Symptom.** Deployed a new Astro `dist/`. Caddy still serves the old build.
Clearing browser cache doesn't help. `ls -la` on the dist folder shows the
new files.

**Root cause.** Caddy uses an internal stat cache for `file_server`. After
a `git reset --hard` + rebuild, mtimes can be very close, and Caddy may
have the old file descriptor.

**Fix.** Reload Caddy after every deploy:

```bash
systemctl reload caddy
```

Add this to the deploy script after `pm2 reload`. Reload is graceful (no
downtime). The Grabit `deploy.yml` was missing this for the first month;
the fix was a one-line addition.

## G-13 Multiple deploys race on the same VPS

**Symptom.** Two PRs merged within 30 seconds of each other. Both
GitHub Actions runs fire. The second `git fetch && reset --hard` overwrites
the first mid-build. PM2 reloads pick up a half-built `apps/app/build/`.
Site 502s for ~3 minutes.

**Root cause.** No concurrency guard.

**Fix.** Add to `.github/workflows/deploy.yml`:

```yaml
concurrency:
  group: deploy-prod
  cancel-in-progress: false   # let in-flight deploy finish
```

This ensures only one deploy runs at a time; the second waits.

## G-14 SvelteKit `node-adapter` build fails on the VPS but works locally

**Symptom.** Local `pnpm build` succeeds. On the server, the build fails
with `JavaScript heap out of memory` or `gyp ERR! stack Error: not found:
make`.

**Root cause.** Two flavours:
1. **Memory:** the CX22 has 4GB RAM, and Vite's build can OOM on monorepo
   builds with many sourcemaps.
2. **Native compilation:** `bcrypt`, `sharp`, or `better-sqlite3` need
   `build-essential` (gcc, make) on the server.

**Fix.**
- For memory: `NODE_OPTIONS=--max-old-space-size=3072 pnpm build`. Or build
  in CI (GitHub Actions runners have 16GB) and `rsync` the artifacts.
- For native: `apt install -y build-essential file` was added to the
  Grabit VPS-SETUP doc as the first apt step.

## G-15 GitHub Actions deploy uses an old SSH key after secret rotation

**Symptom.** Rotated the deploy SSH key. Deploy fails with `Permission
denied (publickey)`. Pasted the new private key into `VPS_SSH_KEY` secret;
still fails.

**Root cause.** The new public key was never added to the VPS's
`~/.ssh/authorized_keys`. Or, the old key wasn't removed.

**Fix.** Rotation procedure:
1. Generate new keypair locally.
2. SSH into VPS with the *old* key, append the new public key to
   `authorized_keys`.
3. Update `VPS_SSH_KEY` GitHub Actions secret with new private key.
4. Trigger a deploy. Verify it works.
5. *Then* remove the old public key from `authorized_keys`.

In that order. The "verify before remove" step has saved more than one
locked-out account.
