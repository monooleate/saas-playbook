# SaaS Playbook

A stack-agnostic knowledge base for solo founders building production SaaS.
Not a boilerplate. Not a starter kit. A field guide.

Each topic answers four questions:

1. **What is it?** (concept)
2. **Why does it matter?** (impact on revenue, retention, sleep)
3. **What do I actually do?** (sprint-ready checklist)
4. **What goes wrong?** (gotchas from real production)

Patterns are framework-neutral. Code examples come in three concrete stacks:
**SvelteKit + Supabase**, **Next.js + Prisma**, **Remix + PlanetScale**.

---

## Reference implementations

The patterns in this playbook were extracted from three running products:

| Product   | Stack                                 | Status            | What it taught us                                                                                  |
|-----------|---------------------------------------|-------------------|----------------------------------------------------------------------------------------------------|
| Grabit    | SvelteKit + Supabase + Hetzner + Caddy | Production        | Multi-tenant subdomains, addon system, Barion + Paddle + Lemon multi-provider billing, pnpm monorepo |
| CutOptim  | Astro + Lemon Squeezy + Supabase      | Production        | Lemon Squeezy MoR billing, Astro static + island hydration, Netlify deploys                         |
| Trackwell | Astro + WeasyPrint + Netlify          | Production        | Server-side PDF pipeline, lightweight reporting SaaS                                                |

When a section says **"From Grabit:"** or **"From CutOptim:"**, that's a real production
pattern, not theory. When something is marked **"Industry pattern (Stripe / Linear /
Vercel):"** it's how the benchmark products do it, with a link to the source.

---

## Topic matrix

25 topics across the SaaS lifecycle. The numeric ID preserves the matrix
order; **the reading order is in [`ROADMAP.md`](./ROADMAP.md)**, organized
by SaaS lifecycle stage (pre-build → foundation → growth → operations →
extension).

If you're using the playbook to plan a real product, start with
[`tasks/MATRIX.md`](./tasks/MATRIX.md) — it's the same 25 rows with
estimated effort, priority, and dependencies, ready to feed into sprint
planning.

| #  | Topic                              | When to implement                      | Implementation cost | Status      |
|----|------------------------------------|----------------------------------------|---------------------|-------------|
| 01 | [Infrastructure & DevOps](./01-infrastructure/)         | Day 1 — before first user             | 1–3 days           | ✅ Filled    |
| 02 | [Auth & Multi-tenancy](./02-auth-multitenancy/)         | Day 1                                  | 3–5 days           | 📋 Outline  |
| 03 | [Billing & Subscription](./03-billing/)                 | Before paying customer #1             | 5–10 days          | ✅ Filled    |
| 04 | [Email & Communication](./04-email/)                    | Day 1 (transactional)                  | 1–2 days           | 📋 Outline  |
| 05 | [Feature Flags & Addon Gating](./05-feature-flags/)     | After billing                          | 2–4 days           | 📋 Outline  |
| 06 | [Onboarding](./06-onboarding/)                          | After first 10 users                   | 3–5 days           | 📋 Outline  |
| 07 | [Notification & Automation](./07-notification/)         | After 50 active tenants                | 2–4 days           | 📋 Outline  |
| 08 | [Analytics & Reporting](./08-analytics/)                | Month 2                                | 2–3 days           | 📋 Outline  |
| 09 | [Internationalization](./09-i18n/)                      | Before second-country launch           | 5–10 days          | 📋 Outline  |
| 10 | [Superadmin](./10-superadmin/)                          | Day 1 — before public launch          | 1–2 days, then forever | 📋 Outline  |
| 11 | [Security](./11-security/)                              | Day 1                                  | Continuous          | 📋 Outline  |
| 12 | [Testing](./12-testing/)                                | When pain > pleasure                   | 2–10 days           | 📋 Outline  |
| 13 | [Observability & Monitoring](./13-observability/)       | Day 1 (Sentry); Month 2 (metrics)      | 1–3 days           | 📋 Outline  |
| 14 | [Data & Database Operations](./14-data-ops/)            | Day 1 (migrations, RLS); before launch (backups) | 2–5 days   | 📋 Outline  |
| 15 | [Legal & Compliance](./15-legal-compliance/)            | Before first paying customer           | 1–3 days           | 📋 Outline  |
| 16 | [Customer Support & Changelog](./16-support-changelog/) | After first 20 users                   | 1–2 days           | 📋 Outline  |
| 17 | [Marketing Site & SEO](./17-marketing-seo/)             | Before public launch                   | 5–15 days           | 📋 Outline  |
| 18 | [Public API & Webhooks](./18-public-api/)               | When the first integration request comes in | 5–10 days   | 📋 Outline  |
| 19 | [Discovery & Validation](./19-discovery-validation/)    | **Before any code**                    | 1–4 weeks           | 📋 Outline  |
| 20 | [Pricing Strategy](./20-pricing-strategy/)              | Before public launch                   | 2–5 days            | 📋 Outline  |
| 21 | [Brand & Design System](./21-brand-design-system/)      | Before any production UI               | 3–7 days            | 📋 Outline  |
| 22 | [Accessibility](./22-accessibility/)                    | Built-in from day 1                    | Continuous          | 📋 Outline  |
| 23 | [User Documentation & Help Center](./23-user-docs/)     | Before public launch                   | 2–5 days            | 📋 Outline  |
| 24 | [Launch Playbook](./24-launch-playbook/)                | 2–3 weeks before public launch         | 5–10 days           | 📋 Outline  |
| 25 | [SEO & GEO (AI Search)](./25-seo-geo/)                  | Before public launch; ongoing          | 2–5 days           | ✅ Filled    |

