# 14 — Example: SvelteKit + Supabase

> **Status:** Outline.

Supabase migrations live in `supabase/migrations/*.sql`. Apply via
`supabase db push`. RLS policies in the same files. Backups via
Supabase Pro point-in-time recovery (7 days). Off-provider backup:
a weekly GitHub Action `pg_dump` to Cloudflare R2.
