# 02 — Auth & Multi-tenancy — Example: Remix + PlanetScale

> **Status:** Outline. Full implementation forthcoming.

## What will go here

Remix has no Auth.js equivalent baked in; the typical pattern is
[remix-auth](https://github.com/sergiodxa/remix-auth) with strategy
modules per provider.

Tenant routing on Fly.io: the request reaches the app intact (no
middleware layer between Fly and your Remix server), so resolution
happens in a Remix loader — typically in a top-level `app/root.tsx`
loader that reads `request.headers.get('host')`.

PlanetScale's no-foreign-key constraint affects auth tables:
`account.user_id` must be guarded application-side, not by the DB.

The example will include:

1. `app/lib/auth.server.ts` with `Authenticator` configured for
   email-magic-link and Google OAuth strategies.
2. `app/lib/tenant.server.ts` with `tenantFromRequest()`.
3. Drizzle schema for users, accounts, sessions, tenants, memberships
   (no FK constraints).
4. Application-side join guards (the ORM-side equivalent of RLS).
5. A typed `requireTenant(request)` helper used at the top of every
   loader.

## Stand-in references for now

- remix-auth: [github.com/sergiodxa/remix-auth](https://github.com/sergiodxa/remix-auth).
- PlanetScale + foreign keys:
  [planetscale.com/docs/learn/operating-without-foreign-key-constraints](https://planetscale.com/docs/learn/operating-without-foreign-key-constraints).
