# 16 — SEO & GEO — Gotchas

> Real things that broke or were caught in audit, written as
> situation → symptom → root cause → fix. Provenance: **[MM]**
> matekmegoldasok (Deno Fresh), **[CO]** cutoptim (Astro), **[GR]**
> grabit (Astro + SvelteKit), **[TW]** trackwell (Astro), **[OX]** operex
> (SvelteKit app + Astro landing, self-hosted Caddy/Hetzner).

## G-01 — Security headers configured but not delivered **[CO]**

- **Situation:** `netlify.toml` declared CSP, X-Frame-Options,
  Referrer-Policy, Permissions-Policy.
- **Symptom:** GEO audit's live curl showed all of them missing in
  production. 30-point security penalty.
- **Root cause:** Config ≠ delivery — Netlify plugin precedence / redirect
  caching silently dropped the header block.
- **Fix:** Verify headers in staging/production with curl, not by reading
  the config. Treat the live response as the source of truth.

## G-02 — Canonical / sitemap trailing-slash mismatch (homepage dupe) **[TW][CO]**

- **Situation:** EN root canonical/hreflang emitted `https://site.eu/`
  (trailing slash); the sitemap emitted `https://site.eu` (none). cutoptim
  had the inverse: server 301'd *to* a trailing slash, canonical omitted it.
- **Symptom:** GSC duplicate-content / "Google chose a different
  canonical."
- **Root cause:** Trailing-slash policy not unified across config,
  URL builder, and sitemap.
- **Fix:** `trailingSlash: 'never'` in config + a single `abs()`/`route()`
  helper that all three (canonical, hreflang, sitemap) call.

## G-03 — noindex page sitting in the sitemap **[TW]**

- **Situation:** Thin `/shop-success` "thank you" page was `noindex,
  nofollow` but still listed in the generated sitemap (all 8 locales).
