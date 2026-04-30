# Sprint A — Discovery & Validation

> First sprint. The one that decides whether the rest of the playbook
> gets used on this product or whether you walk away with the saved
> months as winnings.

## Meta

- **Sprint number:** A (the first)
- **Dates:** 2026-05-04 (Mon, W19) → 2026-05-24 (Sun, W21) — 3 weeks
- **Owner:** taskony@gmail.com
- **Lifecycle stage:** 0 — Pre-foundations
- **Source row in MATRIX:** Topic 19 — Discovery & Validation (P0, XL)

## Goal

> By 2026-05-24, you have either (a) signed evidence that 3+ buyers
> will pay for the product (deposits, LOIs, or strong waitlist signal
> with credit-card-on-file pre-orders), **or** (b) the kill decision,
> documented, and the time freed.

This sprint does not produce code beyond a one-page smoke-test landing.
If you find yourself opening the SvelteKit / Next.js / Remix app
directory before week 3, you are ahead of validation and need to stop.

## Scope

### Story 1 — Problem statement + interview kit

- **Reference:** `../../19-discovery-validation/`, `checklist.md` →
  *Problem validation*
- **Effort:** S — 1 day
- **Target:** end of day Mon 2026-05-04
- **Acceptance:**
  - [ ] One-page problem statement, max 200 words. Names: who hurts,
        what they hurt at, what today's bad-options look like, why
        now.
  - [ ] Interview script written following The Mom Test rules
        (Rob Fitzpatrick). Five open questions about past behavior,
        zero questions about hypothetical future.
  - [ ] Recording + note-taking workflow set up (e.g. Otter.ai +
        Notion / Markdown notes).
  - [ ] List of 30 prospects to contact, with email/LinkedIn handle.
        Goal: 10 interviews from 30 outreach.

### Story 2 — 10 customer interviews

- **Reference:** `../../19-discovery-validation/`, `checklist.md` →
  *Problem validation*, `best-practices.md`
- **Effort:** L — 7–10 days, calendar-bound on prospects' availability
- **Target window:** Tue 2026-05-05 → Wed 2026-05-13
- **Acceptance:**
  - [ ] 10 interviews completed, recorded (with consent).
  - [ ] Each interview has a 5-bullet summary in the notes folder.
  - [ ] Pattern analysis: list the 3–5 statements that came up
        unprompted in 5+ of 10 interviews. These are the validated
        pains.
  - [ ] List of 3–5 statements that surprised you / contradicted your
        prior. These are where the product spec will pivot.
  - [ ] Updated problem statement reflecting the patterns.

### Story 3 — Competitive analysis

- **Reference:** `../../19-discovery-validation/`, `checklist.md` →
  *Competitive analysis*
- **Effort:** S — 1–2 days, can run in parallel with Story 2
- **Target window:** any time during Story 2 calendar gaps
- **Acceptance:**
  - [ ] Spreadsheet of 5–10 existing tools that solve the same or
        adjacent problem. Columns: name, pricing, target customer,
        what they do well, what they don't, "why a customer would
        switch to us."
  - [ ] One paragraph "why now" thesis: what changed in the world
        (regulation, technology, audience expectation) that makes the
        product viable now and not 3 years ago.
  - [ ] 2 of the 10 interviewees probed on whether they tried any of
        the top-3 competitors and why they stopped.

### Story 4 — Smoke test landing page

- **Reference:** `../../19-discovery-validation/`, `examples/sveltekit-supabase.md`
- **Effort:** M — 2–3 days
- **Target window:** Thu 2026-05-14 → Mon 2026-05-18
- **Acceptance:**
  - [ ] One-page Astro (or plain HTML) site live at a real domain.
        Above-fold: headline, value prop, screenshot or mockup, one
        CTA.
  - [ ] CTA goes to email-capture (Resend audience or Supabase row).
        Or, more aggressively: a Stripe Payment Link for €1
        "reserve your spot" — strongest validation possible.
  - [ ] Plausible / Umami analytics installed (privacy-friendly, no
        cookie banner needed).
  - [ ] $50–$200 of paid traffic from the channel where target
        customers actually live (Reddit Ads, Google Ads, X, niche
        newsletter sponsorship).
  - [ ] Conversion rate measured. Target: ≥ 2% email capture, or
        ≥ 0.5% pre-pay.

### Story 5 — MVP scope + kill criteria

- **Reference:** `../../19-discovery-validation/`, `checklist.md` →
  *MVP scope*, *Kill criteria*
