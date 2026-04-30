# 05 — Feature Flags & Addon Gating — Checklist

> **Status:** Outline. Full per-item content forthcoming.

## Decisions

- Picked one feature flag tool (PostHog / LaunchDarkly / GrowthBook /
  homegrown).
- Decided where addon state lives (DB table — see topic 03).
- Documented "flag vs addon" decision rule.

## Plumbing

- `isEnabled(key, ctx)` function in the codebase.
- One call site per feature; no `tenant.plan === 'pro'` strings in
  business code.
- Flag eval cached per request (avoid N calls to PostHog per page).

## Lifecycle

- Every flag has an owner.
- Every flag has a target removal date in its description.
- Cleanup PR template: "this flag rolled out fully on DATE; remove
  branches and the flag itself."
- Addon registry (see topic 03) is the source of truth for paid
  features.

## Operational

- Per-tenant override UI in superadmin (topic 10) for support.
- Audit log of flag changes (who flipped what when).

(Each item to be expanded in v0.2.)
