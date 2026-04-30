# 13 — Observability & Monitoring — Best Practices

> **Status:** Outline.

## Sources

- Google SRE book.
- Sentry docs.
- Charity Majors' blog on observability.

## Practices to expand

- Alert on symptoms (users impacted), not causes (CPU%).
- Every alert must be actionable. If it isn't, downgrade to a daily
  digest.
- Status page is part of customer trust; update it within 5 minutes
  of an incident.
- Logs in structured JSON; one event = one log line.
- Don't log PII; redact at the source.
- SLO: pick one (e.g. 99.9% availability of `/healthz` per month).
  Track it. It's enough.
