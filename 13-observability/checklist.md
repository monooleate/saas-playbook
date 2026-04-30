# 13 — Observability & Monitoring — Checklist

> **Status:** Outline.

- Sentry SDK installed; DSN in env; test error fired in production and
  visible.
- Sentry alert rule: notify on first occurrence in 24h, throttle
  duplicates.
- `/healthz` endpoint returns 200 only if DB reachable.
- Uptime check from ≥2 regions, 5-minute interval.
- SMS + Slack on uptime failure.
- Public status page at `status.example.com`.
- Deploy webhook posts to Slack.
- Signups / payments / cancellations posted to `#events`.
- Daily cost report (cloud bills) posted weekly.
- Runbook in `internal-docs/RUNBOOK.md` covering the 5 most likely
  failures with steps to recover.

(Each item expanded with verification in v0.2.)
