# 14 — Data Operations — Best Practices

> **Status:** Outline.

## Sources

- Stripe blog — *Online migrations at scale*.
- *Database Reliability Engineering* (Campbell, Majors).
- Supabase RLS docs.
- Prisma Migrate docs.

## Practices

- Always additive first. DROP last.
- Tests must run against a real DB (sqlite mocks are not real).
- Backups are not real until you've restored.
- One source of truth per piece of data.
- Soft delete by default; hard delete only by GDPR demand.
- RLS is defense in depth, not the only line — application code is
  also expected to filter by tenant.
