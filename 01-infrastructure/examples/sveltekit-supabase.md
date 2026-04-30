# 01 — Infrastructure — Example: SvelteKit + Supabase + Hetzner + Caddy

The Grabit reference setup, end to end. Production-grade, ~€5/month.

This is a transcript of the actual setup that has run Grabit in production
since 2026-04. Every config is real; gotchas are cross-referenced to
[`gotchas.md`](../gotchas.md).

---

## 1. Stack overview

| Layer            | Choice                              | Why                                                    |
|------------------|-------------------------------------|--------------------------------------------------------|
| App framework    | SvelteKit (`@sveltejs/adapter-node`) | SSR + form actions + great DX                         |
| Marketing site   | Astro 5 (`build.format: 'file'`)    | SSG, fast build, minimal hydration                     |
| Database + Auth  | Supabase (managed Postgres + Auth)  | Free tier carries the first ~10k MAU                   |
| Email            | Resend                              | Cheap, fast, good DX, verified DKIM via Resend domain |
| File hosting     | Supabase Storage                    | Bundled with the DB; one auth context                  |
| Reverse proxy    | Caddy 2.x (xcaddy + cloudflare DNS) | Wildcard SSL via DNS-01                                |
| Process manager  | PM2 with `ecosystem.config.cjs`     | `startOrReload` is graceful, plays nice with deploys   |
| Provider         | Hetzner Cloud CX22                  | €5/m for 2 vCPU / 4 GB / 40 GB                         |
| OS               | Ubuntu 24.04 LTS                    | Long support window, stable kernel                     |
| DNS              | Cloudflare (proxy OFF)              | Free DNS, fast propagation, API for Caddy DNS-01      |
| CI/CD            | GitHub Actions → SSH deploy         | 90s deploys, no proprietary tooling                    |
| Monorepo         | pnpm workspaces + Turbo             | Shared `db` and `billing` packages                     |

---

## 2. Repository layout

```
bookolj/
├── apps/
│   ├── app/                  # SvelteKit product app
│   │   ├── src/
│   │   ├── ecosystem.config.cjs
│   │   ├── package.json      # name: @bookly/app
│   │   └── .env              # not committed
│   └── landing/              # Astro marketing site
│       ├── src/
│       ├── astro.config.mjs
│       └── package.json      # name: @bookly/landing
├── packages/
│   ├── db/                   # Supabase typed client
│   │   ├── index.ts
│   │   └── package.json      # name: @bookly/db
│   └── billing/              # provider-agnostic billing
│       ├── index.ts          # see topic 03
│       ├── pricing.json
│       ├── hu.ts             # Barion adapter
│       ├── paddle.ts         # Paddle adapter
│       └── lemon.ts          # Lemon Squeezy adapter
├── pnpm-workspace.yaml
├── turbo.json
├── package.json
└── .github/workflows/deploy.yml
```

`pnpm-workspace.yaml`:

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

`turbo.json`:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "dev": { "cache": false, "persistent": true },
    "build": { "outputs": [".svelte-kit/**", "dist/**", ".astro/**"] },
    "typecheck": { "dependsOn": ["^build"] },
    "lint": {}
  }
}
```

Root `package.json` declares `packageManager: pnpm@10.33.0`. **Do not pin
pnpm in CI** — that conflicts. See gotcha [G-05](../gotchas.md#g-05-github-actions-pnpm-action-conflicts-with-packagemanager-pin).

---

## 3. VPS bootstrap

The full transcript lives in `internal-docs/VPS-SETUP.md` of the source
project. Condensed:

```bash
# Provision: Hetzner CX22, Ubuntu 24, SSH key only.
ssh root@<ip>

# System update (non-interactive — see gotcha G-08)
export DEBIAN_FRONTEND=noninteractive
apt-get update -qq && apt-get upgrade -y -qq \
  -o Dpkg::Options::='--force-confdef' \
  -o Dpkg::Options::='--force-confold'

# Base packages — NOTE: `file` is not included by default in Ubuntu 24
apt install -y curl git build-essential file ufw fail2ban

# Firewall
ufw allow 22/tcp && ufw allow 80/tcp && ufw allow 443/tcp && ufw enable

