# 14 — Example: Next.js + Prisma

> **Status:** Outline.

Prisma Migrate generates SQL from schema diffs. `prisma migrate deploy`
runs in CI. Neon's branching feature is excellent for migration
testing — branch off prod, run new migration, verify, then apply to
main branch. PITR included on Pro.
