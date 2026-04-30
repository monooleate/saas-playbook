# Topic List Validation

This is the working document behind the matrix in the root `README.md`. It
records the audit of the 12-topic draft, what was changed, and why.

## Method

Each draft topic was checked against four questions:

1. **Standalone?** Can a solo founder read this topic in isolation and act on it?
2. **Atomic?** Or should it be split?
3. **Overlap?** Does it bleed into another topic such that we'll write the same
   content twice?
4. **Production-grade?** Compared with how Stripe / Linear / Vercel / Notion /
   Lemon Squeezy expose this concern publicly, is anything missing?

Verdict on each:

## 01 — Infrastructure & DevOps — KEEP, narrow scope

- Standalone: yes.
- Atomic: no — observability (logs, metrics, alerting, error tracking) is
  hiding inside it. Pull out as topic 13.
- Overlap with Security (11): TLS / SSH hardening overlaps. Recommendation:
  keep TLS basics in 01 (Caddy / Let's Encrypt), keep secrets management +
  pen-test posture in 11.
- Missing: nothing else. Scope: provisioning, DNS, TLS, reverse proxy, process
  manager, deploy pipeline, env management, scaling thresholds.

## 02 — Auth & Multi-tenancy — KEEP, but acknowledge two halves

- Atomic: no, technically two concerns. But a solo founder almost always
  builds them together; splitting would force constant cross-references.
- Decision: one topic, two clear sections (`Auth` and `Tenancy`).
- Missing: SSO / SCIM is enterprise; defer to "Future" callout.

## 03 — Billing & Subscription — KEEP

- Standalone: yes. The largest topic by content volume.
- Missing: dunning / failed payment recovery. Add as a section.
- Overlap with 09 (i18n): multi-currency. Decision — multi-currency lives in
  03 (it's a billing feature); 09 covers locale/translations.
- Overlap with 05 (feature flags / addons): plan→feature mapping. Decision —
  03 owns "what plan a tenant has," 05 owns "what features that unlocks." The
  contract between them: a single `entitlements` table.

## 04 — Email & Communication — KEEP, narrow scope

- Decision: this topic is *transactional + marketing email infrastructure*.
  In-app notifications go to topic 07.
- Missing: domain warm-up, DKIM/SPF/DMARC, deliverability monitoring.

## 05 — Feature Flags & Addon Gating — KEEP

- Atomic: feature flags (LaunchDarkly-style) and entitlements (plan-based)
  are different mechanisms with the same query surface (`isEnabled('foo')`).
  Decision: one topic, two sections, explicit comparison table.
- Missing: kill-switch pattern, per-tenant overrides for support.

## 06 — Onboarding — KEEP

- Standalone: yes.
- Missing: empty state design, sample data seeding.

## 07 — Notification & Automation — KEEP

- Scope: in-app notification center, cross-channel routing, scheduled jobs
  (cron), drip campaigns, queues. Email *infrastructure* is in 04; 07 uses 04
  as a transport.
- Missing: idempotent cron jobs, run-once vs. catch-up semantics.

## 08 — Analytics & Reporting — KEEP

- Atomic: product analytics (PostHog) and customer-facing reporting (CSV
  exports, dashboards inside the app) share little. Decision: one topic, two
  sections, since solo founders implement both with the same tooling early on.
- Missing: cohort analysis, MRR/ARR/churn formulas (link to ChartMogul docs).

## 09 — Internationalization — KEEP, narrow scope

- Decision: this is *strings + dates + numbers + locale routing*.
- Multi-currency lives in 03.
- Missing: RTL languages, pluralization, locale-aware URL strategies.

## 10 — Superadmin — KEEP

- Standalone: yes. Underrated topic; many starters skip it.
- Missing: action audit log, "act as user" with a banner, soft-delete recovery.

## 11 — Security — KEEP

- Standalone: yes.
- Missing: secrets rotation, dependency scanning, OWASP top-10 self-audit.

## 12 — Testing — KEEP

- Standalone: yes.
- Missing: when *not* to test (the solo-founder anti-overtest rule).

---

## Added topics

### 13 — Observability & Monitoring (NEW)

Pulled out of 01. Sentry, structured logs, metrics, uptime checks, alerting,
on-call procedure for a solo founder (= phone notifications + a runbook).
Stripe runs `status.stripe.com` as a public service; Linear publishes incident
post-mortems. Solo founder version: a public status page + a 5-line incident
template.

### 14 — Data & Database Operations (NEW)

Schema migrations, backups + restore drills, RLS, soft-delete vs. hard-delete,
audit log table, GDPR data export and delete. This was about to scatter
across 02, 10, 11, 15. One topic, one mental model.

Why it matters: more solo SaaS founders lose customers to a botched migration
than to feature gaps.

### 15 — Legal & Compliance (NEW)

Terms of Service, Privacy Policy, GDPR data export and account deletion
endpoints, cookie banner, ToS versioning, DPA template, consent records.

Why now: required before first paying EU customer. Solo founders postpone
this until pulled into a compliance fire drill. The playbook pulls it forward.

### 16 — Customer Support & Changelog (NEW)

Inbox routing, status page, public changelog publishing, in-app feedback
widget, NPS or simple thumbs up/down.

Reference: Linear's [/changelog](https://linear.app/changelog) is a marketing
and retention tool, not an afterthought. Stripe's
[/changelog](https://stripe.com/docs/changelog) is structured by API version.

### 17 — Marketing Site & SEO (NEW)

The marketing site (typically a separate codebase or a separate workspace in
the monorepo) needs its own treatment: static generation, OG image pipeline,
sitemap, blog, programmatic SEO patterns, A/B testing landing pages.

Why now: for a solo founder, this is the *only* organic acquisition channel
in year 1. Skipping it because "the app is more important" loses the company
6 months of compound growth.

### 18 — Public API & Webhooks (NEW)

Outbound public API: API keys, rate limits, versioning, webhook delivery with
retries + signing, OpenAPI spec.

May be N/A for v1 of many SaaS, but the moment one customer asks "do you have
an API?", the playbook should already have the answer.

---

## Topics considered and rejected

- **Search & Discovery** (Algolia / Postgres FTS / Typesense). Too narrow;
  belongs as a section inside whichever topic uses it (usually 14 — Data).
- **AI & ML features.** Too product-specific. Will revisit if the playbook
  gains an "advanced" tier.
- **Mobile apps.** Out of scope; the playbook targets web SaaS.
- **Real-time / WebSockets.** Worth a section in 14 (data layer) and 07
  (notifications) but not its own topic.

---

## Final shape (after second audit, 2026-05-01)

The playbook outgrew its original technical-implementation framing. As
the founder started using it to *plan* a SaaS (not just ship one), gaps
appeared — pre-build research, pricing decisions, brand foundations,
accessibility, user docs, the launch event. Six topics added:

- **19 — Discovery & Validation** — pre-build research. Without this,
  every other topic is overhead on the wrong product.
- **20 — Pricing Strategy** — *deciding* prices vs implementing billing.
  Two distinct concerns; kept apart so each gets full treatment.
- **21 — Brand & Design System** — tokens, primitives, identity.
  Foundational; if missed, it becomes a half-built mess in month 6.
- **22 — Accessibility (a11y)** — WCAG 2.2 AA target. Both ethically
  and legally required (EU EAA in force June 2025).
- **23 — User Documentation & Help Center** — knowledge base distinct
  from changelog (16) and API docs (18).
- **24 — Launch Playbook** — one-time, high-leverage event. PH, HN,
  beta program coordination.

## Final shape: 24 topics, grouped by SaaS lifecycle stage

- *Pre-foundations:* 19 (Discovery)
- *Foundations:* 01 (Infra), 02 (Auth+Multi-tenancy), 03 (Billing),
  11 (Security), 14 (Data Ops), 20 (Pricing strategy), 21 (Brand)
- *Growth:* 04 (Email), 06 (Onboarding), 08 (Analytics), 16 (Support),
  17 (Marketing+SEO), 23 (User docs), 24 (Launch)
- *Operations:* 07 (Notification), 09 (i18n), 10 (Superadmin),
  12 (Testing), 13 (Observability), 15 (Legal), 22 (a11y)
- *Extension:* 05 (Feature flags), 18 (Public API)

The numeric IDs preserve matrix order; the lifecycle ordering for
reading is in the root [`ROADMAP.md`](../ROADMAP.md), and the sprint-
ready row-per-topic view is in [`tasks/MATRIX.md`](../tasks/MATRIX.md).

## Topics still considered and rejected

After the second audit, these remain rejected:

- **AI features integration** — too product-specific; varies wildly by
  vertical. Better as a section inside the topic that uses the AI
  feature.
- **Customer success / activation playbook** — sub-section of 06
  (Onboarding) and 16 (Support).
- **Sales process for B2B** — too GTM-specific.
- **Mobile apps** — out of scope (web SaaS focus).
- **Affiliate program** — sub-section of 17 (Marketing).
- **Hiring / first contractor** — not technical enough; varies by
  founder.
- **Multi-region / latency** — sub-section of 01 (Infrastructure).
- **Embedded / white-label** — too product-specific.
- **Migration / Import from competitors** — important for B2B but
  product-specific; mention in topic 06 (Onboarding).