# fail2ban (Ubuntu 24 needs backend = systemd, see gotcha G-03)
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime  = 10m
findtime = 10m
maxretry = 5
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled  = true
backend  = systemd
port     = ssh
maxretry = 5
bantime  = 30m
findtime = 10m
EOF
systemctl enable --now fail2ban

# Node 20 LTS + pnpm + PM2
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
npm install -g pnpm pm2
```

---

## 4. Caddy with Cloudflare DNS plugin

```bash
# Install xcaddy and build a Caddy with the Cloudflare DNS plugin
apt install -y golang-go
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
~/go/bin/xcaddy build \
  --with github.com/caddy-dns/cloudflare \
  --output /usr/bin/caddy

caddy list-modules | grep cloudflare    # sanity check

# Token file (chmod 600!)
mkdir -p /etc/caddy
printf 'CLOUDFLARE_API_TOKEN=<token>\n' > /etc/caddy/env
chmod 600 /etc/caddy/env

# Systemd drop-in to load the env file
mkdir -p /etc/systemd/system/caddy.service.d
cat > /etc/systemd/system/caddy.service.d/override.conf << 'EOF'
[Service]
EnvironmentFile=/etc/caddy/env
EOF

systemctl daemon-reload
```

**Caddyfile** (`/etc/caddy/Caddyfile`):

```caddy
example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }

    # SvelteKit app routes
    @app path /login* /register* /logout* /cancel/* /api/* /_app/* /superadmin*
    handle @app {
        reverse_proxy localhost:3000
    }

    # Trailing slash redirect: /foo/ → /foo
    @trailing path_regexp trailing ^(.+)/$
    handle @trailing {
        redir {re.trailing.1} 301
    }

    # Astro immutable assets — 1 year cache
    handle /_astro/* {
        header Cache-Control "public, max-age=31536000, immutable"
        root * /var/www/example/apps/landing/dist
        file_server
    }

    handle /fonts/* {
        header Cache-Control "public, max-age=31536000, immutable"
        root * /var/www/example/apps/landing/dist
        file_server
    }

    # Static marketing pages (Astro file format)
    handle {
        rewrite /sitemap.xml /sitemap-index.xml
        root * /var/www/example/apps/landing/dist
        try_files {path} {path}.html {path}/index.html
        file_server
    }

    # Real 404 (NOT homepage with status 200) — see gotcha G-04
    handle_errors {
        @404 expression {err.status_code} == 404
        handle @404 {
            root * /var/www/example/apps/landing/dist
            rewrite * /404.html
            file_server
        }
    }
}

# Wildcard tenant subdomains → SvelteKit app
*.example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:3000
}
```

> **Both blocks need the `tls` directive.** See gotcha [G-01](../gotchas.md#g-01-wildcard-ssl-silently-falls-back-to-per-host-issuance).

```bash
caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
systemctl reload caddy

# Verify
journalctl -u caddy --since '1 min ago' | grep -i obtained
# Expect: "certificate obtained successfully" for *.example.com
```

---

## 5. PM2 ecosystem

`apps/app/ecosystem.config.cjs`:

```cjs
const fs = require('fs')
const path = require('path')

function loadEnv(envPath) {
  const result = {}
  try {
    for (const line of fs.readFileSync(envPath, 'utf8').split('\n')) {
      const trimmed = line.trim()
      if (!trimmed || trimmed.startsWith('#')) continue
      const eqIndex = trimmed.indexOf('=')
      if (eqIndex === -1) continue
      result[trimmed.substring(0, eqIndex).trim()] =
        trimmed.substring(eqIndex + 1).trim()
    }
  } catch {}
  return result
}

const dotenv = loadEnv(path.resolve(__dirname, '.env'))

module.exports = {
  apps: [{
    name: 'app',
    script: 'apps/app/build/index.js',
    args: '--port 3000',
    cwd: '/var/www/example',
    env: {
      ...dotenv,
      HOST_HEADER: 'host',
      NODE_ENV: 'production',
    },
  }],
}
```

`HOST_HEADER: 'host'` is SvelteKit's hint to read the `Host` header for
multi-tenant subdomain routing.

---

## 6. GitHub Actions deploy

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

concurrency:
  group: deploy-prod
  cancel-in-progress: false   # see gotcha G-13

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /var/www/example
            git fetch origin main
            git reset --hard origin/main           # see gotcha G-07
            pnpm install --frozen-lockfile
            pnpm build
            pm2 startOrReload apps/app/ecosystem.config.cjs --update-env
            pm2 save
            sudo systemctl reload caddy            # see gotcha G-12

      # Optional: post-deploy one-shot job (e.g. catalog sync after billing
      # changes — see topic 03). Skip silently if no secret is configured.
      - name: Trigger post-deploy sync
        if: success()
        run: |
          if [ -z "${{ secrets.CRON_SECRET }}" ]; then
            echo "::warning::CRON_SECRET not set; post-deploy sync skipped"
            exit 0
          fi
          sleep 30   # let PM2 reload finish
          curl -fSs --max-time 120 \
            -H "Authorization: Bearer ${{ secrets.CRON_SECRET }}" \
            https://example.com/api/cron/post-deploy-sync
```

GitHub Secrets to set:
- `VPS_HOST` = the IPv4.
- `VPS_USER` = `root` or your deploy user.
- `VPS_SSH_KEY` = private key (matching the public key in
  `~/.ssh/authorized_keys` on the VPS).
- `CRON_SECRET` = optional, for post-deploy cron.

---

## 7. Environment variables

`.env.example` (committed):

```bash
PUBLIC_APP_DOMAIN=example.com
PUBLIC_APP_URL=https://example.com

PUBLIC_SUPABASE_URL=
PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SECRET_KEY=

RESEND_API_KEY=
RESEND_FROM_EMAIL=

# Billing — see topic 03
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

Real `.env` lives at `apps/app/.env` on the server. Populate from a
password manager. Don't commit. `.gitignore` already excludes `.env`.

The PM2 `ecosystem.config.cjs` reads `apps/app/.env` directly because
PM2 doesn't auto-load `.env` from the cwd.

---

## 8. Healthcheck

`apps/app/src/routes/api/healthz/+server.ts`:

```ts
import { supabaseAdmin } from '$lib/server/supabase'

export async function GET() {
  try {
    const { error } = await supabaseAdmin.from('tenants').select('id').limit(1)
    if (error) return new Response('db error', { status: 503 })
    return new Response('ok', { status: 200 })
  } catch {
    return new Response('unreachable', { status: 503 })
  }
}
```

Configure UptimeRobot to hit `https://example.com/api/healthz` every 5
minutes. Notification: SMS + Discord webhook.

---

## 9. Common operations cheat sheet

```bash
# Tail app logs
pm2 logs app

# Restart app (rare; reload is preferred)
pm2 restart app --update-env

# Tail Caddy
journalctl -u caddy -f

# Manual deploy from your laptop (when CI is wedged)
ssh root@<ip> 'cd /var/www/example && git fetch && git reset --hard origin/main && pnpm i && pnpm build && pm2 reload all && systemctl reload caddy'

# Rollback to previous commit
ssh root@<ip> 'cd /var/www/example && git reset --hard HEAD~1 && pnpm i && pnpm build && pm2 reload all'

# Disk fill check (CX22 has 40GB)
df -h

# Snapshot before risky change
# Hetzner Console → Server → Snapshots → Create
```

---

## 10. What this setup does NOT do

- **Multi-region.** One VPS, one region. For two regions you need replication
  (or a managed Postgres with read replicas) and DNS-based failover.
- **Zero-downtime database migrations.** Schema changes that aren't
  backwards-compatible cause a few seconds of errors during PM2 reload. See
  topic 14.
- **Auto-scaling.** PM2 cluster mode is possible (one process per CPU), but
  you'll outgrow CX22's 2 vCPU long before you outgrow single-node PM2.
  Vertical scale to CX42 (8GB) is one click and €10/m.
- **WebSocket persistence across reload.** PM2 reload drops WS clients;
  they reconnect. Fine for most use cases. For realtime-heavy products,
  Fly.io or self-hosted with HAProxy is a better shape.
