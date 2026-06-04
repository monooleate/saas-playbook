# 25 — Unified JSON-LD `@graph` Playbook (SEO/GEO)

> **Status:** Battle-tested. Executed on a real 2-site Astro codebase
> (309 + 137 pages) with 0 regressions.
> **When to use:** Any site that emits structured data as **multiple separate
> `<script type="application/ld+json">` blocks** and wants a single, connected,
> AI-citable entity graph.
> **Cost:** ~0.5–1.5 days depending on page-type count.

## Why this matters (SEO + GEO)

Search engines and AI answer engines (ChatGPT, Perplexity, Gemini, Google AI
Overviews) build an **entity graph** from your structured data. Disconnected
JSON-LD blocks — each with its own `@context`, no shared `@id`s — read as
unrelated fragments. A single self-contained `@graph` per page with stable
`@id`s and a `BreadcrumbList → WebPage → WebSite → Organization` spine lets the
engine resolve every reference locally and tie pages to **one founder/author
entity** across your whole network. This is pure upside: you only **reorganize**
existing signals, never drop them.

### The bug this almost always fixes

Sites that split JSON-LD into blocks frequently reference `@id`s that aren't
present on the page — e.g. an Article's `author: {@id: …/#person}` or
`publisher: {@id: …/#organization}` where the `Person`/`Organization` node is
only emitted on the homepage. Those are **dangling refs on every content page**.
Consolidating into one `@graph` that always includes the Org/WebSite/Person/
WebPage base nodes resolves them.

---

## The prompt (copy-paste, stack-agnostic)

Fill the `{{...}}` parameters or let the prompt ask them in Phase 2.

