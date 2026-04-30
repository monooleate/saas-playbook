# 14 — Data & Database Operations

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Day 1 (migrations, RLS); before launch (backups).
> **Cost:** 2–5 days for the foundations.

## Scope

Everything about the data layer that isn't application logic:

1. **Schema migrations** — additive, reversible, expand-then-contract.
2. **Backups** — provider-managed, point-in-time recovery, *tested*
   restore.
3. **Row-Level Security (RLS)** — when supported (Postgres). Pattern
   for tenant isolation.
4. **Soft delete vs hard delete** — `deleted_at` columns, retention,
   final purge.
5. **Audit log** — append-only `audit_log` table.
6. **GDPR data export & delete** — endpoints + tests (cross-link to
   topic 15).
7. **Read replicas / connection pooling** — when needed.

## Why it matters

More solo SaaS founders lose customers to a bad migration than to
feature gaps. The classic fail:
- Run `ALTER TABLE foo DROP COLUMN bar` during business hours.
- Old app version still references `bar`. Errors everywhere.
- Customer support storm; revert scrambles; data lost.

Doing migrations safely is a learnable skill. The expand-then-contract
pattern (below) prevents 90% of incidents.

## The expand-then-contract pattern

For any non-trivial change:

1. **Expand** — add the new column / table; old code keeps using the
   old shape; new code writes both old and new.
2. **Backfill** — populate the new column for existing rows.
3. **Migrate readers** — old code paths updated to read from new column.
4. **Contract** — drop the old column.

Each step is a deploy. Each step is reversible until the next deploy.

For Stripe-scale, see [Online migrations at scale](https://stripe.com/blog/online-migrations).
For solo scale, the same pattern; just more relaxed timing.

## Backup discipline

- Managed Postgres (Supabase, Neon, RDS, PlanetScale) does daily
  backups. Verify: open the dashboard, find the backup list, write
  down where it is.
- Point-in-time recovery (PITR) is what you actually want. Pro-tier
  plans usually include 7–30 days.
- **Restore drill:** at least once, restore a backup to a fresh DB
  and verify the app boots. Without this, you don't have backups; you
  have hopes.
- Off-provider backup: a weekly `pg_dump` to S3/R2 is cheap insurance
  against provider-level disaster.

## What goes in v0.2

- Migration tooling per stack (Supabase migrations, Prisma Migrate,
  Drizzle Kit, PlanetScale deploy requests).
- Soft delete table pattern + retention cron.
- GDPR data export endpoint pattern.
- Audit log table schema and triggers.
- RLS policy patterns (cross-link to topic 02 example).
- The "yes, you can read your own data, you can't read others'"
  integration test.

## Sources

- Stripe — *Online migrations at scale*:
  [stripe.com/blog/online-migrations](https://stripe.com/blog/online-migrations).
- Supabase RLS guide.
- Prisma schema migrations docs.
- PlanetScale's deploy request workflow:
  [planetscale.com/docs/concepts/deploy-requests](https://planetscale.com/docs/concepts/deploy-requests).
