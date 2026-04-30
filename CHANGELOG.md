# Changelog

All notable changes to this playbook are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The playbook is
versioned by *matrix shape*: bumping the major number when topics are added or
removed, the minor when an existing topic gains a major rewrite, and the patch
for everything else.

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
