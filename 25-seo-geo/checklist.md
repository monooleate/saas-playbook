# 16 — SEO & GEO — Checklist

> Run this top-to-bottom on a new project, or jump to a phase to fix a
> known gap. Each item is a binary check. Provenance tags point to the
> project where the lesson was paid for: **[MM]** matekmegoldasok,
> **[CO]** cutoptim, **[GR]** grabit, **[TW]** trackwell.
>
> Legend: `[ ]` to do · `[~]` optional / context-dependent · `[!]`
> high-impact, do not skip.

---

## Phase 0 — Rendering & framework foundation

The single biggest GEO lever is **does the bot see content without
running JS**. AI crawlers (GPTBot, ClaudeBot, PerplexityBot) have weaker
JS engines than Googlebot.

- `[!]` Content is in the **server-rendered HTML**, not injected
  client-side. SSG or SSR, not pure CSR. **[CO]** Astro SSG scored a
  perfect 100/100 technical because the full rendered content ships on
  first fetch.
- `[ ]` Interactive widgets (calculators, islands) are **progressive
  enhancement** — the surrounding article/FAQ/schema renders server-side
  even if the island never hydrates. **[MM]** Preact islands; the
  KaTeX/FAQ/schema is all SSR.
- `[ ]` Math/formula content uses a render that emits real markup
  (KaTeX/MathML), not an image or a `<canvas>`. **[MM]** `\[ ... \]`
  KaTeX so the LLM ingests the formula, not OCR of a picture.
- `[~]` If the app *must* be CSR (dashboard, authed area), it is excluded
  from indexing anyway (Phase 1) — don't waste effort SSR-ing it.

---

## Phase 1 — Crawl & index access

### 1.1 robots.txt — explicit AI-crawler allowlist

- `[!]` `robots.txt` exists and declares the sitemap.
- `[!]` Every answer-engine crawler is listed **by name** with `Allow:
  /`. Do not rely on `User-agent: *` alone — some bots have quirky UA
  routing and the explicit list documents intent. **[CO]** `OAI-SearchBot`
  was missing initially and had to be added; it is *not* covered by the
  generic GPTBot entry.
  - OpenAI: `GPTBot`, `ChatGPT-User`, `OAI-SearchBot`
  - Anthropic: `ClaudeBot`
  - Perplexity: `PerplexityBot`
  - Google AI: `Google-Extended`
  - Apple: `Applebot-Extended`
  - Others: `cohere-ai`, `CCBot`, `Bytespider`, `Amazonbot`, `FacebookBot`
- `[ ]` Private/thin routes are `Disallow`ed: `/api/`, `/auth/`,
  `/login`, `/register`, `/admin`, `/dashboard`, `/cancel/`, framework
  internals (`/_fresh/`, `/_astro/` if not asset-needed). **[MM][GR]**

### 1.2 Multi-tenant indexation strategy (if applicable)

- `[!]` Marketing/landing host is indexed; tenant subdomains/paths are
  **not**. Mixing them = duplicate-content across N near-identical tenant
  sites + leaking private booking/customer data. **[GR]**
- `[ ]` Tenant `noindex` uses **all four layers** (one is not enough —
  Google trusts the header most, robots.txt only controls crawl budget):
  1. `robots.txt` on the tenant host → `Disallow: /`
  2. `X-Robots-Tag: noindex, nofollow` HTTP header (set in
     middleware/hooks when a tenant context is detected) **[GR]**
  3. `<meta name="robots" content="noindex, nofollow">` HTML fallback
  4. Routing/edge rules keeping auth pages off the public surface
- `[~]` Tenant discovery, if wanted, lives as **structured embeds on the
  indexed marketplace page** (LocalBusiness/Service nodes on
  `/discover/...`), not by indexing tenant domains. **[GR]**

### 1.3 CSP / WAF must not block bots

- `[!]` Verify a strict CSP does not break crawler rendering and that
  the CDN/WAF (Cloudflare) is **not** serving a JS challenge/Turnstile
  to bot user-agents. This is a real GEO killer that hides behind
  "everything looks fine in my browser." **[MM]**
