# 02 — Auth & Multi-tenancy — Example: SvelteKit + Supabase

> **Status:** Outline. The Grabit reference implementation is the source
> material; full code will be ported here in v0.2.

## What will go here

The Grabit project resolves tenants in `apps/app/src/hooks.server.ts`
based on the `Host` header (e.g. `tenant.example.com`), looks up the
tenant in Supabase by subdomain slug, and attaches it to `event.locals`.
Every server load + action then has `locals.tenant`, `locals.user`,
`locals.session`.

Authentication uses **Supabase Auth** (built into the database) — email
+ password, magic link, OAuth (Google + Facebook are commonly enabled).
Sessions are stored in `httpOnly` cookies.

Multi-tenancy is enforced via **Postgres RLS policies**: every business
table has a policy like
`USING (tenant_id = current_setting('app.tenant_id')::uuid)`, and the
hook sets `app.tenant_id` per request via `set_config()`.

This example will include:

1. The full `hooks.server.ts` with subdomain → tenant resolution.
2. `apps/app/src/lib/server/supabase.ts` — admin client for server-only
   operations.
3. RLS policies for the canonical tables.
4. The "send magic link" + "verify session" flow.
5. The membership table + invite flow.
6. The "switch tenant" UI for multi-membership users.

## Stand-in references for now

- Supabase Auth quickstart for SvelteKit:
  [supabase.com/docs/guides/auth/quickstarts/sveltekit](https://supabase.com/docs/guides/auth/quickstarts/sveltekit)
- Supabase RLS:
  [supabase.com/docs/guides/auth/row-level-security](https://supabase.com/docs/guides/auth/row-level-security)
