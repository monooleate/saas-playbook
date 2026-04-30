# 05 — Example: SvelteKit + Supabase

> **Status:** Outline.

The Grabit pattern (full code in v0.2): `tenant_addons` table queried by
`isAddonActive(tenantId, slug)`. Guards in `apps/app/src/lib/server/guards/addonGuard.ts`
expose `requireAddon`, `requireResourcePillar`, `requireEventPillar`,
`requireCombinedBooking`. Each can redirect (for HTML routes) or return
403 (for API routes).

PostHog feature flags layered on top via `posthog-node` server-side
SDK; client-side via `posthog-js` SDK with a server-rendered initial
flag set.