- `[ ]` CSP whitelists every domain the page needs to render
  (KaTeX CDN, fonts, payment iframe, analytics) — an over-tight
  `script-src`/`connect-src` can blank the page for a stricter engine.
  **[MM]** AdSense/Paddle/ProfitWell/Google-Fonts all had to be added.

### 1.4 Verify access live (do not assume)

- `[!]` Curl the **production** origin with each bot UA and assert HTTP
  200 + full HTML + zero challenge markers. **[MM]**
  ```bash
  for ua in "GPTBot/1.1" "ClaudeBot/1.0" "PerplexityBot/1.0" \
            "OAI-SearchBot/1.0" "Google-Extended"; do
    code=$(curl -s -o /tmp/body -w '%{http_code}' -A "$ua" https://EXAMPLE.com/)
    grep -qi "just a moment\|turnstile\|cf-mitigated" /tmp/body \
      && echo "$ua: CHALLENGED" || echo "$ua: $code OK ($(wc -c </tmp/body) bytes)"
  done
  ```
- `[ ]` Repeat for a deep content page, not just the homepage.

---

## Phase 2 — Sitemap, canonical, hreflang (the triplet)

> These three must agree **byte-for-byte**. The single largest source of
> Google Search Console errors across all four projects. **[CO][TW]**

### 2.1 Trailing-slash policy is one decision, applied everywhere

- `[!]` Pick `trailingSlash: 'never'` (or `'always'`) once in framework
  config, and make the canonical builder + hreflang builder + sitemap
  emit the **same** form. **[CO][TW]** cutoptim: server 301'd to
  trailing-slash but canonical omitted it → duplicate-content. trackwell:
  canonical `…eu/` but sitemap `…eu` → homepage dupe.
- `[ ]` Canonical and hreflang self-ref are generated from the **same
  function** as the route/URL, not from `request.url`/`pathname`
  independently. **[TW]** A shared `route(key, lang, params)` builder
  guarantees identical bytes.

### 2.2 Sitemap hygiene

- `[!]` Sitemap exists, is reachable (no 403/500 on the sitemap URL
  itself), and is declared in robots.txt.
- `[ ]` `lastmod` is real (from content `refreshed_at`/`published_at`),
  not omitted and not "now" on every URL. Real lastmod = better crawl
  budget allocation. **[MM][CO]** cutoptim shipped 221 URLs with no
  lastmod, then added them in v2.
- `[ ]` `priority`/`changefreq` reflect reality (e.g. calculators
  `weekly`/0.9, evergreen articles `monthly`/0.8, thin pages 0.4). **[MM]**
- `[!]` **No `noindex` URL appears in the sitemap.** Contradictory signal.
  And inversely: nothing in the sitemap is `noindex`. **[TW]** the thin
  `/shop-success` page was `noindex` *and* in the sitemap — filter it out
  for all locales.
- `[~]` If you generate `sitemap-index.xml` + `sitemap-0.xml`, add a
  **301** from the conventional `/sitemap.xml` → index (not a static
  file — the sitemap spec forbids an index referencing another index).
  **[TW]**

### 2.3 hreflang (multilingual sites)

- `[!]` hreflang cluster is **mutual** (EN→DE means DE→EN) and only
  points to pages that **actually exist**. Incomplete/dangling hreflang
  is worse than none — it confuses Google. **[CO]** Guides existed only
  in EN/DE/HU, so IT/FR hreflang was *omitted* until translated, then
  added retroactively.
- `[ ]` `x-default` points to the canonical default-language home (EN).
  **[TW]**
- `[ ]` URL structure is consistent across languages (`/de/guides/…`,
  `/it/guides/…`) — resist language-native paths
  (`/de/ratgeber/`) that don't scale to N locales. **[CO]**
- `[~]` Root domain (`/`) serves the **largest commercial market**, not
  the founder's native language. **[CO]** moved root from HU to EN; HU
  dropped to `/hu/` with no ranking loss.

### 2.4 Host + protocol canonicalization (not just the slash)

- `[!]` Pick one canonical **host** (apex *or* `www`) and **301** the
  other. Serving `example.eu, www.example.eu` from one server block returns
  200 on **both** → duplicate hosts the canonical tag only mitigates.
  **[OX]** A top-level `redir @www https://example.eu{uri} permanent`
  (Caddy) runs before the route handlers, so it also covers app/login paths.
