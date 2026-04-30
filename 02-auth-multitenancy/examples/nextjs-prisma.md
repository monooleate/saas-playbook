# 02 — Auth & Multi-tenancy — Example: Next.js + Prisma

> **Status:** Outline. Full implementation forthcoming.

## What will go here

The canonical Next.js shape uses **Auth.js (NextAuth v5)** with the
**Prisma adapter** for sessions and accounts.

Tenant routing on Vercel: subdomain rewrites in `middleware.ts` (the
[Vercel Platforms Starter Kit](https://github.com/vercel/platforms) is
the canonical reference; this example will adapt that pattern with
Prisma instead of Drizzle).

The example will include:

1. `auth.ts` configuring Auth.js with email + Google providers and the
   Prisma adapter.
2. `middleware.ts` resolving subdomains → tenant slug, rewriting to
   `/_tenants/[slug]/...`.
3. Prisma schema for `User`, `Account`, `Session`, `VerificationToken`
   (Auth.js shape) plus `Tenant` and `Membership`.
4. ORM-level tenant guards: a `withTenant(prisma, tenantId)` wrapper
   that injects `tenantId` into every query (since Prisma + Postgres
   can also use RLS, but most Next.js + Prisma projects don't).
5. Invite flow with single-use tokens.

## Stand-in references for now

- Auth.js docs: [authjs.dev](https://authjs.dev).
- Vercel Platforms Starter Kit:
  [github.com/vercel/platforms](https://github.com/vercel/platforms).
- Prisma multi-tenancy patterns:
  [prisma.io/docs/orm/prisma-client/queries/multi-schema](https://www.prisma.io/docs/orm/prisma-client/queries/multi-schema).