**Legend:** ✅ Full content available · 📋 Outline / scope only — content forthcoming.

---

## How this differs from the original 12-topic list

The user-supplied draft had 12 topics. The matrix grew through two
audits:

**First audit** (added topics 13–18) — observability, data ops, legal,
support+changelog, marketing+SEO, public API. Reasoning in
[`internal/topic-validation.md`](./internal/topic-validation.md).

**Second audit** (added topics 19–24) — discovery & validation, pricing
strategy, brand & design system, accessibility, user docs, launch
playbook. Triggered by the realization that this playbook is being used
to *plan a SaaS from scratch*, not just to ship one — so pre-build
research, pricing decisions, and brand foundations belong inside the
matrix.

Short version of the second audit:

- **Added 19 — Discovery & Validation** (numerically last, logically
  first; without this, the rest is wasted effort).
- **Added 20 — Pricing Strategy** (deciding prices is a different brain
  than implementing billing — kept separate from topic 03).
- **Added 21 — Brand & Design System** (design tokens + components
  before any UI work; foundational across both marketing and app).
- **Added 22 — Accessibility (a11y)** (WCAG 2.2, keyboard, screen
  readers — legally required in some jurisdictions and morally
  required everywhere).
- **Added 23 — User Documentation & Help Center** (knowledge base
  distinct from changelog and API docs).
- **Added 24 — Launch Playbook** (one-time, high-leverage event;
  Product Hunt, beta program, day-of mechanics).

**Third audit** (added topic 25) — SEO & GEO (AI Search), distilled from
four production sites' SEO/GEO work. Split out from topic 17 (Marketing
Site & SEO): 17 is about *building the marketing site*, 25 is the
cross-cutting *technical SEO + Generative Engine Optimization* discipline
(crawler access, JSON-LD `@graph`, `llms.txt`, citability, E-E-A-T,
validation). The unified `@graph` playbook moved from 17 into 25 so all
structured-data material lives together.

---

## How to use this playbook

**Solo founder reading top-to-bottom?** Start with topic 01 README, decide which
hosting model fits, then go down the matrix in order. Skip a topic only if your
product genuinely doesn't need it (e.g. 17 if you're B2B with no organic traffic
strategy, 18 if you have no integration partners).

**Joining an existing project?** Read the topic README closest to the bug or
feature you're working on. Each README includes a "When to implement" line — if
you're still pre-launch and the topic says "Month 2," skip it.

**Stuck on a specific problem?** Use `gotchas.md` files. Search the repo for
keywords; the gotchas are where the lessons live.

Each topic folder follows the same structure:

```
NN-topic/
├── README.md          # concept, why it matters, when to implement
├── checklist.md       # implementation checklist (sprint-ready)
├── best-practices.md  # industry references with links
├── gotchas.md         # common mistakes from real production
└── examples/
    ├── sveltekit-supabase.md
    ├── nextjs-prisma.md
    └── remix-planetscale.md
```

---

## Backlog, roles & process: the `agile/` folder

The topic folders tell you *how to build* each subsystem; [`tasks/`](./tasks/)
tracks *what's left to do*. The [`agile/`](./agile/) folder covers the
product-management layer in between — **what you're building, for whom, and in
what order**:

- [`agile/user-story-template.md`](./agile/user-story-template.md) — story format, fields, status legend, INVEST.
- [`agile/backlog-template.md`](./agile/backlog-template.md) — how to organize a backlog by epic/module.
- [`agile/roles-and-rbac.template.md`](./agile/roles-and-rbac.template.md) — role hierarchy + permission matrix to fill in.
- [`agile/process/`](./agile/process/) — the six SDLC phases and the BA/PO methodology in brief.
- [`agile/examples/industrial-shift-log.md`](./agile/examples/industrial-shift-log.md) — a worked, filled-in backlog excerpt.

Copy it into a new project alongside the AI onboarding kit below.

---

## Starting a new SaaS project? Use the AI onboarding kit

Every new project that uses this playbook should start with two
boilerplate files copied from [`ai/claude/`](./ai/claude/):

- **`CLAUDE.template.md`** → copy to your project root as `CLAUDE.md`.
  The "project constitution" — Claude Code reads it at the start of every
  session. Includes doc-sync rules and a CLAUDE.md self-update rule.
- **`DOC-GAP-AUDIT.template.md`** → copy to `internal-docs/DOC-GAP-AUDIT.md`.
  The single source of truth for documentation coverage, quality audit, and
  the Phase 1/2/3 roadmap.

Plus an **`END-OF-SESSION-CHECKLIST.md`** which is the 1-minute review
Claude (or you) runs at the end of each session to make sure code changes
are reflected in docs.

Two living examples of these templates fully filled in:

- **Grabit** (`bookolj/CLAUDE.md` + `bookolj/internal-docs/DOC-GAP-AUDIT.md`)
- **CutOptim** (`opticut/CLAUDE.md` + `opticut/internal-docs/DOC-GAP-AUDIT.md`)

See [`ai/claude/README.md`](./ai/claude/README.md) for usage details.

---

## Real-world references cited throughout

Whenever this playbook makes a claim like "Stripe does X," there is a link. The
benchmark products and the public sources used:

- **Stripe** — public API docs at [stripe.com/docs](https://stripe.com/docs),
  blog posts at [stripe.com/blog/engineering](https://stripe.com/blog/engineering).
  Cited for: idempotency, billing, webhooks, API design.
- **Linear** — public changelog at [linear.app/changelog](https://linear.app/changelog),
  blog at [linear.app/blog](https://linear.app/blog).
  Cited for: changelog publishing, keyboard-first UX, sync architecture.
- **Vercel** — docs at [vercel.com/docs](https://vercel.com/docs), engineering
  blog at [vercel.com/blog](https://vercel.com/blog).
  Cited for: edge deploys, environment management, preview deployments.
- **Notion** — public docs at [developers.notion.com](https://developers.notion.com).
  Cited for: workspace/multi-tenancy, public API design, OAuth.
- **Lemon Squeezy** — docs at [docs.lemonsqueezy.com](https://docs.lemonsqueezy.com).
  Cited for: merchant-of-record billing, store/product/variant model, webhook
  signing.
- **Supabase** — docs at [supabase.com/docs](https://supabase.com/docs).
  Cited for: row-level security patterns, auth, storage.

When a vendor changes its docs URL, file an issue.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). The bar is: every assertion is testable
or sourced. Every gotcha is a real incident. Toy code in `examples/` will be
rejected.

## License

[MIT](./LICENSE) — copy, adapt, ship.

## Versioning

See [CHANGELOG.md](./CHANGELOG.md). Topics are versioned individually; the root
version reflects the matrix shape (adding/removing topics).