```text
ROLE: You are a Technical SEO/GEO engineer. TASK: convert this project's JSON-LD
structured data into a SINGLE, self-contained @graph per page, with stable and
consistent @ids. Goal: pages can only get BETTER for SEO/GEO — no existing schema
signal is lost, only REORGANIZED.

────────── PARAMETERS (if blank, ASK in Phase 2) ──────────
- PERSON_ID (canonical @id of the author/founder for the whole network): {{e.g. https://name.dev/#person}}
- Brand/Org names + domains per site: {{...}}
- Sibling sites (same owner, SEPARATE brands, NO 1:1 hreflang between them): {{... or "none"}}
- Fabricated aggregateRating on tools/products: {{keep | remove | only-if-real-data}}

────────── WORK RULES (ENFORCE THROUGHOUT) ──────────
1. After EVERY step, VERIFY/AUDIT that the change IMPROVES and does NOT break / NOT
   degrade any existing SEO/GEO signal. Step = a phase or one independently testable edit.
2. NEVER drop a prior schema type/field — the refactor only REORGANIZES (separate
   blocks → one @graph). Sensitive: Product offers/aggregateRating/review, Article
   speakable/about, FAQ, breadcrumb, canonical/robots/hreflang/og/twitter.
3. Git: if on the default branch, branch FIRST. Commit/push/merge ONLY if asked.
   Commit trailer: Co-Authored-By: <model used> <noreply@anthropic.com>. In bash use a
   HEREDOC for the commit message (git commit -F - <<'EOF' … EOF). Do NOT commit
   generated build output (dist/, .next/, .astro/, out/) in the same commit as source —
   ask if the deploy relies on committed build artifacts.
4. Do NOT bulk-edit content/markdown frontmatter — do normalization (author/publisher/
   @context/@id) at RENDER TIME in the route/builder.

────────── PHASE 0 — Discovery (read-only) ──────────
- Do NOT assume a monorepo/stack. Map: how many apps/sites, how they build, shared code
  vs per-site env-config, the stack (Next/Astro/Fresh/Vite/…), and WHERE/HOW JSON-LD
  (application/ld+json) is emitted today, per page type.
- Page-type inventory: route pattern → present schema type(s) → gaps. Flag where NONE exists.
- Is there a shared helper, or per-page duplicated blocks?

────────── PHASE 1 — Audit (read-only) → audit/schema-graph-audit.md ──────────
0. Accessibility: robots.txt + LIVE AI-bot edge test (GPTBot/ClaudeBot/PerplexityBot →
   200 + real HTML, no Cloudflare challenge) for every domain.
1. Inventory table (page type → schema → gap), per site.
2. Entity consistency: is there ONE stable Organization/WebSite @id per site? Is the
   author (Person) @id consistent and pointing to PERSON_ID? Conflicting entity defs?
3. Graph structure: connected @graph via @id refs, or scattered blocks? Is the
   BreadcrumbList → WebPage → WebSite → Organization chain present? DANGLING @id refs?
   Does the primary entity (calculator/product/article) have a meaningful type?
4. Language: does inLanguage match the lang/hreflang?
5. Validation: per-sample JSON.parse + missing required/recommended properties.
Output: executive summary + 0–10 graphiness score, inventory, P0/P1/P2 issues with
routes, shared-helper sketch, 3 concrete diff proposals. AUDIT ONLY — do NOT change code.

────────── PHASE 2 — Plan → audit/schema-graph-implementation-plan.md ──────────
- Design the shared SSOT (a common module fit for the stack), parameterized by site config:
  { siteUrl, brandName, locale, defaultCurrency }. Node builders (return OBJECTS, not strings):
  orgNode({full}), websiteNode(), personNode() [with PERSON_ID],
  webPageNode({url,name,description,type,inLanguage,image,primaryId,hasBreadcrumb}),
  breadcrumbNode(url,items), faqNode(url,items), <primaryEntity>Node(...),
  siblingSiteNodes() [other brands' Org+WebSite nodes, founder→PERSON_ID],
  buildPageGraph({...}) [adds the base Org/WebSite/Person/WebPage nodes CENTRALLY],
  serializeGraph(graph) [JSON.stringify + `<` → < XSS-guard],
  normalizeFrontmatterSchema(raw,{pageUrl}) [strip @context, author→PERSON_ID, stable @id].
  PRINCIPLE: every page's graph contains AT LEAST Org/WebSite/Person/WebPage → every @id-ref
  resolves on-page (0 dangling). @id scheme: ${siteUrl}/#organization|#website,
  ${pageUrl}#webpage|#breadcrumb|#faq|#article|#<entity>|#itemlist.
- TIP: the render layer (layout/template) can assemble the base nodes from existing page
  meta (canonical/title/description/ogImage) → fewer caller changes.
- ASK the user only the genuinely-deciding questions: PERSON_ID, brand names, fate of
  fabricated aggregateRating.
- Order: SSOT → about/Person unification → article/primary-entity pages → category/product →
  missing pages → render cutover (1 id="graph") → sibling linking → validation. The cutover
  is order-bound: route migrations FIRST. Keep the old path backward-compatible during
  migration (fallback), then delete dead code at the end.

────────── PHASE 3 — Implementation (step by step) ──────────
- Introduce the SSOT; switch EVERY page type to ONE @graph (a single
  <script type="application/ld+json" id="graph">). The FIELDS of the old separate blocks
  move into graph nodes. Normalize at render time (strip frontmatter @context, author→
  PERSON_ID, publisher→ORG_ID, every node gets a #fragment @id; the primary entity is a
  FIRST-CLASS top-level node; WebPage.mainEntity and the article reference it by @id).
- No dangling cross-page refs: if the source references a node that only exists on the
  homepage, rewrite it to the page's own #webpage. Homepage + about: orgFull + includeSiblings
  (founder→PERSON_ID). Free tools get real offers {price:"0", priceCurrency:<site currency>}.
  Do NOT fabricate aggregateRating/review (unless the parameter says "keep").

────────── PHASE 4 — Verify after EVERY step ──────────
A) Static: the stack's type checker on MODIFIED files → 0 NEW errors (isolate pre-existing;
   if tsc/checker can't see virtual modules, the BUILD is the ground truth).
B) Build/dev audit: build (or start dev server), then run the scanner (below) on EVERY site,
   EVERY affected page type. Assert: exactly 1 id="graph", 0 legacy ld+json blocks, parse OK,
   0 DANGLING @id-refs, Person @id = PERSON_ID, Organization+WebSite+WebPage present,
   sibling Orgs present (homepage/about), inLanguage = site language, and EVERY prior
   schema type/field still present (regression). PASS/FAIL summary + type histogram.
C) Live AI-bot edge test after deploy (200 + real HTML, no challenge).
D) Google Rich Results Test + validator.schema.org, 1 page per type after deploy: target 0
   errors; document warnings.

────────── PHASE 5 — Git ──────────
- Branch, separate commit(s) per phase. Push/merge ONLY if asked. Short changelog
  (what/files/validation/deviations/next); mark the plan ✅ executed.

────────── SUCCESS CRITERIA ──────────
- Every page exactly 1 unified @graph, 0 dangling @id-refs, one founder entity (PERSON_ID),
  Org/WebSite/WebPage/Breadcrumb chain everywhere; sibling brands (if any) entity-linked via a
  shared founder; 0 lost SEO/GEO signals (scanner proves it); 0 Rich Results errors; clean git
  history + changelog.

Start with PHASE 0. Before finalizing the PHASE 2 plan, ASK about the parameters above.
Run verification after every step.
```

