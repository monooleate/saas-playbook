# 10 — Example: SvelteKit + Supabase

> **Status:** Outline.

The Grabit pattern: `/superadmin/*` route group with a
`+layout.server.ts` that asserts `locals.user.is_superadmin`. Every
mutation goes through a server action that records to `audit_log`
(tenant_id, user_id, action, payload, before/after, ts).

Impersonation creates a Supabase session for the target user with
`impersonating_admin_id` in `app_metadata`; banner reads from there.
