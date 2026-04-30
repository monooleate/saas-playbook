# Topic Matrix — Sprint Planning

One row per topic. Sortable, sliceable, sprint-ready.

**Legend.** Priority: P0 must, P1 should, P2 nice-to-have. Effort:
S ≤1d, M 2–4d, L 5–10d, XL >10d. Status: ✅ done, 🟡 in-progress, 📋
outline, ❌ blocked.

**How to use.**
1. Filter by **Stage** (lifecycle) — see `../ROADMAP.md` for stage
   definitions.
2. Pick rows where dependencies are already ✅.
3. Sum effort to ~1–2 weeks; that's a sprint.
4. Copy details into `sprint-template.md`.

---

## Master matrix (24 rows)

| #  | Topic | Stage | Priority | Effort | Status | Depends on | Remaining work |
|----|-------|-------|----------|--------|--------|------------|----------------|
| 19 | Discovery & Validation | 0 — Pre-foundations | P0 | XL | 🟡 | — | **Sprint A scheduled: [`sprints/2026-W19-discovery.md`](./sprints/2026-W19-discovery.md) (2026-05-04 → 2026-05-24).** Outputs: validated MVP scope, paying-intent signals, GO/PIVOT/KILL decision. |
| 21 | Brand & Design System | 1 — Foundations | P0 | M | 📋 | 19 | Fill all 4 supporting files. Add `tailwind.config.ts` reference, shadcn/ui setup walkthrough, dark-mode token strategy. |
| 20 | Pricing Strategy | 1 — Foundations | P0 | M | 📋 | 19 | Van Westendorp PSM walkthrough, pricing page screenshots study, "raise prices" playbook. |
| 01 | Infrastructure & DevOps | 1 — Foundations | P0 | M | ✅ | — | (Done) |
| 14 | Data & Database Operations | 1 — Foundations | P0 | M | 📋 | 01 | Per-stack migration walkthroughs, RLS policy patterns, soft-delete pattern, GDPR endpoint pattern, audit log table schema. |
| 02 | Auth & Multi-tenancy | 1 — Foundations | P0 | M | 📋 | 01, 14 | Port Grabit's `hooks.server.ts` tenant resolver. RLS policies. Membership + invite flow. Switch-tenant UI. |
| 11 | Security | 1 — Foundations | P0 | Continuous | 📋 | 01 | OWASP top-10 walkthrough, secrets rotation procedure, rate-limit patterns, security-headers checklist. |
| 03 | Billing & Subscription | 1 — Foundations | P0 | L | ✅ | 02, 14, 20 | (Done) |
| 04 | Email & Communication | 2 — Growth | P0 | S | 📋 | 01, 14 | DKIM/SPF/DMARC walkthrough, 10 transactional templates, idempotent send pattern, suppression-list management. |
| 22 | Accessibility | 2 — Growth | P0 | Continuous | 📋 | 21 | Component-by-component a11y checklist, keyboard testing script, screen reader playbook, axe-core CI integration. |
| 06 | Onboarding | 2 — Growth | P0 | M | 📋 | 02, 04, 21 | 5-minute path script, setup checklist UI patterns, drip cadence, activation event taxonomy. |
| 17 | Marketing Site & SEO | 2 — Growth | P0 | L | 📋 | 21 | Astro setup walkthrough, OG image generation, sitemap generation, programmatic SEO templates. |
| 15 | Legal & Compliance | 2 — Growth | P0 | S | 📋 | — | Template ToS structure, cookie banner decision tree, GDPR endpoint patterns, DPA template, consent record schema. |
| 23 | User Documentation | 2 — Growth | P1 | M | 📋 | 17 | Astro Starlight setup, article authoring guide, search setup, inline-help patterns. |
| 16 | Customer Support & Changelog | 2 — Growth | P1 | S | 📋 | 17 | 10 canned response templates, changelog publishing workflow, status-page patterns, feedback widget. |
| 13 | Observability & Monitoring | 3 — Pre-launch | P0 | M | 📋 | 01, 11 | Sentry setup, structured logging, uptime monitoring providers compared, status page setup, runbook template. |
| 10 | Superadmin | 3 — Pre-launch | P0 | S | 📋 | 02, 03 | Auth check + middleware, tenant detail UI sketch, audit-log schema, soft-delete recovery UI, "sign in as user" pattern. |
| 12 | Testing | 3 — Pre-launch | P1 | M | 📋 | All P0 done | Vitest + Playwright setup, DB-touching test patterns, CI runner config, RLS isolation test. |
| 24 | Launch Playbook | 3 — Pre-launch | P0 | L | 📋 | All Stage-2 done | Beta recruitment template, PH pre-launch checklist, email sequences, day-of community engagement, post-launch retro template. |
| 08 | Analytics & Reporting | 4 — Operations | P1 | M | 📋 | 06 | PostHog walkthrough, event taxonomy, MRR formula, customer-facing report patterns, CSV/PDF export. |
| 07 | Notification & Automation | 4 — Operations | P1 | M | 📋 | 04 | In-app `notifications` table, cron scheduler patterns, queue-when-needed guide, Slack/Discord webhook patterns. |
| 05 | Feature Flags & Addon Gating | 4 — Operations | P1 | M | 📋 | 03 | PostHog feature flags walkthrough, entitlements table pattern, per-user vs per-tenant flags, cleanup discipline. |
| 09 | Internationalization | 4 — Operations | P2 | L | 📋 | 06, 17 | Paraglide / next-intl / i18next setup, ICU MessageFormat patterns, plural rules, RTL CSS, hreflang. |
| 18 | Public API & Webhooks | 4 — Extension | P2 | L | 📋 | 02, 03, 14 | API design walkthrough, API key model, rate-limit patterns, webhook delivery service, OpenAPI generation, SDKs. |

