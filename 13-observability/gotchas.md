# 13 — Observability & Monitoring — Gotchas

> **Status:** Outline.

## Categories

1. **Sentry quota burned by a noisy bug.** Add a server-side
   `beforeSend` filter for known noise.
2. **Healthcheck returns 200 always.** See topic 01 G + best-practice 11.
3. **Alerts fired but founder asleep.** Use SMS or Pushover for
   genuine pages; reserve Slack for low-urgency.
4. **Status page lies because manual.** Use a status page that pulls
   from your uptime provider directly.
5. **Logs leak PII.** Audit by grep for `email`, `name`, `card`.
6. **Trace IDs missing.** Adding them to every log line later is hard;
   add a `requestId` middleware on day 1.
7. **Vendor lock-in via SDK.** OpenTelemetry as a generic alternative
   when scale arrives.

(Real incidents in v0.2.)
