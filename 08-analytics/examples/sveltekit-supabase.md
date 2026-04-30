# 08 — Example: SvelteKit + Supabase

> **Status:** Outline.

The Grabit pattern: PostHog server-side + client-side. Customer-facing
reports are SvelteKit `+page.server.ts` loaders pulling from Supabase
views. PDF reports via Puppeteer in a separate Fly Machine (separate
because Puppeteer's 200MB+ Docker image bloats the SvelteKit deploy).

The Trackwell sister project uses **WeasyPrint** (Python) for PDF —
better quality for print-style reports.
