# 07 — Example: SvelteKit + Supabase

> **Status:** Outline.

The Grabit pattern: cron jobs are HTTP endpoints under `/api/cron/*`
hit by GitHub Actions schedule (free, generous limits). Each endpoint
checks a `CRON_SECRET` bearer header. Notifications written to
`notifications` table; bell icon component subscribes to Supabase
Realtime for the active user.
