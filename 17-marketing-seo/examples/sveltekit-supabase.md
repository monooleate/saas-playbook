# 17 — Example: SvelteKit + Supabase (with Astro marketing)

> **Status:** Outline.

The Grabit shape: SvelteKit at `apps/app`, Astro static at
`apps/landing`. Caddy serves Astro `dist/` directly with cache headers
(see topic 01 example). Astro `astro:integrations` for sitemap, OG
images, RSS. Programmatic SEO via content collections + dynamic
routes generating ~30 pages from a JSON list.
