# 07 — Notification & Automation — Checklist

> **Status:** Outline.

- `notifications` table + `notify()` helper.
- Notification bell + center UI in app.
- Cron job runner (Vercel Cron / systemd / Fly Machines).
- Each cron job is idempotent; logs to `cron_runs` table.
- Critical comms (trial-ending, past-due) routed both inline AND via
  daily reconciliation cron — backstop pattern.
- User notification preferences screen.
- Slack/Discord webhook for ops events.
- Queue layer documented and chosen (or marked as "not yet needed").
