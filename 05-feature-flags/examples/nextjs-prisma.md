# 05 — Example: Next.js + Prisma

> **Status:** Outline.

Vercel + Next.js shape uses **Vercel's [Edge Config](https://vercel.com/docs/storage/edge-config)** for hot-flag reads (sub-ms),
or PostHog Server SDK with caching. Entitlements come from Prisma:
`tenant.addons` relation on the `Tenant` model.

`lib/entitlements.ts` exposes `requireFeature(req, key)` for use in API
routes and React Server Components.
