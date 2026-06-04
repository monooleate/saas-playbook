# 17 — Marketing Site & SEO

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before public launch.
> **Cost:** 5–15 days for the initial site; ongoing.

## Scope

The marketing site is half the codebase for most SaaS, and the only
part Google ever sees. Topic 17 covers:

1. **Marketing site framework** — Astro, Next.js (separate workspace),
   Framer, Webflow.
2. **Static generation** — every public page pre-rendered for speed
   and SEO.
3. **OG image pipeline** — auto-generated social previews.
4. **Sitemap + robots.txt** — generated, kept in sync.
5. **Programmatic SEO** — landing page per use case, per location, per
   integration.
6. **Blog** — content marketing engine.
7. **A/B testing landing pages** — Posthog Experiments,
   GrowthBook, or Vercel's edge experiments.
8. **Vs. competitor pages** — `/grabit-vs-calendly` style.
9. **Public-facing case studies / testimonials**.

## Related

All technical SEO + GEO work (crawler access, structured data, `llms.txt`,
citability, E-E-A-T, validation) now lives in its own topic for clarity:

- [**25 — SEO & GEO (AI Search)**](../25-seo-geo/) — the checklist,
  gotchas, and the **[Unified JSON-LD `@graph` playbook](../25-seo-geo/schema-graph-playbook.md)**
  (battle-tested prompt + verification scanner for consolidating scattered
  structured-data blocks into one connected, AI-citable entity graph per
  page). This was previously hosted here; it moved to topic 25 so all
  structured-data material sits together.

Topic 17 stays focused on *building the marketing site*; topic 25 covers
*making it findable and citable*.

## Why it matters

For solo founders, organic search is the cheapest acquisition channel.
A great marketing site compounds for years; paid ads stop the moment
you stop paying.

The Grabit project's `apps/landing/` is an Astro static site with:
- ~30 pages: home, pricing, features, vs-competitor, blog, legal.
- OG images per page generated at build time.
- Sitemap.
- Lighthouse 95+ across all metrics.
- Built and deployed alongside the app, served by the same Caddy.

## Recommended stack for solo founders

- **Astro** — best DX for content-heavy static sites. Markdown +
  components. Ships near-zero JS.
- **Next.js (separate workspace)** — fine if you already use Next for
  the app and want shared components.
- **Framer / Webflow** — visual builders, fast for non-developers,
  costly at scale, opinionated.

## Programmatic SEO at solo scale

Pick 3 dimensions that matter to your customers (use case, industry,
location). Auto-generate landing pages from a JSON of variations.

Example: Grabit could generate `/booking-software-for-{barber|salon|
gym|coach}` landing pages from a single template + 4-row data file.
Each gets unique copy, screenshots, and CTAs.

Pitfall: thin content. Google penalizes "300 nearly-identical pages
with one word changed." The bar is unique value per page.

## What goes in v0.2

- Astro setup walkthrough (Grabit-style).
- OG image generation patterns (Astro / Next.js / serverless).
- Sitemap generation.
- Programmatic SEO templates.
- Blog engine choices.
- A/B testing landing pages.

## Sources

- Astro docs.
- Lemon Squeezy's [seo guide](https://www.lemonsqueezy.com/blog/seo-for-saas-founders).
- Vercel's blog on edge A/B testing.
- Programmatic SEO patterns from Pieter Levels' threads.
