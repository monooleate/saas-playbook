# 23 — User Documentation & Help Center

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before public launch.
> **Cost:** 2–5 days for v1; ongoing.

## Scope

The help-center side of customer comms — distinct from changelog (16,
"what changed") and API docs (18, "how to integrate"):

1. **Knowledge base** — searchable articles for "how do I X".
2. **Onboarding videos / GIFs** — short, embedded in articles.
3. **Inline help** — tooltips, "What is this?" links.
4. **FAQ** — the 20 most asked questions.
5. **Troubleshooting** — what to do when X breaks.
6. **Glossary** — domain terms specific to your product.
7. **Search** — Algolia DocSearch (free for OSS), or in-product search.

## Why it matters

Every "how do I" support ticket that an article could have answered
costs you 10 minutes. Multiply by users × month. A solid help center
deflects 30–60% of low-value tickets.

It's also a quiet SEO play: long-tail "how to X with [your product]"
searches.

## Recommended stack

- **Static site as part of marketing** — `docs.example.com` or
  `/docs`. Pages are MDX; same Astro/Next.js stack as the marketing
  site.
- **Astro Starlight** is purpose-built for this. Used by Cloudflare,
  Bun, Astro itself.
- **Mintlify** for hosted docs with auto-generation (paid).
- **Notion-as-help-center** for very early stage; export and migrate
  before scale.

Search:
- **Algolia DocSearch** — free for OSS / public docs.
- **Pagefind** — static-site search, no service required.
- **Inkeep** — AI-powered support search (paid).

## Pattern

Article structure that scales:

1. **Title** — searchable, in user's words.
2. **Summary** — 1 sentence.
3. **Steps** — numbered, with screenshots/GIFs.
4. **Common errors** — what goes wrong; how to fix.
5. **Related articles** — internal linking.

Linear, Notion, and Vercel docs all follow this structure.

## What goes in v0.2

- Astro Starlight setup.
- Article authoring guide.
- Search setup.
- Inline help patterns ("?" tooltips, contextual links).
- AI assistance (Inkeep, custom RAG) — when worth it.

## Sources

- Astro Starlight: [starlight.astro.build](https://starlight.astro.build/).
- Mintlify: [mintlify.com](https://mintlify.com).
- Cloudflare Docs as a structural reference.
- Linear Help Center as a content-style reference.
- *Every Page is Page One* — Mark Baker.
