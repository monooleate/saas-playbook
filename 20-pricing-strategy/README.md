# 20 — Pricing Strategy

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before public launch; revisit every 6 months.
> **Cost:** 2–5 days for the first model; ongoing tuning.
> **Belongs to:** *Foundations*.

## Scope

How you *choose* prices and packaging. The implementation
(`pricing.json`, plan switching, addon billing) lives in topic **03 —
Billing & Subscription**. This topic is about the decisions that go
into `pricing.json`, not how to spell it.

1. **Pricing model** — flat, tiered, per-seat, per-usage, freemium,
   reverse-trial.
2. **Anchor price** — the headline number that sets perception.
3. **Tier design** — Starter / Pro / Enterprise. What goes where.
4. **Addons vs. tiers** — when to surface a feature as an addon vs.
   bundle into a tier.
5. **Annual discount** — typical 17–20% off; messaging.
6. **Country-specific pricing** — purchasing-power parity (PPP),
   when to use, when to skip.
7. **Free trial vs. freemium vs. money-back guarantee.**
8. **Pricing page design** — the comparison table; the social proof
   pattern.
9. **Pricing experiments** — how to A/B price without breaking trust.
10. **Raising prices** — when and how.

## Why it matters

Pricing is the highest-leverage variable in your business. A 10%
better price is 10% more profit, indefinitely. Solo founders almost
universally underprice — it costs them 50%+ of potential revenue for
the life of the company.

Patrick McKenzie ("patio11") is the canonical Western reference on
SaaS pricing for solo founders; his blog at
[kalzumeus.com](https://www.kalzumeus.com/) is essentially a free
graduate course.

## The four pricing models that matter

| Model       | When it works                                  | Risk                                              |
|-------------|------------------------------------------------|---------------------------------------------------|
| Flat tier   | Simple value, similar customers                | Low ARPU, leaves money on the table               |
| Per-seat    | Team SaaS where seats correlate with value     | Customer games the count, complexity              |
| Per-usage   | Value scales with consumption (API calls)      | Bills surprise customers; harder to forecast      |
| Hybrid      | Most successful B2B SaaS                       | Implementation complexity                          |

Stripe, Linear, Notion all use **hybrid**: flat per-seat with usage-
metered overages on certain features.

## Patterns

### The 10/10/10 rule (Patrick Campbell)

For early SaaS:
- Talk to **10 customers** in your target segment.
- Ask each what they'd pay (using the Van Westendorp 4-question
  technique).
- Set the price **10% higher** than the median acceptable price.
- Re-evaluate every **10 customers** added.

### Anchor + decoy

Show three tiers. Make the middle tier the anchor and the obvious
choice. The high tier exists to make the middle look reasonable.
Notion's pricing page does this perfectly.

### Annual discount messaging

"$20/mo, billed monthly" vs "$192/year, billed annually (save $48)" —
the second wins by ~30% on conversion in most A/B tests. Always show
both.

## What goes in v0.2

- Van Westendorp PSM (Price Sensitivity Meter) walkthrough.
- Pricing page design patterns (annotated screenshots from Stripe,
  Linear, Notion, Lemon Squeezy).
- Addon-vs-tier decision matrix.
- "How to raise prices without losing customers" playbook.
- PPP (purchasing power parity) — when worth it; risks of arbitrage.

## Sources

- [kalzumeus.com / patio11](https://www.kalzumeus.com/) — pricing
  essays.
- ProfitWell / Paddle Studios — SaaS pricing research.
- *Monetizing Innovation* — Madhavan Ramanujam.
- The Notion / Linear / Stripe pricing pages as case studies.