---

## By priority — quick filter

### P0 (must) — 14 rows
19, 21, 20, 01✅, 14, 02, 11, 03✅, 04, 22, 06, 17, 15, 13, 10, 24

(01 and 03 already ✅ — leaving 14 P0 rows of remaining work.)

### P1 (should) — 6 rows
23, 16, 12, 08, 07, 05

### P2 (nice-to-have) — 2 rows
09, 18

---

## Suggested sprint groupings

These are *suggestions*; adjust based on what you've already shipped.

### Sprint A — "Discovery is the work" (1–4 weeks)
- 19 Discovery & Validation (XL)

Output: validated MVP scope, paying-intent signals.

### Sprint B — "Look + price" (1 week)
- 21 Brand & Design System (M)
- 20 Pricing Strategy (M)

Output: design tokens + components, public pricing page mockup.

### Sprint C — "Skeleton" (1 week)
- 14 Data Operations P0 part (M)
- 02 Auth & Multi-tenancy (M)
- 11 Security headers + rate limits (S)

Output: deployable app with tenants and locked-down access.

### Sprint D — "Money in" (1–2 weeks)
- 03 Billing already ✅ — wire to live keys
- 04 Email & Communication (S)
- 15 Legal & Compliance (S)

Output: first real charge possible.

### Sprint E — "Activate" (1–2 weeks)
- 22 Accessibility build-in (continuous, but front-load components)
- 06 Onboarding (M)
- 17 Marketing Site & SEO (L)

Output: public-facing product someone can find and sign up for.

### Sprint F — "Pre-launch" (1 week)
- 23 User Docs cornerstone articles (M)
- 16 Support & Changelog (S)
- 13 Observability (M)
- 10 Superadmin (S)

Output: post-launch operations are possible.

### Sprint G — "Launch" (1 week, concentrated)
- 24 Launch Playbook (L)

Output: Product Hunt #N, day-1 metrics.

### Sprint H+ — "Operate" (ongoing)
- Pull from P1/P2 rows based on signal.

---

## When you complete a row

1. Set Status to ✅.
2. Update `../README.md` matrix to ✅.
3. Note the date in `../CHANGELOG.md` under the next minor version.
4. If new sub-tasks were discovered, add them as separate rows below
   (or as bullets in the row's Remaining-work cell).

---

## Active sprints

| Sprint | Window | Topics | File | Status |
|--------|--------|--------|------|--------|
| A — Discovery | 2026-05-04 → 2026-05-24 | 19 | [`sprints/2026-W19-discovery.md`](./sprints/2026-W19-discovery.md) | 🟡 scheduled |

## Last refresh

This matrix reflects the playbook state on **2026-05-01**. Each row's
remaining-work cell is the snapshot at that date; refine it as work
progresses.