- **Symptom:** Contradictory signal to GSC (crawl this / don't index this).
- **Root cause:** `@astrojs/sitemap` includes every route by default; the
  noindex was added separately without a sitemap filter.
- **Fix:** Sitemap `filter` excludes every localized `/shop-success*`
  slug. Rule: nothing noindex in the sitemap, nothing in the sitemap
  noindex.

## G-04 — OAI-SearchBot not covered by the GPTBot entry **[CO]**

- **Situation:** robots.txt allowed `GPTBot` and assumed OpenAI was
  covered.
- **Symptom:** ChatGPT platform GEO score stuck low; the dedicated search
  crawler wasn't explicitly allowed.
- **Root cause:** OpenAI uses **separate** user-agents for training
  (`GPTBot`), the live ChatGPT browser (`ChatGPT-User`), and search
  (`OAI-SearchBot`). One entry doesn't cover the others.
- **Fix:** List all answer-engine crawlers by name. ChatGPT platform
  score jumped after adding `OAI-SearchBot` + `llms.txt`.

## G-05 — Three competing SoftwareApplication nodes on one page **[CO][MM]**

- **Situation:** BaseLayout emitted a default `SoftwareApplication`, the
  pricing page emitted its own (hardcoded `€6`), each calculator emitted
  another — 2–3 per page with conflicting `applicationCategory` and
  currency (EUR vs USD).
- **Symptom:** No canonical entity; crawlers can't dedupe ambiguous inline
  objects. Graph score 3/10.
- **Root cause:** Inline schema duplication instead of `@id` references.
- **Fix:** One canonical `SoftwareApplication` (`@id: #software`) in a
  unified `@graph`, all pricing tiers in its `offers` array, referenced
  elsewhere by `@id`.

## G-06 — Dangling `@id` reference after graph migration **[MM]**

- **Situation:** Subpages' `SoftwareApplication.isPartOf` pointed at the
  homepage's `#offer-catalog`.
- **Symptom:** The JSON-LD scanner flagged a reference resolving to no
  node in the page's own graph.
- **Root cause:** A node `@id` valid on the homepage doesn't exist in a
  subpage's self-contained `@graph`.
- **Fix:** Render layer rewrites `isPartOf` to each page's own
  `#webpage`. Scanner asserts 0 dangling refs forever after.

## G-07 — Person name mojibake in JSON-LD **[CO]**

- **Situation:** `Person.name` = "János Mészáros" emitted as raw UTF-8.
- **Symptom:** Some crawlers showed "JÃ¡nos MÃ©szÃ¡ros."
- **Root cause:** Charset/encoding handling downstream of the JSON
  serializer is unreliable.
- **Fix:** `serializeGraph()` escapes all non-ASCII as `\uXXXX`
  (`á` for á) → 7-bit-clean JSON, immune to charset bugs.

## G-08 — Fabricated aggregateRating with zero real reviews **[CO][GR][TW]**

- **Situation:** `aggregateRating: { ratingValue: 4.8, reviewCount:
  91–127 }` on pages of products with 0 paying customers and no on-page
  review surface.
- **Symptom:** Eligible for a Google manual action (structured-data spam),
  rich-snippet removal, and — in the EU — an unfair-commercial-practice
  exposure.
- **Root cause:** Asserting ratings as a growth shortcut before any real
  reviews exist.
- **Fix (recommended):** Remove, or reframe honestly ("beta — join the
  first users"), or route reviewers to external platforms (G2/Trustpilot)
  whose ratings crawlers can verify. All three repos kept the rating as a
  *documented, accepted* risk — the lesson is to never do it blind.

## G-09 — llms.txt factual drift **[CO]**

- **Situation:** Product migrated payment provider (Stripe → Lemon
  Squeezy, ADR-002) and changed free-tier limits; llms.txt wasn't touched.
- **Symptom:** AI assistants reading llms.txt reported the wrong payment
  processor, wrong free-tier capacity, and a stale "last updated: March"
  two months later.
- **Root cause:** llms.txt treated as a static SEO artifact, not live
  marketing content.
- **Fix:** Version-control it alongside product changes; pull dynamic
  facts (price) from the same SSOT as the site; add it to the release
  checklist.

## G-10 — Cloudflare/CSP could silently challenge bots **[MM]**

- **Situation:** Brief warned of a "previous Cloudflare bot-block." Pages
  loaded fine in a browser.
- **Symptom (potential):** If the WAF served Turnstile/"just a moment" to
  bot UAs, every AI crawler would get an interstitial instead of content
  — invisible from a normal browser session.
- **Root cause:** WAF challenge behavior is UA/heuristic-dependent and not
  observable from a human visit.
- **Fix:** Curl prod with each bot UA, assert 200 + full HTML + no
  challenge markers (`cf-mitigated`, "just a moment", "turnstile"). Here
  it was clean (3/3 200), but the *verification step* is the takeaway, not
  the result.

## G-11 — YAML frontmatter broken by a colon in a description **[MM]**

- **Situation:** `description: Kelly-kritérium: a tét-méretezés...`
- **Symptom:** `@std/yaml` parse error; page 404'd at build.
- **Root cause:** An unquoted `:` starts an implicit YAML mapping pair.
- **Fix:** Quote any frontmatter value containing `:` → `description:
  "Kelly-kritérium: ..."`. Cheap to prevent, annoying to diagnose.

## G-12 — Missing pricing tier in schema offers **[CO]**

- **Situation:** `SoftwareApplication.offers` listed Free ($0) and Pro
  ($6) but not Business ($19).
- **Symptom:** Search engines never surfaced the Business plan; pricing
  snippets incomplete.
- **Root cause:** Schema authored against headline tiers, not the full
  pricing matrix.
- **Fix:** `offers` mirrors **every** tier, currency derived from locale.
  Cross-check against the pricing SSOT in CI.

## G-13 — Thin content under the GEO length floor **[TW]**

- **Situation:** Shift-worker blog articles were 282–390 words.
- **Symptom:** Poor citation pickup; pages read as stubs.
- **Root cause:** Treating a blog post like a meta description.
- **Fix:** Expand to ≥630 words with question-H2s, direct answers, lists,
  and a clinical/legal caution block. Dense and structured, not padded.

## G-14 — Title/meta length and year-stamping **[GR][TW]**

- **Situation:** Titles up to 67–84 chars; meta descriptions 160–206
  chars; year baked into meta titles; mixed separators (`·`, `—`, `-`).
- **Symptom:** SERP truncation; meta looks stale a year later;
  inconsistent brand presentation.
- **Root cause:** No standardized title/description convention.
- **Fix:** Title ≤ ~59, description ≤ ~155, single `|` separator, brand
  suffix once, **no year in meta** (years live in body copy like "2025
  market prices").

## G-15 — Author attributed to Organization instead of Person **[GR]**

- **Situation:** 10 guides had `Article.author` = the Organization.
- **Symptom:** Weak Expertise/Authoritativeness signal; ~34/100 E-E-A-T.
- **Root cause:** Default author wiring pointed at the org node.
- **Fix:** Point `author` at the founder `Person` node (`#founder`).
  One-line change, scored +7 E-E-A-T.

## G-16 — Custom 404 served with HTTP 200 (soft-404) **[OX]**

- **Situation:** Static Astro landing behind Caddy. An SEO audit flagged
  "no custom 404." The framework's `src/pages/404.astro` existed but with
  `trailingSlash: 'always'` the static build emitted `dist/404/index.html`,
  not `dist/404.html`, and the host had no error handler.
- **Symptom:** Unknown URLs returned the generic server 404 (or, with a
  catch-all `try_files … /index.html` fallback, the homepage with **HTTP
  200** — a soft-404 Google treats as duplicate/low-quality content).
- **Root cause:** Two traps stacked: (a) Astro's `404.astro` output path
  collides with `trailingSlash:'always'`; (b) static hosts don't serve an
  error page unless explicitly told, and a naive rewrite serves it as 200.
- **Fix:** Author error pages as **self-contained `public/404.html` +
  `public/500.html`** (inline CSS/JS, no pipeline dependency) — the build
  copies them verbatim to the exact filenames. Wire the host to serve them
  **and preserve the status code**. Caddy: `handle_errors { root * <dist>;
  rewrite @e4xx /404.html; rewrite @e5xx /500.html; file_server }` —
  `file_server` inside `handle_errors` keeps the original error status
  (no soft-404). The 5xx page also catches a downstream `reverse_proxy`
  502/503 (e.g. the app is down) → a branded page instead of a stack-stub.

## G-17 — `www` and apex served from one block → host duplication **[OX]**

- **Situation:** Caddy site address `example.eu, www.example.eu { … }` —
  both hostnames served the **same** content with a 200.
- **Symptom:** Audit flagged URL-canonicalization (HIGH). `www.example.eu/`
  and `example.eu/` were both indexable, duplicating every page on two
  hosts. The canonical tag mitigated but didn't resolve it.
- **Root cause:** Canonicalization is more than trailing-slash (G-02): the
  **host** (www vs apex) and **protocol** (http vs https) must also resolve
  to one. Sharing a server block serves both hosts without a redirect.
- **Fix:** 301 the non-canonical host to the canonical one:
  `@www host www.example.eu` + `redir @www https://example.eu{uri}
  permanent` (a top-level `redir` runs before `handle`, so it also covers
  app/login paths). http→https is automatic with Caddy/Let's Encrypt; on
  other stacks assert it too. Pick one canonical host and 301 every variant.

## G-18 — Product entity present, but `offers` only on the pricing page **[OX]**

- **Situation:** The unified `@graph` (G-05) put one `SoftwareApplication`
  (`@id:#software`) on every page; `offers` were added only on `/pricing`
  to avoid duplicating the price. The homepage `SoftwareApplication` had
  `aggregateRating` but **no `offers`**.
- **Symptom:** A schema checker run against the **homepage** (the primary
  product/`mainEntity` page) reported "no offer" — the page most likely to
  be parsed as the product looked incomplete.
- **Root cause:** Scoping `offers` to one route, not to the entity. The
  product entity is the same `@id` everywhere; its offers shouldn't depend
  on which page rendered it. (Distinct from G-12 = a *tier missing* from
  the offers array.)
- **Fix:** One shared offer builder feeds **both** the homepage and the
  pricing page's `SoftwareApplication`, so the primary product page carries
  `offers` + `aggregateRating` (the most complete form). Keep price omitted
  (not the whole offer) while pricing is in draft — a priceless `Offer`
  with `category`/`description` is still valid and useful.

## G-19 — Social/OG image as a relative path or SVG → no preview **[OX]**

- **Situation:** Pages set `og:title`/`og:description` but `og:image` was a
  root-relative path (`/og.png`) — or an SVG — and there was no
  `twitter:card`.
- **Symptom:** Audit flagged missing/low-quality social preview; pasted
  links rendered as a bare text stub on X/LinkedIn/Slack instead of a card.
- **Root cause:** Social scrapers (Facebook/X/LinkedIn) fetch the page in
  isolation and require an **absolute** `og:image` URL; X needs an explicit
  `twitter:card` and **does not render SVG** — it needs a raster (PNG/JPG).
- **Fix:** Emit `og:image` as an **absolute** URL (`new URL(path, site)`),
  add `twitter:card=summary_large_image` + `twitter:image`, and ship a real
  **1200×630 PNG** (1.91:1, <5 MB). Generate it from the vector brand
  source at build/release time (e.g. `sharp` rasterizing an SVG) so it
  stays on-brand and versioned, not hand-exported. Same for the icon set:
  `apple-touch-icon` (180), manifest `192/512`, and a PNG `favicon.ico`
  alongside the SVG — auditors and legacy clients look for the raster.
