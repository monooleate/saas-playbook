# 05 — Example: Remix + PlanetScale

> **Status:** Outline.

GrowthBook self-hosted on Fly.io (close to the app) for sub-ms flag
evaluation, or PostHog. Entitlements queried from PlanetScale's
`tenant_addons` table. Loaders call a `requireFeature(request, key)`
helper that throws a Remix `redirect` or `Response(403)`.
