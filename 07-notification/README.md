# 07 — Notification & Automation

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** After 50 active tenants.
> **Cost:** 2–4 days for in-app + cron skeleton.

## Scope

The orchestration layer that decides "this thing happened — who should
know, via which channel?"

1. **In-app notifications** — bell icon, notification center, unread
   count.
2. **Cross-channel routing** — same event → email (topic 04), SMS,
   push, Slack/Discord webhook.
3. **Scheduled jobs / cron** — daily digests, trial-ending checks,
   stale-data cleanup, billing reconciliation.
4. **Drip / sequence** — multi-step automations (onboarding emails —
   topic 06, recall campaigns).
5. **User notification preferences** — which channels, which types,
   which frequency.
6. **Queue + retry** — for jobs that can't run inline.

## Why it matters

Two reasons:
- **Customer comms reliability** — a missed "trial ending tomorrow"
  email costs you the customer.
- **Data eventual consistency** — the cron that retries failed billing
  charges is what keeps your DB and Stripe in sync.

## Patterns

### In-app notifications

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL,
  user_id UUID,                        -- null = tenant-wide
  kind TEXT NOT NULL,                  -- e.g. 'invoice.paid'
  title TEXT NOT NULL,
  body TEXT,
  link_url TEXT,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

A `notify(tenantId, userId, kind, payload)` helper writes to this table
and dispatches via configured channels (email, push, etc.).

### Idempotent cron

Every cron job must be idempotent. The pattern:

```ts
const lastRun = await db.cronRuns.findFirst({ where: { name: 'daily-digest' }, orderBy: { runAt: 'desc' } })
const since = lastRun?.runAt ?? new Date(Date.now() - 24*3600_000)
const events = await getEventsSince(since)
// Process — must be safe to re-run on same events
await db.cronRuns.create({ data: { name: 'daily-digest', runAt: new Date() } })
```

### Queue

For solo SaaS, defer queues until needed:
- **Inline** is fine if the work is < 1s.
- **Background tick** (a separate `node` process running every 30s) is
  fine for non-bursty work.
- **Real queue** (Inngest, Trigger.dev, BullMQ, SQS) when you need
  retries with backoff and concurrency control.

## What goes in v0.2

- Channels: email, in-app, push (Web Push), Slack/Discord webhooks.
- Notification preferences UI.
- Cron scheduling on each platform (Vercel Cron, Fly Machines, plain
  systemd timer on a VPS).
- Inngest / Trigger.dev for queue-as-a-service.
- The "must reach inbox" subset: trial-ending, past-due, security
  alerts.

## Sources

- Inngest docs: [inngest.com/docs](https://www.inngest.com/docs).
- Trigger.dev: [trigger.dev/docs](https://trigger.dev/docs).
- Linear's notification preferences UI is the canonical UX reference.
- Knock ([knock.app](https://knock.app)) is a notification-as-a-service
  worth considering at scale.
