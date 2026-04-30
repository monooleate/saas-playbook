# 02 — Auth & Multi-tenancy — Best Practices

> **Status:** Outline. Source pointers identified; analysis forthcoming.

## Sources to consult and summarize when filling this out

- **OWASP Authentication Cheat Sheet** —
  [cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html).
  Definitive on password storage, lockout, session lifetime.
- **OWASP Session Management Cheat Sheet** —
  [cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
- **Supabase RLS guide** —
  [supabase.com/docs/guides/auth/row-level-security](https://supabase.com/docs/guides/auth/row-level-security).
  Patterns for "user belongs to tenant" RLS policies.
- **Auth.js (NextAuth)** —
  [authjs.dev](https://authjs.dev). Patterns for adapters, providers.
- **Notion's multi-tenant model** — public docs at
  [developers.notion.com](https://developers.notion.com).
- **Stripe's "act as user" pattern** — Stripe support has a documented
  flow that adds a banner; the playbook should adopt the same.

## Practices we'll cover

- Magic link vs password: when each wins.
- Why password+email-as-username beats username+password (recovery).
- Hashing: argon2id is the modern default; bcrypt acceptable.
- "Sign in with Google" as primary, password as fallback.
- Tenant switching: never trust client-side state for active tenant; the
  server resolves on every request from session + URL.
- The Stripe "view-only impersonation" model.
- Row-Level Security as a defense-in-depth, not the only line.

(Each will be expanded with quotes and specifics in v0.2.)
