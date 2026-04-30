# 02 — Auth & Multi-tenancy — Checklist

> **Status:** Outline. Section headings are real; full item-level content
> forthcoming. The READ ME ME at the topic root has the mental model that
> drives this checklist.

## Auth

- Sign-up flow with email verification.
- Sign-in flow with rate limit (5 attempts / minute).
- OAuth providers (Google, GitHub) configured.
- Password reset with single-use tokens that expire in 30 minutes.
- 2FA via TOTP for admin/owner accounts (post-launch is fine).
- Sessions stored as `httpOnly`, `secure`, `sameSite=lax` cookies.
- Forced logout on password reset.
- Account deletion: see topic 15 (Legal & Compliance) — GDPR data delete
  flow.
- Bot defense: hCaptcha or Cloudflare Turnstile on signup.

## Multi-tenancy

- Tenant data model decision documented (shared schema vs schema-per-tenant).
- Every business table has `tenant_id NOT NULL`.
- RLS policies on every table (or equivalent ORM-side guard).
- Subdomain resolution at the framework hook layer; never trust the body.
- Membership table linking `user_id` to `tenant_id` with role.
- Invite flow: email with expiring token; accept creates membership.
- "Switch tenant" UI for users with multiple memberships.
- Audit log of admin actions (also belongs in topic 14).

## Operational

- A "Sign in as user" superadmin button (see topic 10) that produces a
  banner so support can't impersonate silently.
- Test that confirms a tenant cannot read another tenant's data
  (integration test using two tenant accounts).

(Status: outline; each item to be expanded with reasoning, gotchas, and
verification steps.)
