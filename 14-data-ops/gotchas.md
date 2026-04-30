# 14 — Data Operations — Gotchas

> **Status:** Outline.

## Categories

1. **`DROP COLUMN` during deploy.** Old container still references it
   for ~30s. Errors. Fix: expand-then-contract.
2. **Backup never tested.** Discovered to not work the day you need
   it.
3. **RLS bypass via service-role key.** Service role bypasses RLS by
   design; using it from client code is catastrophic.
4. **Soft-deleted rows still visible due to missing WHERE clause.**
   Tenant sees a deleted booking. Fix: ORM-side default filter.
5. **`pg_dump` of 50GB DB takes 6 hours; retention overlap fails.**
   Fix: incremental backups or paid PITR.
6. **GDPR delete leaves orphans.** `users.delete()` cascades, but
   `audit_log` retains user_id. Fix: anonymize, don't delete the audit
   trail.
7. **Migration runs twice on parallel deploys.** Fix: lock during
   migration, or run as a gated CI step.
8. **Schema change locks the entire table.** Tools like
   `pg-online-schema-change` or `gh-ost` for big tables.

(Real incidents in v0.2.)