- `[!]` Force **https**: 301 http→https (automatic with Caddy/Let's
  Encrypt; assert it on other stacks). End state: one host, one protocol,
  one slash policy — every other variant 301s to the canonical URL.

---

## Phase 2.5 — Social sharing, PWA chrome & custom error pages

> Not "crawl/index/schema", but every audit checks them and they're cheap,
> high-visibility wins. A perfectly indexed page can still unfurl as a bare
> text stub or serve a soft-404. **[OX]**

### 2.5.1 Open Graph + Twitter Card

- `[!]` `og:title`, `og:description`, `og:url`, `og:type`, `og:site_name`,
  and an **absolute** `og:image` URL — social scrapers fetch the page in
  isolation, so a root-relative path fails. **[OX]**
- `[!]` `twitter:card=summary_large_image` + `twitter:image` (absolute). X
  does **not** render SVG — the image must be a raster (PNG/JPG). **[OX]**
- `[ ]` `og:image:width`/`:height`/`:type`/`:alt` set; `og:title` ~40–60
  chars, `og:description` ~110–160 (reuse the page `<title>`/meta where
  honest). **[OX]**
- `[ ]` One **1200×630** share image (1.91:1, <5 MB), **generated from the
  vector brand source** (e.g. `sharp` rasterizing an SVG) at build time —
  on-brand, versioned, not a hand-export. Doubles as the SERP/LLM visual
  (Phase 7). **[OX][MM][TW]**

### 2.5.2 PWA / favicon / icon set

- `[ ]` `manifest` linked (name, `theme_color`, `background_color`, icons
  192 + 512, `display`). **[OX]**
- `[ ]` `apple-touch-icon` (180, full-bleed) + favicon set: SVG **and** a
  PNG-based `favicon.ico` (16/32) — auditors and legacy clients probe the
  raster even though modern browsers accept SVG. **[OX]**
- `[ ]` `theme-color` (+ `apple-mobile-web-app-*` if installable). **[CO]**

### 2.5.3 Custom error pages (correct status)

- `[!]` Branded `404` and `5xx` pages served with the **real status code**
  — a custom page returned as HTTP 200 is a **soft-404** (read as
  duplicate/low-quality). **[OX]**
- `[~]` On static hosts, author them as standalone files (`public/404.html`
  / `500.html`, inline CSS/JS, no pipeline dependency) and serve via the
  error handler. Caddy `handle_errors { root *…; rewrite @e4xx /404.html;
  file_server }` **preserves** the status. Astro caveat: `404.astro` +
  `trailingSlash:'always'` builds `404/index.html`, not `404.html` — a
  `public/*.html` file sidesteps it. **[OX]**
- `[~]` The 5xx page also catches a downstream `reverse_proxy` 502/503 (app
  down) → a branded page instead of a stack stub. **[OX]**

---

## Phase 3 — Structured data (JSON-LD `@graph`)

> Target architecture, proven identically in three repos **[MM][CO][GR]**:
> **one** `<script type="application/ld+json" id="graph">` per page, a
> self-contained `@graph` array, stable `@id`s, zero dangling refs.

### 3.1 Architecture

- `[!]` One `@graph` block per page. **Not** 3–8 disconnected
  `<script>` blocks. Multiple inline blocks create competing,
  un-dedupable entities (e.g. three different `SoftwareApplication`
  nodes with conflicting category/currency). **[CO][MM]**
- `[!]` Stable, canonical `@id`s, referenced not duplicated:
  - `${siteUrl}/#organization`
  - `${siteUrl}/#website`
  - `${siteUrl}/#founder` (or a network-level `https://person.dev/#person`
    if the founder spans multiple brands — decide once) **[MM][CO][GR]**
  - per-page: `${pageUrl}#webpage`, `#breadcrumb`, `#faq`, `#article`,
    `#calculator`/`#app`, `#product`, `#howto`, `#itemlist`
- `[ ]` Schema is generated from one builder (`lib/schema.ts` /
  `schema-graph.ts`) at the render layer, fed by frontmatter — **never**
  hand-authored per page. Manual schema across 100+ pages guarantees
  drift. **[CO][MM][GR]**
