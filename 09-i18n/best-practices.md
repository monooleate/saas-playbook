# 09 — Internationalization — Best Practices

> **Status:** Outline.

## Sources

- W3C i18n FAQ: [w3.org/International/questions](https://www.w3.org/International/questions).
- Mozilla Localization style guides.
- Paraglide JS / next-intl / i18next docs.

## Practices to expand

- Translate full sentences, not fragments. "{count} {item_plural}"
  beats "{count}" + " " + "items".
- Always include a default language (English usually) as the fallback.
- Don't translate brand names, proper nouns, or non-translatable
  technical terms.
- Trust translators; use ICU MessageFormat to give them control over
  plurals/gender without code changes.
- Test layout overflow with German (long words) and Japanese (no
  spaces).
