# 10 — Example: Remix + PlanetScale

> **Status:** Outline.

`/admin/*` protected by a `requireSuperadmin(request)` helper at the
top of every loader/action. Drizzle `audit_log` table. Action handlers
wrap mutations with `withAudit(request, fn)`.
