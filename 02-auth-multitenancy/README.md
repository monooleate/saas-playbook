# 02 — Auth & Multi-tenancy

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Day 1.
> **Cost:** 3–5 days for password + magic link + tenant isolation.
> **Belongs to:** *Foundations*.

## What this topic covers

Two related concerns, kept in one topic because solo founders build them
together.

### Auth (the user identity half)

- Sign-up / sign-in: email+password, magic link, OAuth (Google,
  GitHub, optionally Microsoft).
- Session management: cookies vs JWT, sliding expiry, "remember me",
  forced logout on password change.
- Account recovery: password reset, email change confirmation, 2FA.
- Account deletion (GDPR-compatible flow).
- Email verification: required vs optional, when to enforce.
- Bot defense: rate limiting on signup, basic invisible CAPTCHA.

### Multi-tenancy (the workspace isolation half)

- Tenant data model: who owns what. Choices: shared schema with
  `tenant_id` column (Grabit), schema-per-tenant, database-per-tenant.
  Trade-offs documented.
- Subdomain routing: `tenant.example.com` (Grabit, Linear, Notion) vs
  path routing (`example.com/tenant/...`) vs custom domains
  (Webflow, Carrd-style).
- Row-Level Security (RLS) when using Supabase / Postgres natively.
  Policies per table, the "every query has tenant_id" invariant.
- Membership: a user can belong to multiple tenants (Notion, Linear).
  Roles: owner, admin, member.
- Invitations: email-based, expiring tokens, accept/decline flow.
- Switching active tenant from the UI.

## Reference choices for solo founders

- **Auth library:** Supabase Auth or NextAuth/Auth.js or Clerk. The
  Grabit project uses Supabase Auth (built-in with the database). Clerk
  has the slickest UX and the highest cost; NextAuth is the most
  flexible.
- **Tenancy strategy:** **shared schema + `tenant_id` column + RLS** is
  the right default for solo SaaS. It scales to ~10k tenants on a
  single Postgres instance with no operational pain.
- **Routing strategy:** **subdomain** for B2B SaaS (looks professional,
  enables wildcard SSL approach in topic 01); **path** for B2C tools
  where users don't need a "workspace" feel.

## Why it matters

Auth bugs kill trust faster than any other category. A tenant data
leak — "I logged in and saw another company's data" — is a
near-extinction-level event for a 1-person SaaS. Topic 11 — Security
covers the threat model; this topic covers the implementation hygiene.

## What's in the rest of this folder

- [`checklist.md`](./checklist.md) — outline.
- [`best-practices.md`](./best-practices.md) — outline.
- [`gotchas.md`](./gotchas.md) — outline.
- [`examples/`](./examples/) — stub files for three stacks.

## Source projects to mine

When this topic is filled out (v0.2), the source material will come from:

- **Grabit** — Supabase Auth + RLS + tenant subdomains via `Host` header
  resolution. The `apps/app/src/hooks.server.ts` file does the
  request-time tenant resolution.
- **Notion** — public docs at
  [developers.notion.com/docs/authorization](https://developers.notion.com/docs/authorization)
  for OAuth as a tenant-aware integration story.
- **Linear** — workspaces + invites pattern, documented in their
  [API docs](https://developers.linear.app/docs).
- **Stripe Connect** — for the rare case where each tenant has its own
  Stripe account; mostly out of scope for this playbook.
- **Supabase** — RLS docs at
  [supabase.com/docs/guides/auth/row-level-security](https://supabase.com/docs/guides/auth/row-level-security).
