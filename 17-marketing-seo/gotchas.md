# 17 — Marketing Site & SEO — Gotchas

> **Status:** Outline.

## Categories

1. **Soft 404s.** Astro `try_files` fallback to home page returns
   200 for missing paths. Cross-link to topic 01 G-04.
2. **Mixed deployment of marketing + app.** Marketing rebuild requires
   app rebuild. Fix: separate workspace, separate build trigger.
3. **OG images broken on social shares.** Cache invalidation;
   regenerate on every deploy.
4. **Sitemap stale.** Generated at build time; if pages change in DB
   (e.g. blog), needs runtime sitemap.
5. **Programmatic SEO too thin.** Google demotes; recovery takes months.
6. **`hreflang` misconfigured for multi-language.** Cross-link to
   topic 09.
7. **`robots.txt` blocks staging URL → blocks production by mistake.**
   Different env files.

(Real incidents in v0.2.)
