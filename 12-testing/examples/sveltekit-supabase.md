# 12 — Example: SvelteKit + Supabase

> **Status:** Outline.

Vitest workspace at root with two projects: `billing` (unit tests for
pricing math) and `app` (Supabase integration tests against a local
Supabase Docker stack). Playwright in `apps/app/playwright/` running
against `pnpm preview`. CI cache: `pnpm-lock.yaml` + `~/.local/share/pnpm/store`.
