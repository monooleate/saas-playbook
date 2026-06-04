# 16 — SEO & GEO — Example: SvelteKit + Supabase (multi-tenant)

> Shape: an **Astro static landing** (`grabit.hu`) for marketing/content
> that gets indexed, plus a **SvelteKit app** (`*.grabit.hu`) for the
> multi-tenant product that must stay out of the index. Mirrors the
> grabit.hu production setup. Supabase for auth/data; tenant resolved
> from the subdomain in `hooks.server.ts`.

## 1. Indexation split — landing indexed, tenant subdomains noindex

### Landing `robots.txt` (Astro — `apps/landing/public/robots.txt`)

```
User-agent: *
Allow: /
Disallow: /api/

# Answer-engine crawlers — explicit, by name
User-agent: GPTBot
Allow: /
User-agent: ChatGPT-User
Allow: /
User-agent: OAI-SearchBot
Allow: /
User-agent: ClaudeBot
Allow: /
User-agent: PerplexityBot
Allow: /
User-agent: Google-Extended
Allow: /
User-agent: Applebot-Extended
Allow: /
User-agent: CCBot
Allow: /

Sitemap: https://grabit.hu/sitemap-index.xml
```

### Tenant `robots.txt` (SvelteKit — `apps/app/src/routes/robots.txt/+server.ts`)

```typescript
import type { RequestHandler } from './$types';

export const GET: RequestHandler = () =>
  new Response('User-agent: *\nDisallow: /\n', {
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'public, max-age=86400',
    },
  });
```

### Tenant `X-Robots-Tag` header (SvelteKit — `apps/app/src/hooks.server.ts`)

```typescript
export const handle: Handle = async ({ event, resolve }) => {
  // resolve tenant from subdomain earlier in the chain...
  const response = await resolve(event);

  // Google trusts the header most; robots.txt only controls crawl budget.
  if (event.locals.tenant) {
    response.headers.set('X-Robots-Tag', 'noindex, nofollow');
  }
  return response;
};
```

Plus the HTML fallback in the tenant root layout:

```svelte
<!-- apps/app/src/routes/+layout.svelte -->
<svelte:head>
  <meta name="robots" content="noindex, nofollow" />
</svelte:head>
```

## 2. Sitemap (landing only)

`@astrojs/sitemap` generates `sitemap-index.xml` + `sitemap-0.xml`.
Redirect the conventional `/sitemap.xml` to the index (a 301, not a
static file — the spec forbids an index pointing to another index):

```toml
# apps/landing/netlify.toml  (or equivalent on your host)
[[redirects]]
  from = "/sitemap.xml"
  to = "/sitemap-index.xml"
  status = 301
```

Exclude any noindex/thin route from the sitemap:

```javascript
// apps/landing/astro.config.mjs
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://grabit.hu',
  trailingSlash: 'never',           // one decision, applied everywhere
  integrations: [
    sitemap({
      filter: (page) =>
        !/\/(shop-success|thank-you)(\/|$)/.test(page) &&
        !page.includes('/admin'),
    }),
  ],
});
```

## 3. One `@graph` per page from a single builder

```typescript
// apps/landing/src/lib/schema.ts
const SITE = 'https://grabit.hu';
const ORG_ID = `${SITE}/#organization`;
const SITE_ID = `${SITE}/#website`;
// Network-level founder id — shared across the founder's brands.
const PERSON_ID = 'https://jmeszaros.dev/#person';

const orgNode = () => ({
  '@type': 'Organization',
  '@id': ORG_ID,
  name: 'Grabit',
  url: SITE,
  founder: { '@id': PERSON_ID },
  sameAs: ['https://www.linkedin.com/company/grabit'], // must resolve!
});

const personNode = () => ({
  '@type': 'Person',
  '@id': PERSON_ID,
  name: 'János Mészáros',
  jobTitle: 'Founder',
  sameAs: ['https://www.linkedin.com/in/jmeszaros', 'https://github.com/...'],
});

const webSiteNode = () => ({
  '@type': 'WebSite',
  '@id': SITE_ID,
  url: SITE,
  name: 'Grabit',
  inLanguage: 'hu-HU',
  publisher: { '@id': ORG_ID },
});

