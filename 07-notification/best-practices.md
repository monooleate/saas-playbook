# 07 — Notification & Automation — Best Practices

> **Status:** Outline.

## Sources to cite

- Inngest docs.
- Linear's notification settings UI as a reference.
- Stripe's webhook patterns (cross-link to topic 03).

## Practices

- Critical comms inline; non-critical async.
- Every cron job is idempotent.
- Notifications are commands the user can act on; if there's no action,
  it's a log line, not a notification.
- Default to fewer notifications, not more.
- Dedupe within a tight window: 5 likes in 1 minute = one notification.
