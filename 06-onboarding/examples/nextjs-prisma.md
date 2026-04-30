# 06 — Example: Next.js + Prisma

> **Status:** Outline.

Pattern: Auth.js `events.createUser` callback fires the welcome email
+ initialises onboarding state on `Tenant.onboardingState` Prisma JSON
field. Setup checklist component reads from this field, mutates via
server action.

Drip emails scheduled with [Inngest](https://inngest.com/docs/quick-start)
or Vercel Cron (`/api/cron/drip`).