export function buildPageGraph(opts: {
  url: string;
  type: 'WebPage' | 'Article' | 'ContactPage';
  title: string;
  breadcrumb?: { name: string; item?: string }[];
  faq?: { q: string; a: string }[];
  author?: boolean; // articles → author is the Person, not the Org
}) {
  const nodes: object[] = [orgNode(), webSiteNode(), personNode()];

  nodes.push({
    '@type': opts.type,
    '@id': `${opts.url}#webpage`,
    url: opts.url,
    name: opts.title,
    isPartOf: { '@id': SITE_ID },
    about: { '@id': ORG_ID },
    inLanguage: 'hu-HU',
    ...(opts.author && { author: { '@id': PERSON_ID } }),
  });

  if (opts.breadcrumb?.length) {
    nodes.push({
      '@type': 'BreadcrumbList',
      '@id': `${opts.url}#breadcrumb`,
      itemListElement: opts.breadcrumb.map((b, i) => ({
        '@type': 'ListItem',
        position: i + 1,
        name: b.name,
        ...(b.item && { item: b.item }),
      })),
    });
  }

  if (opts.faq?.length) {
    nodes.push({
      '@type': 'FAQPage',
      '@id': `${opts.url}#faq`,
      // DRY: same source array also renders the visible <details> FAQ.
      mainEntity: opts.faq.map((f) => ({
        '@type': 'Question',
        name: f.q,
        acceptedAnswer: { '@type': 'Answer', text: f.a },
      })),
    });
  }

  return { '@context': 'https://schema.org', '@graph': nodes };
}

// 7-bit-clean serialization — immune to charset bugs downstream.
export const serializeGraph = (g: object) =>
  JSON.stringify(g).replace(/[-￿]/g, (c) =>
    '\\u' + c.charCodeAt(0).toString(16).padStart(4, '0'),
  );
```

Emit exactly one block in the base layout:

```astro
---
// apps/landing/src/layouts/BaseLayout.astro
import { buildPageGraph, serializeGraph } from '../lib/schema';
const { url, type, title, breadcrumb, faq, author } = Astro.props;
const graph = buildPageGraph({ url: Astro.url.href, type, title, breadcrumb, faq, author });
---
<script type="application/ld+json" id="graph" set:html={serializeGraph(graph)} />
```

## 4. CI scanner — the regression guard

```javascript
// apps/landing/scripts/scan-jsonld.mjs  →  `node scripts/scan-jsonld.mjs`
import { readFileSync } from 'node:fs';
import { globSync } from 'glob';

let fail = 0;
for (const file of globSync('dist/**/*.html')) {
  const html = readFileSync(file, 'utf8');
  const graphs = [...html.matchAll(/id="graph"[^>]*>(.*?)<\/script>/gs)];
  const legacy = (html.match(/application\/ld\+json/g) || []).length;

  if (graphs.length !== 1) { console.error(`${file}: ${graphs.length} graph blocks`); fail++; continue; }
  if (legacy !== 1)        { console.error(`${file}: ${legacy} ld+json blocks (expect 1)`); fail++; }

  const g = JSON.parse(graphs[0][1]);
  const ids = new Set(g['@graph'].map((n) => n['@id']).filter(Boolean));
  const refs = JSON.stringify(g['@graph']).match(/"@id":"([^"]+)"/g) || [];
  for (const r of refs) {
    const id = r.slice(7, -1);
    // a ref to a node not defined in THIS graph is dangling (ignore external person id)
    if (!ids.has(id) && !id.startsWith('https://jmeszaros.dev')) {
      console.error(`${file}: dangling @id ref ${id}`); fail++;
    }
  }
  for (const t of ['Organization', 'WebSite', 'WebPage']) {
    if (!g['@graph'].some((n) => n['@type'] === t)) { console.error(`${file}: missing ${t}`); fail++; }
  }
}
process.exit(fail ? 1 : 0);
```

Wire it after build: `astro build && node scripts/scan-jsonld.mjs`.

## 5. llms.txt fed from the same price SSOT

```astro
---
// apps/landing/src/pages/llms.txt.ts  (Astro endpoint)
import { PRICING } from '../lib/pricing';   // the one source of truth
export const GET = () => new Response(
`# Grabit
> Online booking platform for Hungarian small businesses. ${PRICING.tiers.length} plans from ${PRICING.from} HUF/mo.

## Key pages
- [Home](https://grabit.hu): What Grabit does, for whom.
- [Pricing](https://grabit.hu/arazas): ${PRICING.tiers.map(t => t.name).join(', ')}.
- [Calculators](https://grabit.hu/kalkulatorok): No-show, ROI, capacity.
- [Guides](https://grabit.hu/utmutatok): Booking setup, Google reviews, retention.

## Legal
- [Terms](https://grabit.hu/aszf) · [Privacy](https://grabit.hu/adatkezeles)
`,
  { headers: { 'Content-Type': 'text/plain; charset=utf-8' } },
);
---
```

Because the price comes from `PRICING`, a pricing change can't leave
`llms.txt` (or the `Product` schema) stale — the G-09 drift class is
designed out.

## 6. Verify access before declaring done

```bash
for ua in "GPTBot/1.1" "ClaudeBot/1.0" "PerplexityBot/1.0" "OAI-SearchBot/1.0"; do
  curl -s -o /tmp/b -w "$ua: %{http_code}\n" -A "$ua" https://grabit.hu/
  grep -qi "just a moment\|turnstile" /tmp/b && echo "  ^ CHALLENGED"
done
# Tenant must be the opposite — noindex header present:
curl -sI -H "Host: demo.grabit.hu" https://grabit.hu/ | grep -i x-robots-tag
```
