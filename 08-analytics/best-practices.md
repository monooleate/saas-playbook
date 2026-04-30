# 08 — Analytics & Reporting — Best Practices

> **Status:** Outline.

## Sources

- PostHog blog.
- Amplitude product analytics playbook.
- ChartMogul help docs.

## Practices

- Event names are imperative + past-tense (`workspace_created`, not
  `creation`). Stable taxonomy beats clever taxonomy.
- Identify users early; tie anonymous events to user events on signup.
- Don't over-instrument. 30 events is plenty for year 1.
- Don't bake business logic into your analytics tool's transformations
  — it'll be wrong, slow to fix, and tied to that tool.
- Customer-facing reports query the production DB; product analytics
  query the analytics warehouse. Keep them separate.
