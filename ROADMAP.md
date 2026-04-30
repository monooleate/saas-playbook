# Roadmap — Reading & Implementation Order

The numeric topic IDs (01–24) preserve the matrix order in `README.md`.
This file rearranges the same 24 topics into the **order you'd actually
implement them** in a real SaaS.

If you're using the playbook to plan a new SaaS:
1. Read this roadmap top to bottom.
2. Open [`tasks/MATRIX.md`](./tasks/MATRIX.md) — same 24 rows with
   effort, priority, dependencies.
3. Build sprints from `tasks/MATRIX.md`, store sprint plans in
   `tasks/sprints/`.

---

## Stage 0 — Pre-foundations (before any code)

Goal: prove the problem is real and someone will pay you.

| # | Topic | Time |
|---|-------|------|
| **19** | Discovery & Validation | 1–4 weeks |

Output: a one-page MVP scope, 3+ paying-intent signals (deposits,
LOIs, or strong waitlist).

---

## Stage 1 — Foundations (first 2–4 weeks of build)

Goal: a deployable, billable, secure skeleton.

| # | Topic | Time | Why this order |
|---|-------|------|----------------|
| **21** | Brand & Design System | 3–7 days | Tokens before components; everything else inherits these |
| **20** | Pricing Strategy | 2–5 days | You need the price model before you can implement billing |
| **01** | Infrastructure & DevOps | 1–3 days | Where the code runs |
| **14** | Data & Database Operations | 2–5 days (P0 part) | Migrations + RLS + backups; the cost of getting this wrong is high |
| **02** | Auth & Multi-tenancy | 3–5 days | Without it, you don't have tenants; without tenants, no SaaS |
| **11** | Security | Continuous | Headers, secrets, dep scanning — set up day 1, watch forever |
| **03** | Billing & Subscription | 5–10 days | The whole point of SaaS is recurring revenue |

Output: a deployable app with auth, tenants, the first plan, and HTTPS.

---

## Stage 2 — Growth surface (weeks 4–8)

Goal: turn deployable into something someone can sign up for.

| # | Topic | Time | Why this order |
|---|-------|------|----------------|
| **04** | Email & Communication | 1–2 days | Verify, reset, receipts — required for sign-up |
| **22** | Accessibility | Continuous | Build into components from day 1; cheaper than retrofitting |
| **06** | Onboarding | 3–5 days | Activation is the conversion lever |
| **17** | Marketing Site & SEO | 5–15 days | Public face of the product |
| **15** | Legal & Compliance | 1–3 days | ToS, Privacy, GDPR endpoints — required before first paying customer |
| **23** | User Documentation | 2–5 days | Cornerstone articles before launch |
| **16** | Customer Support & Changelog | 1–2 days | Inbox + status page + changelog page |

Output: a product the public can find, sign up for, pay for, and learn
to use.

---

## Stage 3 — Pre-launch & Launch (weeks 8–10)

Goal: maximize the one launch moment.

| # | Topic | Time |
|---|-------|------|
| **13** | Observability & Monitoring | 1–3 days |
| **10** | Superadmin | 1–2 days |
| **12** | Testing | 2–10 days |
| **24** | Launch Playbook | 5–10 days, concentrated |

Output: launched product, day-1 metrics, post-launch retro.

---

## Stage 4 — Operations (post-launch, ongoing)

Goal: stay alive, stay paid, keep learning.

| # | Topic | When the cost shows up |
|---|-------|------------------------|
| **08** | Analytics & Reporting | Month 2 |
| **07** | Notification & Automation | After 50 active tenants |
| **05** | Feature Flags & Addon Gating | When you need gradual rollouts |
| **09** | Internationalization | Before second-country launch |
| **18** | Public API & Webhooks | When the first integration request comes in |

Output: a SaaS that compounds.

---

## Anti-patterns

- **Skipping 19 and going straight to 01.** You will build the wrong
  thing. Fix: spend the 1–4 weeks.
- **Skipping 20 and going straight to 03.** You will under-price.
  Fix: do the customer interviews first.
- **Skipping 21 and going straight to 06 / 17.** Your design will
  diverge across the marketing site and the app. Fix: tokens first,
  components second, pages last.
- **Doing 12 (Testing) before 03 (Billing) is shipped.** Premature
  testing for a product that doesn't yet have customers. Fix: ship
  first, harden second.
- **Saving 22 (a11y) for "later".** "Later" never arrives, retrofit
  is 10× the cost. Fix: build into the design system in Stage 1.
- **Saving 15 (Legal) for "after we have customers".** EU GDPR fines
  exist; B2B buyers ask for DPA before signing. Fix: do it in Stage 2.

---

## How long does the whole thing take?

Realistic for a solo founder building full-time:

| Stage | Calendar time |
|-------|---------------|
| Stage 0 — Discovery | 2–4 weeks |
| Stage 1 — Foundations | 3–5 weeks |
| Stage 2 — Growth surface | 4–6 weeks |
| Stage 3 — Pre-launch & Launch | 2 weeks |
| **Pre-launch total** | **~3–4 months** |
| Stage 4 — Operations | Indefinite |

The largest variable is Stage 1 — pre-existing experience with the
chosen stack cuts this in half.
