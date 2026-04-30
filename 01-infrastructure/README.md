# 01 — Infrastructure & DevOps

> **When to implement:** Day 1, before the first user.
> **Cost:** 1–3 days for a single VPS, ~€10/month at the Hetzner/Vultr/Hostinger
> tier. ~$0–$20 for a serverless setup until first ~10 paying customers.
> **Belongs to:** *Foundations*. Without this, everything else is a demo.

## What this topic covers

The narrow, opinionated definition of "infrastructure" used by this playbook:

1. **Where the code runs** — VPS, PaaS, or serverless.
2. **How traffic gets there** — DNS, TLS, reverse proxy, wildcard subdomains
   for multi-tenancy.
3. **How you ship changes** — git push → production with one rollback button.
4. **How env config is managed** — secrets, per-environment values, what's
   committed and what isn't.
5. **How the codebase is shaped** — single repo vs. monorepo, how to share
   `db` / `billing` packages between the marketing site and the app.

What this topic explicitly does NOT cover (and where it lives instead):

- Logging, metrics, error tracking, alerting → topic **13 — Observability**.
- WAF, DDoS, secrets rotation, dependency scanning → topic **11 — Security**.
- Database migrations, backups, restore drills → topic **14 — Data Operations**.

---

## Why it matters

Two reasons one beats the other depending on your stage:

**Pre-launch:** You don't have customers, so revenue isn't on the line yet.
But every hour you spend wrestling with Kubernetes is an hour not spent
shipping product. The right infra at this stage is "boring, fast to set up,
and easy to migrate away from." The playbook recommends a single VPS for most
solo founders pre-revenue. It costs €10/month and you can move off it in a
weekend.

**Post-launch:** Now downtime costs money and trust. The right infra at this
stage is "uncomplicated rollback, predictable bill, and someone to call when
the disk fills up." A single VPS can carry you to €10k MRR comfortably; a
PaaS like Fly or Railway or Render is the natural next step when you need a
second region or zero-downtime deploys with no ops effort.

The mistake to avoid: optimizing for scale you don't have. Stripe runs on
heavily-engineered infra. Linear runs on Vercel + a managed Postgres. Linear
is a $400M ARR business serving millions of users on infra one person can
understand in an afternoon. You are not Stripe.

---

## The three viable shapes

| Shape           | Examples              | When it fits                        | Cost (m€) | Pain when wrong                       |
|-----------------|-----------------------|-------------------------------------|-----------|---------------------------------------|
| Single VPS      | Hetzner, Vultr, OVH   | One process, predictable load       | 5–20      | Manual ops; one disk fills up = down |
| PaaS            | Fly.io, Railway, Render, Vercel | Fast iteration, hate ops          | 20–200    | Vendor lock-in, surprise bills       |
| Serverless      | Vercel, Cloudflare Workers, Lambda | Spiky traffic, mostly read       | 0–100     | Cold starts, debugging, cost-per-request |

The reference implementation in this playbook (Grabit) is **single VPS +
Caddy + PM2**. It is the cheapest, most controllable shape, and it survived
the first ~200 paying tenants on a 4GB Hetzner instance.

The example files give the same patterns adapted to:
- **SvelteKit + Supabase + Hetzner** (the Grabit shape).
- **Next.js + Prisma + Vercel** (the PaaS shape; Vercel + Neon).
- **Remix + PlanetScale + Fly.io** (the PaaS-on-VM shape; Fly + PlanetScale).

---

## Mental model: separation of concerns at the edge

Whatever shape you pick, the boundaries are the same:

```
   ┌────────┐         ┌──────────────┐    ┌──────────────────┐
DNS│ Anycast│  TLS    │  Reverse     │    │  App processes   │
  ─►│  edge  ├────────►│  proxy       ├───►│ (1..N replicas)  │
   └────────┘         │  / static    │    │                  │
                      │  hosting     │    └────────┬─────────┘
                      └──────────────┘             │
                                                   ▼
                                          ┌──────────────────┐
                                          │  Database (managed)│
                                          └──────────────────┘
```

- DNS and TLS belong at the edge. Don't terminate TLS in your app process.
- A reverse proxy serves static assets directly (with cache headers) and
  forwards everything else. This pattern is unchanged whether the proxy is
  Caddy, Cloudflare, Vercel's edge, or Fly's proxy.
- App processes are stateless. State lives in Postgres / SQLite / a managed
  store, not on the app's disk.
