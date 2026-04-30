# 04 — Email — Example: Next.js + Prisma + Resend

> **Status:** Outline. Full implementation forthcoming.

The canonical Next.js shape uses
[react-email](https://react.email) for templates and Resend for sending:

- `apps/web/emails/*.tsx` files are React components rendered by
  react-email's `render()` to HTML + plain-text.
- `lib/email.ts` wraps `resend.emails.send()` with idempotency and
  Prisma-backed `EmailLog` row.
- `app/api/webhooks/resend/route.ts` updates log rows on delivery
  status events.

References: [react.email](https://react.email),
[resend.com/docs/send-with-nextjs](https://resend.com/docs/send-with-nextjs).
