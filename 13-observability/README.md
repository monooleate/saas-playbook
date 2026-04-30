# 13 — Observability & Monitoring

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Sentry on day 1; metrics in month 2.
> **Cost:** 1–3 days for Sentry + uptime + dashboards.

## Scope

The four pillars at solo scale:

1. **Errors** — Sentry (or equivalent: Highlight, Bugsnag, GlitchTip
   for self-hosted).
2. **Logs** — structured JSON, shipped to a log aggregator (Better
   Stack, Axiom, Datadog at scale).
3. **Metrics** — request rate, latency, error rate. Vercel Analytics,
   Plausible, or custom Grafana for VPS.
4. **Uptime / synthetic checks** — UptimeRobot, BetterStack,
   Cronitor.

## Why split from topic 01

Infrastructure (topic 01) gets you the *server*; observability is what
tells you the server is doing what it should.

Solo-founder reality: the alerting is more important than the
dashboards. You don't have time to look at dashboards. You need a
phone notification when something is wrong, and a *runbook* telling
you what to do.

## What "good" looks like

Minimum viable observability for a solo SaaS:

- **Sentry**: every uncaught exception in app and webhooks land here.
  Slack notification on first occurrence in last 24h.
- **Uptime check**: HTTPS hit on `/healthz` from 2+ regions, every 5
  minutes. SMS + Slack on failure.
- **Status page**: a public `status.example.com` that customers can
  check. BetterStack and StatusPage have free tiers.
- **Deploy notifications**: every deploy posts to a Slack channel
  with the commit + author.
- **Critical events stream**: signups, payments, cancellations posted
  to `#events` Slack. Lets you smell trouble before alerts fire.

## What goes in v0.2

- Sentry setup walkthrough (SDK install + alert rules).
- Structured logging pattern and shipping options.
- Metrics: what to graph, alert thresholds.
- Uptime monitoring providers compared.
- Status page setup.
- The "phone-runbook" pattern for incident response.

## Sources

- Sentry docs: [docs.sentry.io](https://docs.sentry.io).
- BetterStack uptime: [betterstack.com/uptime](https://betterstack.com/uptime).
- Google SRE book — "Service Level Objectives" chapter is foundational.
- StatusPage atom feeds for inspiration.
