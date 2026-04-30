# 06 — Onboarding — Gotchas

> **Status:** Outline.

## Categories

1. **Empty state with no CTA.** User logs in, sees a blank page, leaves.
2. **Tour starts before page is loaded.** Tooltip points to a button
   that hasn't rendered yet.
3. **Sample data marked as "demo" but counted in usage.** Tenant hits
   "max 5 records on free tier" with 5 demo records.
4. **Email verification required to log in second time.** User signs
   up, doesn't verify, logs out, can't get back in.
5. **Trial countdown shows wrong timezone.** "5 days left" then ends
   at midnight UTC, customer thinks they were robbed of a day.
6. **OAuth-only signup blocks email signup.** A returning user with a
   different email can't reconcile.
7. **"Set up later" path becomes "never".** Setup checklist
   dismissable; users dismiss; product unused.

(Real incidents in v0.2.)
