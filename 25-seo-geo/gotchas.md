# 16 — SEO & GEO — Gotchas

> Real things that broke or were caught in audit, written as
> situation → symptom → root cause → fix. Provenance: **[MM]**
> matekmegoldasok (Deno Fresh), **[CO]** cutoptim (Astro), **[GR]**
> grabit (Astro + SvelteKit), **[TW]** trackwell (Astro).

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
