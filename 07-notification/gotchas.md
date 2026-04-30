# 07 — Notification & Automation — Gotchas

> **Status:** Outline.

## Categories

1. **Cron silently stops.** Vercel Cron / systemd timer broke; nobody
   noticed for weeks. Fix: heartbeat ping to a monitoring service.
2. **Same event delivered N times.** Cron runs every 5 minutes; job
   takes 7 minutes; two run in parallel and process the same events.
   Fix: lock + idempotency.
3. **Notification storm.** A bug fires `notify` in a loop;
   thousands of emails sent in minutes. Fix: rate-limit per-user.
4. **Time zones in scheduled emails.** "Daily digest at 9am" sends at
   9am UTC; customer in PT receives at 1am.
5. **Slack webhook URL leaked in logs.** Webhooks are bearer tokens.
6. **DST transitions skip or duplicate cron runs.** Fix: use UTC-aware
   schedulers.

(Real incidents in v0.2.)
