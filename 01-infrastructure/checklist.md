# 01 — Infrastructure & DevOps — Checklist

Sprint-ready. Tick items off as you go. The checklist is shape-agnostic — pick
your shape (Single VPS / PaaS / Serverless) in the first section, then follow
the relevant items.

## Shape decision

- [ ] Decided on a shape (VPS / PaaS / Serverless). See README's decision tree.
- [ ] Estimated monthly cost at expected month-3 traffic. Wrote it down.
- [ ] Identified the *exit door*: if this shape is wrong in 6 months, what's
      the migration plan? (Single VPS → PaaS is easy; PaaS → VPS is medium;
      Serverless → anything else can be hard.)

## Domain & DNS

- [ ] Bought the domain. (Namecheap, Cloudflare Registrar at-cost, Porkbun.)
- [ ] DNS hosted on Cloudflare (free tier is fine).
- [ ] **A record** at `@` → server IPv4.
- [ ] **A record** at `*` → server IPv4 (wildcard for tenant subdomains;
      skip if you don't do per-tenant subdomains).
- [ ] **AAAA record** matching, if you have IPv6.
- [ ] **Cloudflare proxy: OFF** (grey cloud) on records you'll terminate TLS
      for via Caddy/Let's Encrypt with DNS-01. Otherwise the cert challenge
      breaks.
- [ ] DNS TTL set to 300s during setup, raise to 3600s once stable.
- [ ] **MX records** delegated to your transactional email provider (Resend,
      Postmark, SendGrid). See topic 04.
- [ ] **TXT records:** SPF, DKIM (one per email service), DMARC at minimum
      `v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com` to start, tighten
      later.

## TLS

### If using a PaaS (Vercel / Fly / Railway / Render)

- [ ] Followed the platform's "add custom domain" flow.
- [ ] Verified TLS works at both apex (`example.com`) and `www`.
- [ ] If multi-tenant via subdomain: confirmed wildcard SSL is supported on
      your tier (Vercel needs Pro for wildcard; Fly supports it via per-cert).

### If self-managed (Caddy)

- [ ] Installed Caddy with required DNS plugin (Cloudflare in our reference
      setup) using `xcaddy build --with github.com/caddy-dns/cloudflare`.
- [ ] **Stored API token in `/etc/caddy/env` with `chmod 600`**, not in the
      Caddyfile.
- [ ] Created a systemd drop-in at
      `/etc/systemd/system/caddy.service.d/override.conf` with
      `EnvironmentFile=/etc/caddy/env`.
- [ ] Verified `systemctl show caddy | grep EnvironmentFile` reports the file.
- [ ] Caddyfile has `tls { dns cloudflare {env.CLOUDFLARE_API_TOKEN} }` in
      **both** the apex and the `*.example.com` blocks (forgetting one
      breaks wildcard).
- [ ] First reload's logs show
      `"msg":"certificate obtained successfully","identifier":"*.example.com"`.

## Server provisioning (VPS)

- [ ] Provider chosen. Hetzner CX22 / Vultr Cloud Compute / Hostinger VPS at
      the €5/m tier is enough for first ~200 active users on a typical CRUD
      SaaS.
- [ ] Provisioned SSH-key-only access (no password login) at order time.
- [ ] OS: Ubuntu 24.04 LTS or Debian 12. Stick to LTS.
- [ ] First SSH succeeded.
- [ ] `apt update && apt upgrade -y` run. Reboot if `/var/run/reboot-required`.
- [ ] Installed `curl git build-essential file ufw fail2ban`.
- [ ] **UFW configured:** allowed 22, 80, 443; enabled. Verified with
      `ufw status`.
- [ ] **fail2ban configured** with `[sshd]` jail using `backend = systemd`
      (Ubuntu 24+ requires this — default `auto` won't find logs).
- [ ] **Swap exists** (Hetzner auto-creates 2GB; verify `free -h`).
- [ ] Created a non-root deploy user with sudo (or kept root with key-only
      auth — small-team trade-off, document the choice).

## Runtime

- [ ] Node 20 LTS installed via NodeSource (or 22 LTS on first deploy after
      April 2026).
- [ ] `pnpm` installed globally. **Don't pin a version in CI** if your
      `package.json` already has `packageManager: pnpm@x.y.z`. Both will
      conflict; let `package.json` win.
- [ ] PM2 installed if you're not using a process manager from systemd. PM2
      `ecosystem.config.cjs` reads `.env` from the app dir.
- [ ] Confirmed `node --version`, `pnpm --version`, `pm2 --version`.

## Source control & deploy

- [ ] Repo created on GitHub (private if pre-launch, public for OSS).
- [ ] `.env.example` committed. Real `.env` ignored. Verified `.gitignore`.
- [ ] Branch protection on `main`: require PR, status checks if any.
- [ ] **Deploy SSH key:** generated a deploy-only SSH key pair, public key
      added to the VPS's `~/.ssh/authorized_keys`, private key stored as
      `VPS_SSH_KEY` GitHub Actions secret.
- [ ] GitHub Actions workflow at `.github/workflows/deploy.yml` that on push
      to `main` SSHes into the VPS, fetches, installs, builds, reloads PM2.
- [ ] `git fetch origin main && git reset --hard origin/main` is used (not
      `git pull`) to avoid divergence if someone patched on the server.
- [ ] First successful deploy verified end-to-end. Time-boxed: under 3 minutes.
- [ ] Rollback strategy tested: `git reset --hard <prev-commit> && pnpm
      build && pm2 reload`. Wrote it on a sticky note.

## Monorepo (if applicable)

- [ ] `pnpm-workspace.yaml` lists the workspace globs (`apps/*`, `packages/*`).
- [ ] Each `package.json` has `"name": "@yourorg/foo"`, `"private": true`.
- [ ] Turbo or Nx configured for `dev` / `build` / `typecheck` / `lint` —
      Turbo's `turbo.json` is enough for most setups.
- [ ] `packages/db` exports typed DB clients used by both `apps/app` and
      `apps/landing`.
- [ ] `packages/billing` (or your domain package) holds provider-agnostic
      logic; concrete providers are in subfiles. See topic 03.
- [ ] Imports use the workspace name (`import { x } from '@yourorg/db'`),
      not relative `../../packages/db`.

## Environments

- [ ] Three environments named: **local**, **staging** (optional but
      recommended), **production**.
- [ ] Per-env `.env` files. Single source of truth: a password manager
      vault (1Password / Bitwarden) for the team copy.
- [ ] `PUBLIC_*` prefix (or framework equivalent) for client-bundled vars.
      Anything sensitive must NOT have the public prefix.
- [ ] Staging uses *test mode* of every paid service: Stripe test, Resend
      test mode, OAuth provider test app. Document each toggle.
- [ ] Production secrets rotated quarterly (calendar reminder).

## Reverse proxy & static assets

- [ ] Marketing site assets (Astro `dist/`, Next.js `out/`, etc.) are served
      directly by the proxy with long cache headers
      (`Cache-Control: public, max-age=31536000, immutable`) for hashed asset
      paths.
- [ ] App routes (`/api/*`, `/login*`, `/_app/*`, etc.) are proxied to the
      app process.
- [ ] **404 fallback**: a non-existent path returns `404`, not the home page
      with status `200`. (Common Astro / `try_files` bug — see gotchas.)
- [ ] Trailing-slash redirect rule: `/foo/` → `/foo` 301, OR `/foo` → `/foo/`
      301. Pick one and stick to it.
- [ ] Sitemap path served (`/sitemap.xml`). See topic 17.

## Observability hooks (just enough)

> Full content lives in topic **13 — Observability**. The minimum to set up
> *now*:

- [ ] Sentry (or equivalent) DSN added; basic uncaught-error capture verified
      with a test error in production.
- [ ] Uptime check (UptimeRobot free tier, BetterStack, or Cronitor) hitting
      `/healthz` every 1–5 minutes from at least two regions.
- [ ] `/healthz` returns 200 only if DB is reachable.
- [ ] Slack/Discord webhook for deploy failures and uptime alerts. Phone
      notifications enabled.

## Backups & recovery (just enough)

> Full content lives in topic **14 — Data Operations**. The minimum:

- [ ] Database backups are on (managed Postgres: usually default; verify the
      retention period).
- [ ] Wrote down (in `internal-docs/RUNBOOK.md`) how to restore a backup
      to a fresh DB.
- [ ] **Did the restore at least once.** A backup you've never restored is a
      Schrödinger's backup.

## Cost control

- [ ] Billing alert set on the cloud provider (Hetzner: project budget;
      Vercel: spend cap; AWS: AWS Budgets).
- [ ] Domain auto-renew on. Calendar reminder one month before expiration.
- [ ] Wrote down (in `internal-docs/COSTS.md`) the per-service monthly
      bill. Review quarterly.

## Final smoke test

- [ ] HTTPS works on apex, `www`, and a tenant subdomain.
- [ ] `curl -sSI https://example.com` shows HTTP/2 200 and a sane
      `cache-control`.
- [ ] `curl -sSI https://example.com/route-that-does-not-exist` returns 404.
- [ ] `pm2 status` shows the app `online` with stable RSS over 10 minutes.
- [ ] Deployed twice in a row from CI without manual intervention.
- [ ] Tested rollback by deploying a deliberately broken commit, verifying
      the failure, then rolling back. (Optional but recommended once.)
