# 12 — Example: Remix + PlanetScale

> **Status:** Outline.

Vitest + Playwright. PlanetScale doesn't support local Docker (uses
Vitess); use a separate `test` branch or a local MySQL image with
Drizzle migrations applied. Tests use `db.transaction(async (tx) =>
... rollback)` pattern.
