# 01 — Infrastructure — Example: Remix + PlanetScale + Fly.io

The PaaS-on-VM shape. Persistent processes, full Node, multi-region, no
ops-on-Linux. Slightly more involved than Vercel but more flexible and
about half the cost at scale.

---

## 1. Stack overview

| Layer            | Choice                              | Why                                                    |
|------------------|-------------------------------------|--------------------------------------------------------|
| App framework    | Remix (`@remix-run/node`)           | Full SSR, no edge-only constraints                     |
| ORM              | Drizzle ORM                         | Lightweight, schema-as-code, MySQL-friendly            |
| Database         | PlanetScale (managed MySQL/Vitess)  | Branching, schema migrations, global replicas          |
| Email            | Postmark (or Resend)                | Postmark for transactional reliability                 |
| File hosting     | Cloudflare R2                       | Cheap egress, S3-compatible                            |
| Reverse proxy    | Fly's edge proxy (built-in)         | Anycast, automatic TLS                                 |
| Process model    | Fly Machines (persistent VMs)       | Predictable, supports WebSockets                       |
| CI/CD            | GitHub Actions → `fly deploy`       | Standard, transparent                                  |
| Monorepo         | pnpm workspaces                     | Same as before                                         |
| Cost             | $5/m for 1 shared CPU + 256 MB → scales linearly |

---

## 2. Repository layout

```
your-saas/
├── apps/
│   └── web/                  # Remix app
│       ├── app/              # Remix routes
│       ├── drizzle/
│       │   └── schema.ts
│       ├── package.json      # name: @yourorg/web
│       ├── Dockerfile        # Fly.io needs this
│       └── fly.toml
├── packages/
│   ├── db/                   # Drizzle exports
│   └── billing/              # see topic 03
├── pnpm-workspace.yaml
└── turbo.json
```

---

## 3. Fly.io app setup

```bash
# Install flyctl
brew install flyctl    # or curl -L https://fly.io/install.sh | sh

# One-time auth
fly auth signup         # or login

# From apps/web/
fly launch              # interactive — declines DB offer (we use PlanetScale)
```

`apps/web/fly.toml`:

```toml
app = "your-saas-web"
primary_region = "fra"   # Frankfurt — close to PlanetScale EU primary

[build]
  dockerfile = "Dockerfile"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = "stop"     # stop machines when no traffic (saves $$)
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[http_service.checks]]
  interval = "15s"
  timeout = "5s"
  grace_period = "10s"
  method = "get"
  path = "/healthz"

[[vm]]
  cpu_kind = "shared"
  cpus = 1
  memory_mb = 512
```

`apps/web/Dockerfile` (multi-stage with pnpm):

```dockerfile
FROM node:20-slim AS base
RUN corepack enable
WORKDIR /app

FROM base AS deps
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY apps/web/package.json apps/web/
COPY packages/db/package.json packages/db/
COPY packages/billing/package.json packages/billing/
RUN pnpm install --frozen-lockfile

FROM deps AS build
COPY . .
RUN pnpm --filter=@yourorg/web build

FROM base AS runtime
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/apps/web/build ./apps/web/build
COPY --from=build /app/apps/web/public ./apps/web/public
COPY --from=build /app/apps/web/package.json ./apps/web/

EXPOSE 3000
CMD ["node", "apps/web/build/index.js"]
```

Deploy:

```bash
fly deploy --remote-only   # builds in Fly's builder, not your laptop
```

---

## 4. PlanetScale database

```bash
pscale auth login
pscale database create your-saas --region eu-central
pscale branch list your-saas

# Create connection password for production
pscale password create your-saas main prod-password
```

`apps/web/drizzle/schema.ts`:

```ts
import { mysqlTable, varchar, datetime, primaryKey, uniqueIndex } from 'drizzle-orm/mysql-core'

export const tenants = mysqlTable('tenants', {
  id: varchar('id', { length: 26 }).primaryKey(),     // ulid
  slug: varchar('slug', { length: 64 }).notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  createdAt: datetime('created_at').notNull(),
}, (t) => ({
  slugIdx: uniqueIndex('tenants_slug_idx').on(t.slug),
}))

export const users = mysqlTable('users', {
  id: varchar('id', { length: 26 }).primaryKey(),
  tenantId: varchar('tenant_id', { length: 26 }).notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  createdAt: datetime('created_at').notNull(),
}, (t) => ({
  tenantEmailIdx: uniqueIndex('users_tenant_email_idx').on(t.tenantId, t.email),
}))
```

