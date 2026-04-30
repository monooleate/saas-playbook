# 09 — Internationalization — Gotchas

> **Status:** Outline.

## Categories

1. **Hardcoded English string slips through.** Solution: ESLint rule.
2. **Date formatted server-side in UTC, displayed client-side as
   local.** Off-by-one days.
3. **Number formatting differences.** `1,234.56` (US) vs `1.234,56`
   (DE). Don't format manually; use `Intl.NumberFormat`.
4. **Plural rules wrong for Polish, Russian, Arabic.** English's "1
   item / N items" doesn't transfer.
5. **String concatenation breaks word order.** "There are {n} items"
   doesn't translate to all SOV languages.
6. **Right-to-left CSS missed.** `margin-left` is wrong; use
   `margin-inline-start`.
7. **`hreflang` tag wrong.** Google ignores invalid language codes;
   no SEO benefit.
8. **Translation key collision.** `errors.required` used in two
   contexts means changes in one bleed into the other.

(Real incidents in v0.2.)
