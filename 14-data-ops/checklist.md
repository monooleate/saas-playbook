# 14 — Data Operations — Checklist

> **Status:** Outline.

## Migrations

- Migration tool chosen and committed.
- Every migration is reviewed; PR template includes "is this
  expand-then-contract?".
- Destructive migrations (DROP COLUMN, DROP TABLE) require two
  deploys.
- Migrations run automatically on deploy (or as a separate gated
  step).

## Backups

- Provider's automatic backup verified (open the dashboard).
- Point-in-time recovery enabled.
- One restore drill performed. Date noted.
- Off-provider weekly `pg_dump` to object storage.
- Retention policy documented.

## Tenant isolation

- RLS on every business table (Postgres). Tested with two-tenant
  integration test.
- Soft-delete columns (`deleted_at`) on all user-facing tables.
- Retention cron for soft-deleted rows after N days.

## Audit log

- `audit_log` table append-only.
- Hooks on superadmin actions (topic 10) and key business events.

## GDPR

- Data export endpoint returns all data for a user / tenant.
- Account delete endpoint hard-deletes within 30 days.
- Email confirmation on both.

(Each item expanded in v0.2.)