- The database is **always managed**, even on a VPS. Self-hosted Postgres
  for a solo SaaS is a learning project disguised as infrastructure. Use
  Supabase, Neon, PlanetScale, or AWS RDS.

This is the same model Vercel publishes for itself, the same model Fly
documents in its [App architecture guide](https://fly.io/docs/reference/architecture/),
and the same model `caddyfile` examples on `caddyserver.com` use. It's not
fancy because it doesn't need to be.

---

## How a solo founder picks a shape

Decision tree, in order:

1. **Is the app a long-running process (WebSocket server, background workers)
   you can't easily run as serverless functions?** → Single VPS or PaaS.
   Skip serverless.
2. **Do you understand how to administer Linux (security updates, log
   rotation, backups, fail2ban) or are you happy to learn?** → Single VPS is
   €10/m and you control everything.
3. **Are you allergic to ops?** → PaaS. Vercel for Next.js + Vercel-friendly
   stacks. Fly for everything else. Render or Railway as middle ground.
4. **Is your traffic spiky (e.g., a calculator that gets occasional bursts)?**
   → Serverless can be the right answer. Check pricing carefully — Vercel
   bandwidth at scale is brutal.

The Grabit (`bookolj`) project chose **single VPS** because:
- The founder wanted root access for diagnostics.
- The cost ceiling is predictable (€5/m for the CX22 tier).
- The product is multi-tenant via subdomains, and wildcard SSL on Vercel
  free is gated behind their Pro tier — Caddy + Cloudflare DNS is free.

---

## Reference architecture (the Grabit shape)

Documented in detail in [`examples/sveltekit-supabase.md`](./examples/sveltekit-supabase.md).

- **Provider:** Hetzner Cloud, CX22 (2 vCPU, 4 GB RAM, 40 GB SSD), Ubuntu 24
  LTS, ~€5/m.
- **DNS:** Cloudflare, **proxy disabled** (orange cloud OFF) so Caddy can
  obtain Let's Encrypt certificates. Wildcard A record `*.example.com` →
  the VPS IPv4.
- **TLS:** Caddy with the `caddy-dns/cloudflare` plugin built via `xcaddy`,
  using DNS-01 challenge for the wildcard. The Cloudflare API token is
  zone-scoped (`Edit zone DNS` template, restricted to one zone).
- **Reverse proxy:** Caddy fronts both the Astro-built marketing site (served
  as static files directly by Caddy with `try_files` and 404 fallback) and
  the SvelteKit app (proxied to `localhost:3000`). Tenant subdomains all
  hit the same SvelteKit process.
- **Process manager:** PM2 with an `ecosystem.config.cjs` file. The deploy
  script does `pm2 startOrReload`, which is zero-downtime if the process
  exits cleanly.
- **Monorepo:** pnpm workspace with `apps/app` (SvelteKit), `apps/landing`
  (Astro), `packages/db` (typed Supabase client), `packages/billing`
  (provider-agnostic billing adapters). Turbo coordinates `dev` / `build` /
  `typecheck` / `lint` tasks.
- **CI/CD:** GitHub Actions on push to `main` → SSH to VPS → `git pull && pnpm
  install && pnpm build && pm2 reload`. ~90 seconds end-to-end. A second
  step pings a post-deploy cron endpoint (e.g. payment-provider catalog
  re-sync) so the freshly-deployed code can run a one-shot job.
- **Backups:** Hetzner snapshot (manual, weekly). Database backups are
  handled by Supabase (point-in-time recovery on the Pro tier — see
  topic 14).
- **Brute force defense:** `fail2ban` with the `[sshd]` jail using
  `backend = systemd` (Ubuntu 24's sshd logs to journald, not auth.log —
  default `backend = auto` finds nothing).

The full configuration with every gotcha encountered in setup is in
[`examples/sveltekit-supabase.md`](./examples/sveltekit-supabase.md).

---

## Read next

1. [`checklist.md`](./checklist.md) — sprint-ready setup checklist for any of
   the three shapes.
2. [`best-practices.md`](./best-practices.md) — what Vercel, Fly, and the
   Caddy maintainers actually recommend, with sources.
3. [`gotchas.md`](./gotchas.md) — what bit us in production. Read before, not
   after, your first incident.
4. Pick your example: SvelteKit + Supabase, Next.js + Prisma, or
   Remix + PlanetScale.