PlanetScale enforces *no foreign keys* by default (Vitess constraint). Use
application-level checks. This is documented in the
[PlanetScale FAQ](https://planetscale.com/docs/learn/operating-without-foreign-key-constraints).

`apps/web/drizzle.config.ts`:

```ts
import type { Config } from 'drizzle-kit'

export default {
  schema: './drizzle/schema.ts',
  out: './drizzle/migrations',
  dialect: 'mysql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config
```

Schema changes use **PlanetScale branches + deploy requests**, not raw
migrations. See PlanetScale's [non-blocking schema changes](https://planetscale.com/docs/concepts/nonblocking-schema-changes).
Topic 14 has more.

---

## 5. Multi-tenant subdomain routing

Remix's loader sees the request; tenant resolution is straightforward:

```ts
// apps/web/app/lib/tenant.server.ts
import { db } from '@yourorg/db'

const APP_DOMAIN = process.env.APP_DOMAIN ?? 'example.com'

export async function tenantFromRequest(req: Request) {
  const url = new URL(req.url)
  const host = url.host
  if (host === APP_DOMAIN || host === `www.${APP_DOMAIN}`) return null
  const slug = host.replace(`.${APP_DOMAIN}`, '')
  const tenant = await db.query.tenants.findFirst({ where: (t, { eq }) => eq(t.slug, slug) })
  return tenant ?? null
}
```

Wildcard SSL on Fly:

```bash
fly certs add "*.example.com" --app your-saas-web
fly certs add "example.com"   --app your-saas-web
```

Fly handles DNS-01 if your DNS provider has an API integration; otherwise
add the TXT record manually. Cloudflare DNS works.

DNS records:
- `A` `@` → Fly's anycast IP4 (`fly ips list`).
- `AAAA` `@` → IPv6.
- `A` `*` → same IP4 (or `CNAME *` → app's `.fly.dev` hostname).

---

## 6. Environment variables

```bash
fly secrets set \
  DATABASE_URL="mysql://...?sslaccept=strict" \
  APP_DOMAIN="example.com" \
  POSTMARK_TOKEN="..." \
  STRIPE_SECRET_KEY="..." \
  STRIPE_WEBHOOK_SECRET="..."
```

These become `process.env.*` inside the container. Fly stores them
encrypted at rest.

---

## 7. Healthcheck

`apps/web/app/routes/healthz.ts`:

```ts
import { db } from '@yourorg/db'

export async function loader() {
  try {
    await db.execute(`SELECT 1`)
    return new Response('ok', { status: 200 })
  } catch {
    return new Response('db unreachable', { status: 503 })
  }
}
```

Fly's healthcheck (defined in `fly.toml`) hits this. If 3 consecutive
checks fail, the machine is replaced.

---

## 8. GitHub Actions deploy

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Fly.io

on:
  push:
    branches: [main]

concurrency:
  group: deploy-prod
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only --config apps/web/fly.toml --dockerfile apps/web/Dockerfile
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

Generate the token: `fly tokens create deploy --expiry 8760h`. Store as
`FLY_API_TOKEN` in GitHub Secrets.

---

## 9. Cost & scaling

- **Single shared-CPU machine, 512 MB:** ~$5/m. Carries first ~50 active
  tenants on a typical CRUD SaaS.
- **`auto_stop_machines = "stop"`:** machines stop after idle. First request
  after idle pays a ~3s cold start. Fine for B2B; bad for B2C.
- **Multi-region:** add a region with `fly machine clone --region <code>`.
  PlanetScale read replicas in the same regions; Drizzle uses the closest
  via `RDONLY` connection. Multi-region is a topic-14 concern; mention here
  for completeness.
- **Scaling vertically:** `fly scale vm shared-cpu-2x` doubles CPU/RAM.

---

## 10. What this setup does well

- WebSocket support out of the box (just listen on `internal_port`).
- Long-running jobs in the same process — no separate worker.
- Predictable, cheap (~$5–$25/m for first year).
- Multi-region with 4 commands.

## 11. What this setup does NOT do well

- **DX vs. Vercel.** Fly's preview URLs require manual setup
  (`fly machine clone --image <ref> --region staging`). Worth it for
  flexibility, not free.
- **PlanetScale's foreign-key story.** Application-level integrity is more
  defensive coding. Some teams prefer Neon (Postgres) for this.
- **Cold starts** with `auto_stop_machines`. Set `min_machines_running = 1`
  to avoid them; that's the always-on $5/m you're already paying.
