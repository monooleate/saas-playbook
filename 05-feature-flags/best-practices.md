# 05 — Feature Flags & Addon Gating — Best Practices

> **Status:** Outline.

## Sources

- PostHog: [posthog.com/docs/feature-flags](https://posthog.com/docs/feature-flags).
- LaunchDarkly Effective Feature Management.
- GrowthBook docs.

## Practices to expand

- One feature, one flag. Don't bundle multiple unrelated features
  behind one flag — you can't roll them out independently.
- Flags die. Every flag created should have a kill date.
- Server-side flags > client-side flags for security-relevant gating.
  A client flag is a nice-to-have toggle, not a paywall.
- Default: closed. New flag = `false` until explicitly enabled.
- Plan/addon gating goes in the DB; A/B/percentage flags go in the
  flag service.
- "Trial with all features" → "Free tier with limited features" is a
  common product evolution. Plan it for from day 1: feature mapping
  must support both shapes.

(Quotes and examples in v0.2.)
