# Changelog

All notable changes to this playbook are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The playbook is
versioned by *matrix shape*: bumping the major number when topics are added or
removed, the minor when an existing topic gains a major rewrite, and the patch
for everything else.

## [0.4.0] — 2026-06-04

### Added

- **New topic 25 — SEO & GEO (AI Search)** — full content (README,
  checklist, best-practices, gotchas, `examples/sveltekit-supabase.md`),
  distilled from the SEO/GEO work of four production sites:
  matekmegoldasok.hu (Deno Fresh), cutoptim.com (Astro), grabit.hu
  (Astro + SvelteKit), trackwell.eu (Astro). Every checklist item carries
  a provenance tag mapping it to the project that paid for the lesson.
  Covers the full pipeline: crawler access → indexation → JSON-LD `@graph`
  → citability → E-E-A-T, plus a validation harness and llms.txt.

### Changed

- **Moved the unified JSON-LD `@graph` playbook** from
  `17-marketing-seo/schema-graph-playbook.md` to
  `25-seo-geo/schema-graph-playbook.md`, so all structured-data material
  lives in one topic. Topic 17 now stays focused on *building the
  marketing site*; topic 25 covers the cross-cutting *technical SEO + GEO*
  discipline. `17-marketing-seo/README.md` updated to cross-reference 25.
- Root `README.md` matrix expanded from 24 to 25 topics (+ third-audit
  note). `ROADMAP.md` (Stage 2 — Growth) and `tasks/MATRIX.md` updated
  to 25 rows.

## [0.3.0] — 2026-05-02

### Added

- **`ai/claude/` projekt-onboarding kit** — sablonok új SaaS projekt
  Claude Code-os indításához:
  - `CLAUDE.template.md` — projekt-alkotmány sablon (doksi-sync szabály
    + CLAUDE.md self-update szabály + end-of-session checklist).
  - `DOC-GAP-AUDIT.template.md` — sablon a doksi-fedettség + quality
    audit + Phase 1/2/3 roadmap mátrixhoz.
  - `END-OF-SESSION-CHECKLIST.md` — 1 perces review minden session
    végén, akár emberi review akár Claude prompt formájában.
  - `README.md` — használati útmutató.

### Changed

- Root `README.md` egy új szakasszal bővült ("Starting a new SaaS
  project? Use the AI onboarding kit"), ami a `ai/claude/` template-ekre
  irányít.

## [0.2.0] — 2026-05-01

### Added

- **Six new topics (19–24)** following a second audit triggered by
  using the playbook to plan a SaaS, not just ship one:
  - 19 — Discovery & Validation.
  - 20 — Pricing Strategy.
  - 21 — Brand & Design System.
  - 22 — Accessibility (a11y).
  - 23 — User Documentation & Help Center.
  - 24 — Launch Playbook.
- All 6 topics scaffolded with substantive READMEs and outline-marked
  supporting files (forthcoming in v0.3+).
- **`ROADMAP.md` at root** — lifecycle-ordered view of when each topic
  matters in a SaaS's life, separate from the numeric-ID matrix view.
- **`tasks/` folder** — sprint-planning structure:
  - `tasks/README.md` — workflow guide.
  - `tasks/MATRIX.md` — single big table: every topic = one row, with
    remaining work, effort estimate, priority, dependencies.
  - `tasks/sprint-template.md` — template for actual sprint plans.
  - `tasks/sprints/` — empty directory where actual sprint plans live.

### Changed

- Root `README.md` matrix expanded from 18 to 24 topics.
- `internal/topic-validation.md` updated with reasoning for the second
  audit and the lifecycle-stage grouping of all 24 topics.

## [0.1.0] — 2026-04-30

Initial public commit.

### Added

- Repository structure with 18 topics (matrix in `README.md`).
- Topic 01 — Infrastructure & DevOps: full content.
  - README, checklist, best-practices, gotchas.
  - Examples: SvelteKit + Supabase, Next.js + Prisma, Remix + PlanetScale.
- Topic 03 — Billing & Subscription: full content.
  - README, checklist, best-practices, gotchas.
  - Examples for the same three stacks, with Stripe / Lemon Squeezy / Paddle
    + Barion provider variants.
- Outline-only content for topics 02, 04–18. Content in subsequent releases.
- Root files: `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `LICENSE` (MIT).
- `internal/topic-validation.md` — reasoning for the 18-topic shape vs. the
  original 12.

### Sources for filled topics

- Topic 01: based on Grabit (`bookolj` monorepo) production VPS setup —
  Hetzner CX22, Caddy with Cloudflare DNS plugin for wildcard SSL, PM2 process
  manager, GitHub Actions SSH deploy.
- Topic 03: based on Grabit's `pricing.json` + `BillingAdapter` interface,
  generalized for Stripe and Lemon Squeezy.

### Known gaps

- Topic 02 (Auth & Multi-tenancy) is high priority for the 0.2 release.
- Topic 14 (Data & Database Operations) likewise — backups and migrations
  guidance is too important to leave outlined.
- Three example stacks per topic is a stretch goal; many examples will be
  written for SvelteKit + Supabase first and ported afterwards.
