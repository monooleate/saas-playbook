# 09 — Internationalization — Checklist

> **Status:** Outline.

- i18n library chosen and configured.
- All user-visible strings extracted (no hardcoded `'Hello'`).
- Locale-aware routing decision made (prefix / subdomain / cookie).
- `<html lang="...">` set per request.
- `hreflang` tags on marketing pages.
- Date/time formatting via `Intl.DateTimeFormat`.
- Number formatting via `Intl.NumberFormat`.
- Pluralization tested for non-English plural rules (Russian, Polish).
- One non-trivial language fully translated (review by native speaker).
- Translation workflow documented (PR vs SaaS).

(Each expanded in v0.2.)
