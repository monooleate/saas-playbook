# 01 — Infrastructure — Example: Next.js + Prisma + Vercel + Neon

The PaaS shape. Ops effort approaches zero; cost approaches non-zero.
Right choice for solo founders who hate Linux and want their weekends back.

---

## 1. Stack overview

| Layer            | Choice                              | Why                                                    |
|------------------|-------------------------------------|--------------------------------------------------------|
| App framework    | Next.js 15 (App Router)             | First-class Vercel support, ecosystem                  |
| ORM              | Prisma 5                            | Type-safe, fast iteration, multi-DB support            |
| Database         | Neon (managed Postgres)             | Free tier with branching, serverless billing           |
| Email            | Resend                              | Native Vercel integration                              |
| File hosting     | Vercel Blob OR Cloudflare R2        | R2 wins on bandwidth cost                              |
| Edge / CDN       | Vercel Edge Network (built-in)      | Zero config                                            |
| Process model    | Serverless functions (Edge or Node) | No process manager to maintain                         |
| CI/CD            | Vercel Git integration              | Push to main → preview URL; merge → production         |
| Monorepo         | pnpm workspaces + Turbo             | Same as VPS shape; works fine on Vercel                |
| Cost             | $0 free tier; ~$20/m at first paid customer; watchful at scale |

---

## 2. Repository layout

```
your-saas/
├── apps/
│   ├── web/                  # Next.js app
│   │   ├── app/              # App Router pages
│   │   ├── lib/
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   ├── package.json      # name: @yourorg/web
│   │   └── next.config.mjs
│   └── marketing/            # optional: separate marketing site (Next or Astro)
├── packages/
│   ├── db/                   # Prisma client + types re-exports
│   │   └── package.json      # name: @yourorg/db
│   └── billing/              # see topic 03
├── pnpm-workspace.yaml
├── turbo.json
└── vercel.json
```

`vercel.json` (root):

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "buildCommand": "cd ../.. && pnpm turbo build --filter=@yourorg/web",
  "ignoreCommand": "npx turbo-ignore @yourorg/web"
}
```

The `ignoreCommand` skips deploys when the app's dependency graph hasn't
changed. Turbo's `turbo-ignore` evaluates the affected packages.

---

## 3. Vercel project setup

1. Sign up at [vercel.com](https://vercel.com); free tier.
2. **New Project → Import** the GitHub repo.
3. **Root Directory:** `apps/web`. (Vercel auto-detects monorepos but
   making it explicit removes a class of bugs.)
4. **Framework Preset:** Next.js.
5. **Install Command:** `pnpm install --frozen-lockfile`.
6. **Build Command:** the `vercel.json` `buildCommand` overrides this.
7. **Output Directory:** `.next` (auto-detected).
8. Add env vars (see §6).
9. Add custom domain. Vercel handles TLS — including wildcards on Pro.

For wildcard subdomains on Vercel:
- Hobby tier: **not supported** for wildcards. You can add up to 50 custom
  domains, one per tenant. OK at small scale only.
- Pro tier: supports `*.example.com` as a single custom domain. ~$20/m.
- Enterprise: SSL for SaaS (`vercel.com/docs/multi-tenant`) for per-tenant
  custom domains. Out of scope here.

---

## 4. Database (Neon)

```bash
# Sign up at neon.tech
# Create a project; choose region closest to Vercel's deploy region (or
# the function's edge region — Neon's "primary" branch is regional)

# DATABASE_URL is provided. For Vercel + Prisma:
DATABASE_URL="postgres://...?sslmode=require"
DIRECT_URL="postgres://...?sslmode=require"   # for migrations
```

`apps/web/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")   // bypass connection pooler for migrations
}

model Tenant {
  id        String   @id @default(cuid())
  slug      String   @unique
  email     String
  createdAt DateTime @default(now())

  users     User[]
}

