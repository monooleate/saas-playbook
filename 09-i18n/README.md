# 09 — Internationalization

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before second-country launch.
> **Cost:** 5–10 days for the first additional language; 1–2 days each
> after that.

## Scope

Strings, dates, numbers, locale-aware URL routing, plural forms,
right-to-left layouts. **Not multi-currency** — that lives in topic 03.

1. **i18n library** — Paraglide JS, next-intl, react-intl, i18next.
2. **Locale detection** — URL prefix vs subdomain vs cookie vs
   `Accept-Language`.
3. **Translation workflow** — JSON files in repo vs Crowdin/Lokalise.
4. **Plurals** — ICU MessageFormat, CLDR plural rules.
5. **Number / date / currency formatting** — `Intl.*` browser APIs.
6. **Right-to-left** — Arabic, Hebrew. CSS logical properties.

## Locale URL strategies

| Strategy             | Example                  | Pros                         | Cons                          |
|----------------------|--------------------------|------------------------------|-------------------------------|
| Prefix               | `example.com/de/...`     | Simple, SEO-friendly         | Subdomain routing harder      |
| Subdomain            | `de.example.com`         | Clean separation             | Conflicts with tenant subdomain |
| Cookie + auto-detect | `example.com` everywhere | Simple URLs                  | Bad SEO, no shareable links   |

For multi-tenant SaaS where tenants live at `tenant.example.com`,
**path prefix** wins (`tenant.example.com/de/admin/...`).

## Recommended for solo founders

- **Paraglide JS** (used by Grabit) — typed, tree-shaken, no runtime
  cost for unused strings. SvelteKit-friendly.
- **next-intl** for Next.js.
- **i18next** for Remix and React general.

Translation source files in repo. Use Crowdin only when you have a
team of translators.

## What goes in v0.2

- Setup walkthrough for each i18n library.
- ICU MessageFormat patterns (plurals, gender, lists).
- Date formatting with `Intl.DateTimeFormat`.
- Pluralization edge cases (Slavic languages have 3–4 forms).
- RTL CSS patterns (`logical properties`, `direction`).
- SEO: `hreflang` tags, `lang` attribute, locale-aware sitemaps.
- Translator workflow: PR-based vs Crowdin sync.

## Sources

- Paraglide JS: [inlang.com/m/gerre34r/library-inlang-paraglideJs](https://inlang.com/m/gerre34r/library-inlang-paraglideJs).
- next-intl: [next-intl-docs.vercel.app](https://next-intl-docs.vercel.app).
- CLDR plural rules: [cldr.unicode.org/index/cldr-spec/plural-rules](https://cldr.unicode.org/index/cldr-spec/plural-rules).
- MDN `Intl` namespace docs.
