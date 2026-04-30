# 11 — Example: SvelteKit + Supabase

> **Status:** Outline.

Hardening checklist for the Grabit shape:
- RLS on every table; tenant_id as session var.
- Caddy adds security headers.
- fail2ban on the host (see topic 01 G-03).
- `app.locals.tenant` resolved server-side; never trust the client.
- Supabase service-role key only in `$lib/server/*` (SvelteKit
  enforces this).