---

## Verification scanner (Node, stack-agnostic)

Scans built static HTML (Astro `dist/`, Next `out/`, Fresh, any static export).
`node scripts/verify-graph.mjs <htmlDir> <PERSON_ID>`

```js
// scripts/verify-graph.mjs
import { readFileSync, readdirSync, statSync } from "node:fs";
import { join } from "node:path";
const DIR = process.argv[2] || "dist";
const PERSON_ID = process.argv[3] || "https://CHANGE-ME/#person";
const walk = (d) => readdirSync(d).flatMap((e) => {
  const p = join(d, e); return statSync(p).isDirectory() ? walk(p) : p.endsWith(".html") ? [p] : [];
});
function dangling(g) {
  const ids = new Set(g.map((n) => n["@id"]).filter(Boolean)); const refs = [];
  const w = (o) => { if (Array.isArray(o)) return o.forEach(w);
    if (o && typeof o === "object") { const k = Object.keys(o);
      if (k.length === 1 && k[0] === "@id") refs.push(o["@id"]); else for (const x of k) if (x !== "@id") w(o[x]); } };
  w(g); return [...new Set(refs.filter((r) => !ids.has(r)))];
}
let pass = 0, fail = 0, noGraph = 0; const fails = []; const types = {};
for (const f of walk(DIR)) {
  const html = readFileSync(f, "utf8");
  const graphs = [...html.matchAll(/<script type="application\/ld\+json" id="graph"[^>]*>([\s\S]*?)<\/script>/g)];
  const legacy = [...html.matchAll(/<script[^>]*type="application\/ld\+json"[^>]*>/g)].length - graphs.length;
  if (graphs.length === 0) { noGraph++; continue; }
  const rel = f.replace(DIR, "").replace(/\\/g, "/");
  const ok = (c, m) => c ? pass++ : (fail++, fails.push(`FAIL ${rel}: ${m}`));
  ok(graphs.length === 1, `1 id=graph (got ${graphs.length})`);
  ok(legacy === 0, `0 legacy ld+json (got ${legacy})`);
  let g; try { g = JSON.parse(graphs[0][1].replace(/\\u003c/g, "<"))["@graph"]; }
  catch (e) { fail++; fails.push(`FAIL ${rel}: parse ${e.message}`); continue; }
  ok(dangling(g).length === 0, `dangling ${JSON.stringify(dangling(g))}`);
  const person = g.find((n) => n["@type"] === "Person");
  ok(person && person["@id"] === PERSON_ID, `Person @id=PERSON_ID (got ${person && person["@id"]})`);
  ok(g.some((n) => n["@type"] === "Organization"), "Organization present");
  ok(g.some((n) => n["@type"] === "WebSite"), "WebSite present");
  for (const n of g) { const t = Array.isArray(n["@type"]) ? n["@type"].join("+") : n["@type"]; types[t] = (types[t]||0)+1; }
}
console.log("TYPES:", Object.entries(types).sort((a,b)=>b[1]-a[1]).map(([k,v])=>`${k}:${v}`).join("  "));
if (fails.length) fails.slice(0,40).forEach((l) => console.log("  " + l));
console.log(`no-graph pages: ${noGraph} | RESULT: ${pass} PASS, ${fail} FAIL`);
process.exit(fail > 0 ? 1 : 0);
```

