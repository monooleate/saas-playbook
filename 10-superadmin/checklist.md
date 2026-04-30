# 10 — Superadmin — Checklist

> **Status:** Outline.

- `users.is_superadmin` boolean column.
- `/superadmin/*` route group with auth guard.
- Tenant list with search by id / email / slug.
- Tenant detail page with: plan, status, addons, billing history,
  members.
- "Sign in as user" with banner (see README pattern).
- Refund button (links to topic 03 flow).
- Plan / addon override UI without going through checkout.
- Audit log table + hooks on every superadmin action.
- Soft-delete recovery within retention window.
- Two superadmin accounts at minimum (so you can lock yourself out and
  recover).
