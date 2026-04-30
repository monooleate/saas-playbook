# 06 — Onboarding

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** After first 10 users. Re-iterate every quarter.
> **Cost:** 3–5 days per major iteration.

## Scope

What happens between sign-up and first valuable use. The single
biggest lever on activation, retention, and trial-to-paid conversion.

Specific surfaces:
1. **Sign-up form** — minimum viable fields, social login first.
2. **Email verification** — required vs optional, when to gate.
3. **Workspace creation** — pick a workspace name, subdomain, country
   (sets currency for billing — see topic 03).
4. **First-run state** — empty state with sample data, guided tour, or
   blank canvas with copy.
5. **Aha-moment path** — the explicit shortest sequence from sign-up to
   the user experiencing the core value.
6. **Setup checklist** — Linear-style task list visible in-app.
7. **Drip campaign** — emails over the first 14 days nudging unused
   features.
8. **Trial-to-paid moment** — when the trial ends, what they see, what
   they do next.

## Why it matters

The data is consistent across SaaS:
- ~50% of trial signups never log in twice.
- Of those who do, ~50% activate (perform the core action) within 7 days.
- Activation rate is the single best predictor of paid conversion.

A solo founder cannot afford a generic "you're all set!" page. Every
unnecessary click in the first 5 minutes is paid for in churn.

## Patterns

### Sample data vs blank canvas

- **Sample data** (Notion, Linear's "demo workspace") — user sees a
  working example, less intimidating, but slow to "delete this and
  start over."
- **Blank canvas** (Figma, blank file) — user faces the empty state
  immediately.
- **Guided creation** (Webflow, Stripe) — user is walked through
  creating their first real artifact.

For most B2B SaaS, **guided creation of a real first artifact** beats
both. Sample data can confuse "is this mine?"; blank canvas is too
intimidating.

### Setup checklist

A persistent UI element ("3 of 7 steps complete") works for products
with multi-step setup. Linear's checklist is the canonical reference.
Risk: users dismiss it, lose track, and the unfinished setup blocks
value.

### Email drip cadence

- Day 0 — verify email + welcome.
- Day 1 — "Did you complete X?" if not.
- Day 3 — feature highlight (the most-loved feature).
- Day 7 — "Your trial is going well" or "Need help?"
- Day 11 — trial ending in 3 days.
- Day 13 — trial ending tomorrow.
- Day 14 — trial ended; reactivate.

## What goes in v0.2

- The 5-minute path: a written script of "starting from sign-up,
  what's the fastest possible path to value?"
- Setup checklist UI patterns.
- Tour libraries: react-joyride, intro.js, Driver.js — when to use,
  when to skip.
- Tracking activation: events to fire (sign-up, verified, workspace
  created, first artifact, invited collaborator).
- Funnel analysis (link to topic 08).

## Sources

- Wes Bush — *Product-Led Growth*. Onboarding chapter.
- Userpilot, Appcues, Pendo blogs.
- Linear's onboarding [video](https://linear.app) — copy the structure
  if not the implementation.