Live AI-bot edge test (bash, run per production domain):

```bash
for ua in "GPTBot/1.0" "ClaudeBot/1.0" "PerplexityBot/1.0"; do
  code=$(curl -s -o /tmp/r.html -A "$ua" -w "%{http_code}" --max-time 20 "https://<DOMAIN>/")
  chal=$(grep -ci "just a moment\|verify you are human\|cf-challenge\|turnstile" /tmp/r.html)
  echo "$ua -> HTTP $code | challenge-markers: $chal (0 = not blocked)"
done
```

---

## Gotchas (learned the hard way)

- **Don't assume monorepo.** A single codebase building N sites via env vars is common
  and is the *easiest* case — one SSOT module covers every site. Detect first.
- **Centralized assembler beats per-caller `buildGraph`.** Put the base Org/WebSite/Person/
  WebPage assembly in the render layer (it already has canonical/title/description/ogImage).
  Callers only pass page-specific nodes → far fewer touch points, guaranteed 0 dangling.
- **Build output is the ground truth, not `tsc`.** For Astro/Next, bare `tsc` can't resolve
  virtual modules (`astro:content`, `import.meta.env`) and reports noise. Build + scan HTML.
- **Don't commit generated `dist/`/`.next/`/`.astro/` with the source change.** It's huge
  churn and, in a multi-site repo, only captures one site's build. Most hosts (Netlify/Vercel)
  rebuild on deploy. Scope the commit to source unless the deploy reads committed artifacts.
- **Cross-domain sibling `@id` is intentional, valid entity-linking** — not `sameAs` (separate
  brands) and not hreflang. Only emit sibling nodes on homepage/about.
- **Person node must be site-invariant.** The same `@id` (PERSON_ID) appears on every page of
  every site; keep its definition identical (no per-site `worksFor`) so engines don't see
  conflicting statements about one entity. Express the founder link on the Org side.
- **Mark the graph with `id="graph"` and apply the `<` → `<` XSS-guard.** The scanner and
  every regression check key off that marker.
- **Keep the `hreflang`/canonical/robots layer separate.** It lives in `<link>`/`<head>`, not
  in JSON-LD; the refactor must not touch it. Verify it survives.
- **`aggregateRating`/`review` on tools must be a deliberate decision.** Fabricated ratings are
  a Google structured-data policy risk; never silently add or remove them — ask the owner.

## Worked example

A 2-site Astro codebase (Romanian + Hungarian, ~450 pages combined) executed this playbook:
moved 12 separate JSON-LD block builders into one `seo.ts` SSOT + a `BaseLayout`-level
assembler. Result: every page 1 unified `@graph`, **0 dangling refs** (previously every tool
and category page had dangling `#founder`/`#organization`/`#website` refs), one founder entity
across 3 sibling brands. Scanner: **2142 + 938 assertions PASS, 0 FAIL**; `hreflang` and
`aggregateRating` preserved.