- `[ ]` Frontmatter is schema-validated at build (Zod for Astro content
  collections, YAML parse for Deno) so a page can't publish with missing
  required fields. **[CO]**

### 3.2 Node coverage by page type

- `[ ]` **Every** page: `WebPage` + `WebSite` + `Organization`
  (+ `BreadcrumbList` where there's a hierarchy). **[GR]** after refactor:
  Organization/WebSite/Person went from partial to all 49 pages.
- `[ ]` Articles/guides: `Article`/`TechArticle`/`MedicalWebPage` with
  `author: {@id: #founder}` (a **Person**, not the Organization),
  `publisher`, `datePublished`, `dateModified`. **[TW][GR]**
- `[ ]` Calculators/tools: `SoftwareApplication` with
  `applicationCategory`, `offers` (even free → `price: "0"`,
  `isAccessibleForFree: true` to clear the rich-result warning),
  `featureList`. **[MM]** Domain tools may also warrant a domain schema
  (e.g. `MedicalRiskCalculator` **[TW]**) — emit *both*, Google renders
  off `SoftwareApplication`.
- `[ ]` FAQ blocks: `FAQPage` whose `mainEntity` is generated from the
  **same** data as the visible on-page FAQ (DRY — never two sources).
  **[TW][MM]**
- `[ ]` Pricing page: `Product` with `offers`/`AggregateOffer` covering
  **all** tiers, currency from locale. **[CO]** Business tier ($19) was
  missing from `offers` — only Free/Pro were visible to engines.
- `[ ]` Contact page: `ContactPage`. **[GR]**

### 3.3 Correctness rules

- `[!]` **Zero dangling `@id` references** — every `isPartOf`/`author`/
  `publisher`/`about` ref resolves to a node present in the graph.
  **[MM]** subpages' `isPartOf` pointed at the homepage `#offer-catalog`
  → dangling; fixed to each page's own `#webpage`.
- `[ ]` Single `@context` at graph root (strip per-node `@context`). **[MM]**
- `[ ]` `inLanguage` uses BCP-47 (`hu-HU`, `de-DE`) and matches the
  page's `<html lang>` / hreflang. **[CO]**
- `[!]` Escape non-ASCII in JSON-LD as `\uXXXX` to survive charset bugs
  downstream. **[CO]** "János Mészáros" rendered as mojibake in some
  crawlers until escaped.
- `[ ]` Ratings are deterministic if synthesized at all (see Phase 5.3
  on the policy risk) — a hash-derived value is stable across crawls so
  it doesn't look manipulated. **[MM]** FNV-1a from slug.

### 3.4 Cross-site / network entity linking (if you run multiple brands)

- `[~]` Sister brands share one founder `Person` `@id` across domains
  (entity grounding for E-E-A-T), linked via `founder` — **not** `sameAs`
  (different brands are not the same entity). **[MM]** matekmegoldasok +
  instrumenteonline.ro + konvertalo.hu share `#founder`.

---

## Phase 4 — Content GEO (citability)

> Goal: an LLM lifts your passage **verbatim** as the answer and cites
> you as the source. Short, self-contained, metric-rich, structured.

- `[!]` H1 and H2/H3 are **questions** in the user's words ("What is
  X?", "How do I calculate Y?"). FAQ questions live in **heading tags**
  (`<h3>`), not just `<summary>`/`<dt>` — stronger snippet/citation
  signal. **[CO][TW][MM]**
- `[!]` Each section opens with a **direct 1–2 sentence answer**, then
  the deep dive (lists, stats, tables). No throat-clearing preamble.
  **[TW]** medical blog pattern: question-H2 → direct answer → detail →
  red-flags → free-vs-paid.
- `[ ]` Self-contained **definition blocks** using `<dl>/<dt>/<dd>` so
  the model quotes the definition and names you as source. **[CO]**
  homepage glossary (kerf, yield, nesting…) — goal is "CutOptim is the
  *source* of the definition," not just a tool mentioned.
- `[!]` **Comparison tables** (your product × competitors, 6–8
  attributes) and **before/after metrics with numbers** ("30% waste →
  7%, €X saved"). These are the single most AI-cited content shapes —
  AI Overviews and Perplexity lift them directly. **[CO]**
  - `[~]` Add a methodology/disclosure line to competitor comparisons
    ("based on our testing on YYYY-MM-DD") — engines flag undisclosed
    comparisons as misleading. **[CO]**
- `[ ]` Content length matches intent: ~480–650 words of dense,
  structured text beats a 2000-word essay for citation; **<400 words is
  too thin**. **[TW]** shift articles were 282–390 words → expanded to
  ≥630.
- `[ ]` Cross-link ≥2 related pages by **entity** (article ↔ its
  calculator, subcategory landing ↔ its articles) so the graph connects
  related answers. **[MM]**
- `[~]` `speakable` (`SpeakableSpecification`, cssSelector on h1 / FAQ
  dt/dd) for voice-assistant citation. Low text-search impact. **[CO]**
- `[ ]` **Wedge positioning** for content topics: target specific
  long-tail queries where you can win, not the generic head term where
  incumbents/apps dominate. **[TW]** "menstrual migraine diary" + non-
  English EU markets, not "migraine tracker." AI engines reward the
  narrow, deep page on specific questions.

---

## Phase 5 — E-E-A-T & brand authority

> On-page plumbing caps at ~60–75 of a 100 GEO score. The ceiling is
> **external presence** and it takes months. **[CO]**

### 5.1 Person / author (cheap, high impact)

- `[!]` Author/founder is a standalone `Person` node (`@id: #founder`),
  and article `author` points to it — **not** to the Organization.
  **[GR]** a one-line `Organization → Person` swap was scored +7 E-E-A-T.
- `[ ]` `Person` carries credentials: `jobTitle`, `alumniOf`,
  `knowsAbout: [...]`, and a **resolving** `sameAs` (LinkedIn, GitHub,
  personal site). **[CO]**
- `[ ]` Visible author byline on guide pages (name, ideally photo +
  credentials), and an `/about` link in the **top nav**, not just footer
  — a Google quality-rater signal. **[CO]**

### 5.2 Organization sameAs / entity recognition

- `[!]` `Organization.sameAs` is populated and **every link resolves**.
  An empty `sameAs` makes you invisible to knowledge graphs; a **broken
  (404) sameAs link is worse than none** — it actively hurts entity
  recognition. **[CO]** the LinkedIn link in `sameAs` was a 404.
- `[~]` Long-term: a Wikidata Q-ID becomes the canonical entity
  identifier across all AI knowledge graphs — but it requires notability
  first (press, awards), so it's a Phase-2+ goal. **[CO]**

### 5.3 Reviews / ratings policy (legal + spam risk)

- `[!]` **Do not assert `aggregateRating` without visible, verifiable
  on-page reviews.** Fabricated ratings/testimonials are a Google
  Structured-Data spam-policy violation (manual action, rich-snippet
  removal) and, in the EU, an unfair-commercial-practice exposure.
  **[CO][GR][TW]** all three carried fabricated ratings as a *documented,
  accepted risk* — know you're doing it, don't do it blind.
- `[ ]` Prefer **external review platforms** (G2, Capterra, Trustpilot,
  Product Hunt) — verifiable by crawlers — over an unverified on-site
  number. A verified 4.7 on G2 beats a claimed 4.9 on your own page. **[CO]**
- `[~]` Pre-traction honest framing ("0/12 beta users — join") beats
  fake social proof; it reads as trustworthy to both raters and LLMs.
  **[GR][TW]**

### 5.4 Community & ecosystem presence (the slow 30 points)

- `[~]` Reddit: an honest founder post (with proof, e.g. a results
  screenshot) in relevant subreddits. AI assistants heavily cite Reddit;
  one upvoted thread > 10 site pages for entity grounding. **[CO]**
- `[~]` YouTube: tutorial/demo videos with `VideoObject` schema — the
  strongest *Gemini / AI-Overviews* signal (Google owns both). **[CO]**
- `[~]` LinkedIn company page + (for local) Google Business Profile,
  linked from `sameAs`. **[CO]**

---

## Phase 6 — Internationalization (multilingual sites)

- `[!]` hreflang correctness → covered in Phase 2.3.
- `[ ]` Localized routes have **real static pages**, not a sidebar
  language toggle over one URL. **[TW]** 8 × {home, landing, sample,
  blog index, legal} page-wrappers generated.
- `[ ]` Localization ≠ translation: localized content reflects local
  market reality — standard sizes, VAT, currency, **native terminology**
  and **local keyword research**, not a literal translation. **[CO]** DE
  woodworkers use different stock dimensions than IT ones.
- `[ ]` Meta descriptions are trimmed **per language** (DE/IT/FR run
  longer than EN for the same meaning) to stay under the SERP cutoff.
  **[CO]**
- `[ ]` Translation bundles pass a **structural parity check** (every
  key present, every array same length across all locales) in CI — hand
  translation drifts silently. **[TW]** Python parity check, 0 missing /
  0 extra.
- `[~]` Placeholder locales may EN-fallback to ship, but track them —
  fallback `title`/`description`/OG must not stay English once the
  locale goes live. **[TW]**

---

## Phase 7 — Performance, mobile, accessibility

- `[ ]` Core Web Vitals baseline: explicit `width`/`height` on images
  (no CLS), font `preload` + `font-display: swap` (no FOIT), analytics
  loaded `async` (not render-blocking). **[CO]** Note: **INP** has
  replaced FID as the responsiveness metric.
- `[ ]` OG hero image per content type, 1200×630, ≤300KB (WebP/SVG). It
  doubles as the visual an LLM/SERP can surface. **[MM][TW]**
- `[ ]` Mobile: usable at 375px, KaTeX/tables `overflow-x` scroll, dark
  mode correct. **[MM]**
- `[ ]` Accessibility overlaps SEO: heading hierarchy (h1→h6 no skips),
  `<label for>`, semantic `<nav>/<article>/<aside>`, skip-to-content
  link, `aria-label` on icon buttons (alt-text equivalent),
  `prefers-reduced-motion`. WCAG failures often = structured-data /
  parsing failures. **[CO][GR]**
- `[ ]` PWA/mobile meta present (`theme-color`, `apple-mobile-web-app-*`)
  if it's an installable tool. **[CO]** (Full PWA/icon set in Phase 2.5.)
- `[ ]` Eliminate **render-blocking resources**: inline critical CSS (on a
  lean site, inline *all* page CSS so first paint needs no extra round-trip
  — Astro `build.inlineStylesheets:'always'`), defer/`async` non-critical
  JS. LCP/CLS/INP are ranking inputs and real UX. **[OX]**
- `[~]` Security headers **delivered** (verify with curl, not config — see
  G-01): HSTS (`Strict-Transport-Security`) on https-only sites; ideally
  also CSP, `X-Content-Type-Options: nosniff`, `Referrer-Policy`. **[CO][OX]**

---

## Phase 8 — llms.txt & machine feeds

- `[!]` `/llms.txt` exists: a human+machine-readable site summary —
  H1 (what the product is), a ≤140-char blockquote for model context,
  then sections (Key facts, top 10–20 pages as `[name](url): desc`,
  pricing, legal, contact). **[MM][CO][GR][TW]**
- `[ ]` `llms.txt` numbers are **concrete** ("51+ calculators", "500+
  worked examples") — better LLM grounding than vague prose. **[MM]**
- `[!]` Treat `llms.txt` as **live marketing content under version
  control**: it drifts. Re-check on every pricing/provider/feature
  change. **[CO]** llms.txt said "Stripe" months after migrating to Lemon
  Squeezy, wrong free-tier limits, stale "last updated" date.
- `[~]` `llms-full.txt` — fuller content dump of the top pages so models
  can ingest without crawling each. **[MM]** (602 lines) **[CO]** planned.
- `[~]` Pull dynamic facts (current price) into `llms.txt` from the same
  SSOT as the page, not a hardcoded copy. **[TW]**
- `[~]` Bing: register Bing Webmaster Tools + implement **IndexNow**
  (push notifications → indexing in minutes vs 24h). Powers Bing Copilot,
  often the lowest-scoring platform. **[CO]**
- `[~]` Dedicated machine feed (`/feed.xml` RSS/Atom, or a custom
  `/nlweb-feed.json` with `name/url/whatItComputes/inputs/outputs/
  examples` per entity) if you target agent ingestion (NLWeb/MCP).
  **[MM]** RSS was a 404; the SSOT (`calculators/hu.json`) can generate
  the feed since it's already single-source.

---

## Phase 9 — Validation & QA harness (build this once, run forever)

- `[!]` **Automated JSON-LD scanner** in CI over the built `dist/`,
  asserting: exactly 1 `id="graph"` per page; 0 legacy `ld+json` blocks;
  graph parses; **0 dangling `@id` refs**; Person/Org/WebSite/WebPage
  present; `Person.@id` == constant; `inLanguage` == page lang; and a
  **regression baseline** (FAQ count, breadcrumb count, offers,
  aggregateRating, speakable, featureList never silently drop). **[MM]**
  109 PASS/0 FAIL · **[CO]** `scripts/scan-jsonld.mjs` + baseline ·
  **[GR]** 51 pages, before/after node counts never decrease.
- `[!]` Google **Rich Results Test** (`search.google.com/test/rich-
  results`) on each schema type (Article, FAQPage, SoftwareApplication,
  Product, BreadcrumbList) — local mock validators don't catch what the
  real one does. **[MM]** 74 valid items / 0 errors after launch.
- `[ ]` GSC checks post-deploy: hreflang report (catches syntax errors
  immediately), "submitted URL not found" (routing bugs), sitemap
  fetchable. **[CO]**
- `[ ]` Verify **security headers are actually delivered** in production
  (curl), not just present in `netlify.toml`/config — config ≠ behavior.
  **[CO]** netlify.toml declared CSP/X-Frame/Referrer-Policy; live
  response was missing all three.
- `[ ]` Title ≤ ~59 chars, meta description ≤ ~155 chars, single brand
  separator convention (`|`), brand suffix once, **no year in meta**
  (looks fresh today, stale in 12 months — years belong in body copy).
  **[GR][TW]**
- `[ ]` Visible-page ↔ schema parity: every price, rating, FAQ answer in
  schema also appears on the rendered page (Google rejects schema that
  doesn't match visible content). **[TW][CO]**
- `[ ]` Drift check (cron/CI) across SSOT ↔ payment catalog ↔ asset
  store, so schema/llms.txt prices can't silently diverge. **[TW]**

---

## Phase 10 — Maintenance & drift watch

> GEO/SEO is not ship-once. These bite weeks later.

- `[ ]` **llms.txt drift** — re-audit on every product change (Phase 8). **[CO]**
- `[ ]` **Date-driven content** — seasonal CTAs/banners auto-expire
  correctly; a "deadline approaching" banner shown after the deadline
  misleads users. **[MM]** the érettségi banner ran 4 weeks past the exam
  because `endDate` was wrong.
- `[ ]` **Rating/review policy** — revisit fabricated-rating risk; swap
  to real/external reviews as traction arrives (Phase 5.3). **[CO][GR][TW]**
- `[ ]` **hreflang backfill** — when a page gets a new translation, add
  its hreflang retroactively to all sibling languages (Phase 2.3). **[CO]**
- `[ ]` **New crawler UAs** — the AI-crawler list grows; revisit
  robots.txt periodically (Phase 1.1).
- `[ ]` **Re-score quarterly** — track the GEO score trend and which
  lever moved it. Expect plumbing to plateau and brand authority to be
  the long pole. **[CO]** 56 → 64 in two audits; ceiling gated by
  external presence.

---

## The 10-minute triage (existing site, where to start)

1. Curl the prod origin with 3 bot UAs → any challenge/non-200? Fix
   Phase 1 first, nothing else matters. **[MM]**
2. View-source one content page → is the content there without JS, and is
   there exactly one `@graph`? → Phase 0 / Phase 3.
3. Diff canonical vs hreflang-self vs sitemap-loc for one URL → Phase 2.
4. Open `/llms.txt` → exists? prices current? → Phase 8.
5. Check `Organization.sameAs` links resolve and author is a `Person` →
   Phase 5.