- **Effort:** S — 1 day
- **Target:** end of day Fri 2026-05-22
- **Acceptance:**
  - [ ] One-page MVP scope document. Three "must have" features.
        Everything else is "later." Done-definition for MVP written
        in two sentences.
  - [ ] Kill criteria written: "If by 2026-08-15 I do not have N
        paying customers and a clear path to M, I will pivot or
        stop." N and M are specific.
  - [ ] Decision recorded in `tasks/sprints/2026-W19-discovery.md`
        retro: GO / PIVOT / KILL.

## Out of scope

- Writing any product code beyond the smoke-test landing.
  Discovery before construction; that's the whole point.
- Topic 21 (Brand & Design System) — wait until Sprint B unless the
  smoke-test landing forces a logo decision. A wordmark in Inter
  Bold is sufficient for this sprint.
- Topic 20 (Pricing Strategy) — informed by interviews, but the
  Van Westendorp 4-question test belongs in Sprint B with the real
  pricing-page work.

## Dependencies

- **External:**
  - Domain purchased + DNS configured (one-off, ~1 hour). If not done
    yet, do it Mon 2026-05-04 morning.
  - Resend / Postmark account for email capture.
  - Stripe account if pre-pay path chosen — KYB can take 1–7 days.
    Apply Mon 2026-05-04 to have it ready by Story 4.
  - $50–$200 ad spend budget, on a card.
- **Internal:** none — this is Sprint A.

## Risks

| # | Risk | Mitigation |
|---|------|------------|
| 1 | < 10 interviews booked from 30 outreach | Outreach 50 instead of 30. Personalize each message. Offer a 15-minute slot, not 30. Use Calendly. |
| 2 | Smoke test conversion < 2% | First failure is messaging, not product. Run two A/B variants; if both flat, the audience is wrong, not the product. |
| 3 | Discovery findings contradict existing direction | Welcome it. That's the whole point. Update problem statement; carry forward the surprises into MVP scope. |
| 4 | "Just one more interview" delays the decision | Hard date: 2026-05-22 retro. Decide with whatever data you have. Sunk cost is not data. |
| 5 | Stripe KYB takes longer than 7 days | Backup path: email capture only, no pre-pay. Slightly weaker signal but still actionable. |

## Daily log

> Update at end of each working day. One row, three fields.

| Date       | What shipped | Blocker |
|------------|--------------|---------|
| 2026-05-04 |              |         |
| 2026-05-05 |              |         |
| 2026-05-06 |              |         |
| 2026-05-07 |              |         |
| 2026-05-08 |              |         |
| 2026-05-11 |              |         |
| 2026-05-12 |              |         |
| 2026-05-13 |              |         |
| 2026-05-14 |              |         |
| 2026-05-15 |              |         |
| 2026-05-18 |              |         |
| 2026-05-19 |              |         |
| 2026-05-20 |              |         |
| 2026-05-21 |              |         |
| 2026-05-22 |              |         |

(Weekends omitted; add rows if you work them.)

## Mid-sprint check-in (Wed 2026-05-13)

End of Story 2's interview window. Answer in 3 bullets each:

- **Pattern check:** What 3 statements came up in 5+ interviews?
- **Surprise check:** What contradicts your prior?
- **Momentum check:** Are you on track for the smoke test next week, or
  do you need to extend Story 2 by 2–3 days? (Allowed; cut Story 4 ad
  spend by 50% to compensate.)

## Decision gate (Fri 2026-05-22)

The sprint exists to produce one of three outputs:

- **GO** — 3+ paying-intent signals (deposits / LOIs / smoke-test
  pre-pay / >2% email capture from cold traffic on a buyer-targeted
  channel). MVP scope clear. Kill criteria written. Move to Sprint B.
- **PIVOT** — interviews revealed a different problem than expected,
  but it's a real one. Re-run a tighter Sprint A1 of 1 week on the
  pivoted thesis.
- **KILL** — < 3 paying-intent signals, no clear pivot. Document
  what you learned in `KILL.md`, take the saved months as winnings.
  This is a successful sprint outcome too.

## Retro (fill at sprint end)

### What went well
- ...

### What didn't
- ...

### What we'd change next sprint
- ...

### Updates to MATRIX.md
- Topic 19 — Discovery & Validation: Status 🟡 → ✅ (or 🟡 → ❌
  blocked / pivoted, depending on outcome).
- Sprint B (Brand + Pricing) start date: ...

### Decision
- **Outcome:** GO / PIVOT / KILL
- **Reasoning (2 sentences):** ...
- **Next sprint file:** `tasks/sprints/2026-WXX-...md`