model User {
  id        String   @id @default(cuid())
  tenantId  String
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  email     String
  createdAt DateTime @default(now())

  @@unique([tenantId, email])
}
```

Run migrations (locally, against a Neon branch):

```bash
pnpm exec prisma migrate dev --name init
pnpm exec prisma generate
```

For production, Vercel runs `prisma migrate deploy` as part of the build:

```json
// apps/web/package.json
{
  "scripts": {
    "build": "prisma generate && prisma migrate deploy && next build"
  }
}
```

**Note:** running migrations during the build is fine for solo SaaS. At
scale, migrations should be a separate, gated step.

---

## 5. Multi-tenant subdomain routing

`apps/web/middleware.ts`:

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

const APP_DOMAIN = process.env.NEXT_PUBLIC_APP_DOMAIN || 'example.com'

export function middleware(req: NextRequest) {
  const host = req.headers.get('host') || ''
  const subdomain = host.replace(`.${APP_DOMAIN}`, '')

  // apex or www → marketing pages, no rewrite
  if (host === APP_DOMAIN || host === `www.${APP_DOMAIN}`) {
    return NextResponse.next()
  }

  // tenant subdomain → rewrite to /[tenant]/...
  const url = req.nextUrl.clone()
  url.pathname = `/_tenants/${subdomain}${url.pathname}`
  return NextResponse.rewrite(url)
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

App Router path: `apps/web/app/_tenants/[tenant]/page.tsx`.

The Vercel docs cover this pattern in their
[Platforms Starter Kit](https://github.com/vercel/platforms) — the canonical
multi-tenant Next.js reference.

---

## 6. Environment variables

In Vercel: **Project → Settings → Environment Variables**. Three scopes:
**Production**, **Preview**, **Development**.

```
NEXT_PUBLIC_APP_DOMAIN=example.com
NEXT_PUBLIC_APP_URL=https://example.com

DATABASE_URL=postgres://...?sslmode=require
DIRECT_URL=postgres://...?sslmode=require

NEXTAUTH_URL=https://example.com
NEXTAUTH_SECRET=

RESEND_API_KEY=
RESEND_FROM_EMAIL=

STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

Local dev: `vercel env pull .env.local` syncs from Vercel.

---

## 7. Healthcheck + uptime

`apps/web/app/api/healthz/route.ts`:

```ts
import { db } from '@yourorg/db'

export async function GET() {
  try {
    await db.$queryRaw`SELECT 1`
    return new Response('ok', { status: 200 })
  } catch {
    return new Response('db unreachable', { status: 503 })
  }
}

export const runtime = 'nodejs'      // not edge — Prisma needs Node
export const dynamic = 'force-dynamic'
```

UptimeRobot → `https://example.com/api/healthz` every 5 minutes.

---

## 8. Cost guardrails (CRITICAL)

Vercel Hobby tier limits to know before you scale:
- **100 GB bandwidth/month** — a viral post can burn this in hours.
- **6,000 build minutes/month** — usually not a concern for solo.
- **1M function invocations** — fine for typical SaaS.

Vercel Pro is $20/m flat + usage. Bandwidth at $40 per 100GB, function
invocations at $0.60 per 1M. Cost monitoring is non-optional.

**Set the Spend Cap** at *Settings → Billing → Spend Management*. Pick a
limit you can afford. When hit, Vercel pauses your project — you go down,
but you don't go bankrupt. See gotcha [G-10](../gotchas.md#g-10-vercel-bandwidth-bill-explodes-after-a-marketing-campaign).

Cheaper alternative for static-heavy traffic: put **Cloudflare** in front
of Vercel. Cloudflare's egress is free; Vercel sees only origin pulls. Some
features (preview deploys, edge functions) become awkward, so save this for
post-launch when traffic patterns are clearer.

---

## 9. Deployment workflow

Vercel's git integration is standard:

- Push to a feature branch → Vercel creates a preview URL like
  `your-saas-git-feature-x-org.vercel.app`.
- Open a PR → Vercel comments with the preview URL.
- Merge to `main` → Vercel deploys to production.
- Each commit gets a unique URL; previous deploys remain accessible for
  rollback via the Vercel dashboard's *Promote to production* button on any
  past deployment.

For migrations that go with code changes (additive only — see topic 14),
this is fine. For destructive changes, follow the expand/contract pattern
(also topic 14).

---

## 10. Observability (just enough — full content in topic 13)

- **Vercel Logs:** the `Logs` tab streams function logs in real time.
  10-day retention on Pro.
- **Sentry:** install `@sentry/nextjs`, add the DSN. Wraps all API routes
  and pages.
- **Web Vitals:** Vercel Analytics dashboard (free tier) shows LCP / CLS /
  INP per page.
- **Error rate alert:** Vercel doesn't alert on error rate by default;
  use Sentry's alert rules or BetterStack to monitor `/api/healthz`.

---

## 11. What this setup does NOT do well

- **Long-running jobs.** Vercel functions max at 10s on Hobby, 60s on Pro,
  300s on Pro with `maxDuration` config. For longer jobs, use a separate
  worker (e.g. on Fly Machines) or a queue (Inngest, Trigger.dev).
- **WebSockets.** Edge functions don't support persistent connections.
  Move WS to a different platform (Cloudflare Durable Objects, Pusher,
  Ably).
- **Heavy file uploads.** Vercel function bodies are limited (4.5 MB on
  Hobby). Use direct-to-blob uploads with a presigned URL.
- **Predictable monthly cost.** Vercel can surprise you. Cap and monitor.
