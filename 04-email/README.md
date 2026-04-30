# 04 — Email & Communication

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Day 1 (transactional). Marketing email later.
> **Cost:** 1–2 days for transactional; 3–5 days for templates and
> deliverability hygiene.

## Scope

This topic owns *email infrastructure* for two categories:

1. **Transactional** — sign-up confirmation, password reset, payment
   receipts, invoice delivery, password change notification, trial-
   ending, past-due. Required, must be reliable, often time-sensitive.
2. **Marketing** — onboarding drip, feature announcements, win-back
   campaigns. Optional, can be delayed.

In-app notifications (toasts, notification center) are **not** in this
topic — they live in topic **07 — Notification & Automation**.

## Why it matters

Email is the only channel where a single misconfiguration ("DKIM not
set up") silently degrades trust over weeks. Symptoms appear as Gmail
delivery to spam, Hotmail bounces, and customers reporting "I never got
the receipt." All preventable with day-1 hygiene.

## Recommended stack for solo founders

- **Provider:** Resend (cheap, fast, great DX, ~$20/mo to start).
  Postmark for max deliverability ($15/mo, slightly less DX). SendGrid
  for legacy, Mailgun if you're already there. Avoid AWS SES until you
  need pennies-per-email; the warm-up tax isn't worth it.
- **Domain:** Send from a sub-domain (`mail.example.com` or
  `email.example.com`), not the apex. Reputation is per-subdomain;
  marketing screw-ups don't poison transactional.
- **Templates:** [react-email](https://react.email) or
  [maizzle](https://maizzle.com). Stick to one. Plain HTML works for
  small-volume.
- **Plain-text fallback:** every HTML email has a text/plain part.
  Spam filters check for it.

## Deliverability essentials (DKIM/SPF/DMARC)

The three records that decide whether your email lands in inbox or
spam:

- **SPF** — TXT record at the sending domain listing servers allowed to
  send on your behalf. Resend gives you the value.
- **DKIM** — public key TXT record so receivers can verify the
  signature your provider adds. Resend gives you 3 records (`resend._domainkey`).
- **DMARC** — TXT record at `_dmarc.example.com` saying what to do
  with messages that fail SPF/DKIM. Start with `p=none` (monitor),
  upgrade to `p=quarantine` after a month of clean reports, then
  `p=reject`.

Test with [mail-tester.com](https://www.mail-tester.com) — score 9+/10
before going live. Topic 04's full version has the exact records.

## What goes here in v0.2

- Setup walkthrough for Resend + Cloudflare DNS.
- Template library: 8–10 critical transactional emails (verify, reset,
  receipt, past-due, trial-ending, invoice, invitation, weekly digest).
- Idempotent sending: same trigger twice = one email, not two.
- Suppression list management (bounces, complaints).
- The "must reach inbox" subset vs "everything else."
- Marketing email infrastructure: list segmentation, unsubscribe link
  legality (CAN-SPAM, GDPR consent), double opt-in.
- Reply-handling: parsing inbound replies into support tickets (topic
  16 territory).

## Sources to mine

- Resend docs: [resend.com/docs](https://resend.com/docs).
- Postmark "Rebel's Guide to Email": good
  deliverability primer.
- Stripe's transactional email patterns are a public-template gold
  standard — every receipt is plain, dense, useful.
