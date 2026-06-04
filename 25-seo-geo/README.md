# 25 — SEO & GEO (AI Search)

> **Status:** v0.1 — distilled from four production projects
> (matekmegoldasok.hu / Deno Fresh, cutoptim.com / Astro, grabit.hu /
> Astro + SvelteKit, trackwell.eu / Astro). Every item below has a
> provenance: it was shipped, broke, or was audited in one of those repos.
>
> **Scope vs. [topic 17 — Marketing Site & SEO](../17-marketing-seo/):**
> topic 17 is about *building the marketing site* (framework, OG image
> pipeline, programmatic landing pages, blog, A/B tests, vs-competitor
> pages). This topic (25) is the cross-cutting *technical SEO + GEO
> discipline* — getting any page crawled, indexed, structured, and cited
> by AI. The two are siblings; build the site with 17, make it findable
> and citable with 25.

## What this topic covers

Two overlapping disciplines, one pipeline:

- **SEO (Search Engine Optimization)** — getting ranked and clicked in
  classic search (Google, Bing): crawlability, indexation, canonical/
  hreflang correctness, sitemaps, Core Web Vitals, meta tags.
- **GEO (Generative Engine Optimization)** — getting *cited* by answer
  engines (ChatGPT, Perplexity, Claude, Gemini, Google AI Overviews,
  Bing Copilot): AI-crawler access, structured data the model can ground
  on, `llms.txt`, citable passage structure, entity/brand authority.

They share ~70% of the plumbing. GEO is not a replacement for SEO — it
is SEO's foundation (crawl + index + structured data) plus a content and
entity layer optimized for *being quoted* rather than *being clicked*.

## The mental model

```
        ┌─────────────────────────────────────────────┐
        │  Can a bot reach it?   (robots, CSP, WAF)     │  ← if no, nothing else matters
        ├─────────────────────────────────────────────┤
        │  Can it index/dedupe it? (canonical,          │
        │  hreflang, sitemap, noindex hygiene)          │
        ├─────────────────────────────────────────────┤
        │  Can a machine understand it? (JSON-LD @graph,│
        │  semantic HTML, llms.txt)                     │
        ├─────────────────────────────────────────────┤
        │  Will an LLM quote it? (citable passages,     │
        │  FAQ/HowTo, comparison tables, definitions)   │
        ├─────────────────────────────────────────────┤
        │  Does it trust the source? (E-E-A-T: Person   │
        │  schema, sameAs, real reviews, Reddit/YT)     │
        └─────────────────────────────────────────────┘
```

Work top-down. A perfect `@graph` on a page Cloudflare challenges to
GPTBot is worthless. A 4.9★ `aggregateRating` with zero visible reviews
is a liability, not an asset.

## Files in this topic

- [`checklist.md`](checklist.md) — the full go-through, phase by phase.
  This is the artifact you run against a new project.
- [`best-practices.md`](best-practices.md) — the "why" and the patterns,
  with sources.
- [`gotchas.md`](gotchas.md) — real things that broke, written as
  situation → symptom → root cause → fix.
- [`examples/sveltekit-supabase.md`](examples/sveltekit-supabase.md) —
  concrete code for a multi-tenant SvelteKit + Astro-landing stack
  (the grabit.hu shape).
- [`schema-graph-playbook.md`](schema-graph-playbook.md) — the deep-dive
  for Phase 3: a battle-tested, stack-agnostic copy-paste **prompt** +
  **verification scanner** for consolidating scattered JSON-LD blocks
  into one connected `@graph` per page. Executed on a real 2-site Astro
  codebase (~450 pages, 0 regressions). Moved here from topic 17 so all
  structured-data work lives in one place.

## Scoring (how to know you're done)

A page-type is "GEO-complete" when all five layers above pass. A *site*
is GEO-complete when:

1. Every AI crawler gets HTTP 200 on the live origin (verified, not
   assumed — see G-01).
2. One `@graph` per page, zero dangling `@id` refs, validated by an
   automated scanner in CI.
3. Canonical = hreflang self-ref = sitemap loc, byte-for-byte.
4. `llms.txt` exists and matches current product reality (price,
   provider, feature counts).
5. Founder/author is a real `Person` node with a *resolving* `sameAs`.

Targets seen in practice: cutoptim went 56 → 64 GEO score over two
audits; the ceiling (75+) was gated entirely by **brand authority**
(external presence), not on-page work. Plumbing is the cheap 60 points;
the last 30 are Reddit/YouTube/reviews/press and take months.

## When to use

- New marketing site or content hub → run the whole checklist before launch.
- Multi-tenant SaaS → pay special attention to Phase 1 (indexation
  strategy: landing indexed, tenant subdomains `noindex`).
- Existing site → run Phase 9 (validation harness) first to find the
  drift, then work the failures.
