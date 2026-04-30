# 08 — Analytics & Reporting

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Month 2.
> **Cost:** 2–3 days.

## Scope

Two distinct surfaces, often built with the same tooling:

### 1. Product analytics (for you)

What you measure to make product decisions:
- Sign-ups, activations, retention curves.
- Funnel conversion (sign-up → workspace → first action → invited).
- Feature usage (which plan tier uses which features).
- Cohort retention.
- MRR / ARR / churn (link to topic 03).

### 2. Customer-facing reporting (for them)

What your customers measure inside your product:
- Their dashboards (revenue, bookings, tickets, whatever the SaaS is
  about).
- CSV / PDF exports.
- Scheduled email reports.

## Recommended stack

- **PostHog** for product analytics. Free tier covers 1M events/month.
  Has feature flags (topic 05) and session replay too.
- **Plausible** or **Umami** for marketing-site analytics (privacy-
  friendly, GDPR cookie-banner-free if configured right).
- **Customer-facing reports** built into the app, querying your DB
  directly. Don't put PostHog data into customer-facing UI.
- **MRR/ARR**: ChartMogul or Baremetrics free tiers plug into Stripe/
  Lemon and give you the numbers without writing code.

## The shared event taxonomy

A small, stable list of event names is worth a quarter of an
engineer's salary in saved confusion.

```
signup
email_verified
workspace_created
artifact_created
artifact_published
member_invited
integration_connected
plan_upgraded
plan_downgraded
trial_started
trial_ended
subscription_created
subscription_cancelled
```

Properties on each event include: `tenant_id`, `user_id`, `plan`,
`country_code`. Funnels and cohort analyses come for free.

## What goes in v0.2

- PostHog setup walkthrough (server-side + client-side SDKs, identity
  reconciliation across them).
- The minimum viable funnel dashboard.
- Customer-facing report patterns (charting libraries, CSV streaming,
  PDF generation — see Trackwell example for WeasyPrint).
- Privacy: PostHog GDPR settings, EU data residency option.
- KPI: MRR formula, ARR, churn, CAC, LTV — what to compute and when.

## Sources

- PostHog docs: [posthog.com/docs](https://posthog.com/docs).
- ChartMogul MRR formula:
  [chartmogul.com/help/getting-started/mrr-calculation/](https://chartmogul.com/help/getting-started/mrr-calculation/).
- Stripe revenue recognition docs.
