# 04 — Email — Example: SvelteKit + Supabase + Resend

> **Status:** Outline. Full implementation forthcoming.

## What will go here

The Grabit project sends all transactional email via Resend. The
pattern:

1. A `packages/email` workspace with template components (one per
   email kind).
2. A `sendEmail({ to, template, props })` function wrapping the Resend
   SDK with: per-tenant from-address (default + custom email domain
   addon — see topic 05), idempotency key, and an `email_log` row
   recorded in Supabase.
3. Resend webhooks for `delivered` / `bounced` / `complained` updating
   the email log row.
4. Per-tenant template overrides (some addons add custom branding).

References:
- Resend SvelteKit guide: [resend.com/docs/send-with-sveltekit](https://resend.com/docs/send-with-sveltekit).
- Supabase for storing email log + suppression list.
