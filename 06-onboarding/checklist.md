# 06 — Onboarding — Checklist

> **Status:** Outline.

## Sign-up

- Single-page sign-up form.
- Google OAuth as the primary CTA, email/password secondary.
- Email field validated client + server.
- Marketing consent checkbox separate from ToS (GDPR — topic 15).

## First-run

- Workspace creation: name, subdomain (auto-suggested + editable),
  country.
- Skip-able tour, restartable from settings.
- One CTA visible: "Create your first X."

## Activation tracking

- Events fired: `signup`, `email_verified`, `workspace_created`,
  `first_artifact_created`, `invited_member`, `connected_integration`.
- Funnel viewable in topic 08 analytics tool.

## Drip

- Verification email sent on signup.
- Welcome email after verification.
- Day-3, day-7, day-11, day-13 emails.
- All idempotent — no duplicate sends.

## Trial expiration

- In-app banner from day 11: "Trial ends in N days."
- Day 14: trial ended page with reactivate CTA.
- Past 14: data preserved for 30 days; deletion flow per topic 15.

(Each step expanded in v0.2.)
