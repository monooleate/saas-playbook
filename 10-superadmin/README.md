# 10 — Superadmin

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Day 1 — before first paying customer.
> **Cost:** 1–2 days for v1; iterate forever.

## Scope

The internal-only admin surface for the SaaS founder/operator. Not a
customer-facing feature. Often skipped, then frantically built when the
first urgent support ticket lands.

1. **Tenant list / detail** — search, view, edit any tenant.
2. **"Sign in as user"** — impersonation with a non-dismissible banner.
3. **Plan / addon overrides** — toggle features on for a customer
   without sending them through checkout.
4. **Refunds and credits** (cross-link to topic 03).
5. **Account merge / split** (rare but eventually needed).
6. **System dashboard** — recent signups, churn, MRR, error rate (light
   version of topic 13).
7. **Audit log of admin actions** — every change records who did it,
   when, with what.
8. **Soft-delete recovery** — un-delete a tenant within retention window.

## Why it matters

Without superadmin, every support ticket requires the founder to write
a SQL UPDATE. That's fine for the first 5 tickets, untenable at 50.

Worse: ad-hoc SQL has no audit trail. When a customer says "you
changed my plan without telling me," you have no history.

## The auth model

Superadmin is a property of the *user*, not the tenant. Some users
have `is_superadmin = TRUE`. The superadmin routes
(`/superadmin/*`) check this; everything else is normal app routes.

The Grabit pattern: a single boolean column, set by direct SQL during
setup. Not exposed in any UI. Two superadmin users (founder +
co-founder/contractor).

## Impersonation pattern

```
1. Superadmin clicks "Sign in as user" on a tenant detail page.
2. Server creates a session for that user with a flag
   `impersonating_admin_id = <superadmin_user_id>`.
3. Server redirects to the tenant's subdomain with a session cookie.
4. App renders a banner: "You are signed in as Customer X (impersonator
   Y). Click to exit."
5. Banner is non-dismissible until impersonation ends.
6. All actions performed during impersonation are logged with both
   `user_id` and `impersonating_admin_id`.
```

## What goes in v0.2

- Auth check + middleware pattern.
- Tenant detail UI (sketch).
- Audit log table schema and hooks for writing to it.
- Soft-delete pattern + recovery UI.
- Refund button (cross-link to topic 03).
- "Update billing without checkout" admin override flow.

## Sources

- Stripe's internal admin tooling has been written about in their blog.
- Linear / Notion likely have similar internal tools; not public.
- Stripe Sigma (read-only SQL on Stripe data) is a public pattern that
  inspired many founder-built admin panels.
