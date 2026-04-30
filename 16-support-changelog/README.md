# 16 — Customer Support & Changelog

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** After first 20 users.
> **Cost:** 1–2 days.

## Scope

Two tightly-coupled growth surfaces that most starters skip:

1. **Customer support inbox** — single inbox for all customer email,
   triage, response templates.
2. **In-app feedback widget** — "got a question?" / "report a bug" /
   👍 / 👎 sentiment.
3. **Public changelog** — `/changelog` page with weekly or per-release
   updates. Linear, Notion, Stripe all have one.
4. **Public roadmap** (optional) — what's coming next.
5. **Status page** (cross-link to topic 13).

## Why it matters

- **Support** is your primary user research at small scale. You learn
  what's confusing, what's wanted, what's broken.
- **Changelog** is a retention tool: customers see continuous progress,
  trust the product, churn less. It's also a *marketing* tool — every
  changelog entry can be an X/Twitter post or LinkedIn update.
- **Public roadmap** signals direction. Reduces "is this product
  abandoned?" doubts.

## Recommended stack

- **Support inbox:** Help Scout (best DX, ~$25/mo), Front, Plain (dev-
  centric). Or Gmail with shared label until you outgrow it.
- **Feedback widget:** Featurebase, Canny, or homegrown form posting
  to a shared inbox.
- **Changelog:** publish as static pages on the marketing site.
  Format: title + 1-paragraph summary + screenshot or GIF + tag
  (Feature / Improvement / Fix). Link from the app's footer or a bell
  icon.
- **Roadmap:** Trello board (public), GitHub Projects, or Featurebase.
- **Status page:** BetterStack StatusPage or own static page driven
  by uptime data.

## Linear's pattern (highly studied)

Linear publishes [linear.app/changelog](https://linear.app/changelog)
as part of its main marketing site. Each entry:
- Has a date, version, and tag.
- Has a 30-second video or animated screenshot.
- Is written by the engineer who shipped, not the marketing team.
- Goes up the day of release.

Adopt this pattern: changelog isn't a chore; it's product marketing.

## What goes in v0.2

- Support response templates (the 10 most common questions).
- Inbox routing rules.
- In-app feedback widget patterns.
- Changelog publishing workflow.
- Roadmap publishing trade-offs.

## Sources

- Linear's changelog: [linear.app/changelog](https://linear.app/changelog).
- Stripe's changelog: [stripe.com/docs/changelog](https://stripe.com/docs/changelog).
- Notion: [notion.so/whats-new](https://www.notion.so/releases).
- Help Scout's "Support Driven" content.
