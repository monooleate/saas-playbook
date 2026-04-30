# 04 — Email & Communication — Checklist

> **Status:** Outline. Full per-item content forthcoming in v0.2.

## Domain & DNS

- Sub-domain dedicated to email (`mail.example.com`).
- SPF record published.
- DKIM records (3 from Resend / 1 from Postmark) published.
- DMARC record at `_dmarc.example.com`, starting `p=none`.
- Reverse DNS set on the sending IP (provider does this if you use
  theirs).
- Test with mail-tester.com → 9+/10.

## Transactional templates (must-haves)

- Sign-up email verification.
- Password reset.
- Magic link (if used).
- Payment receipt.
- Invoice delivery.
- Past-due / dunning.
- Trial-ending in 3 days.
- Trial-ending tomorrow.
- Subscription cancelled confirmation.
- Tenant invitation.

## Sending hygiene

- Same trigger twice = one email (idempotency in app code).
- Email log table records every send with delivery status from webhook.
- Suppression list synced from provider's bounces/complaints webhook.
- No-reply addresses are themselves monitored (auto-reply explaining
  where to actually email).

## Marketing email

- Double opt-in for any non-transactional list.
- Unsubscribe link in every marketing email (one-click,
  list-unsubscribe header).
- Audit consent: when, where, by whom for every recipient.

(Each section will be expanded with verification steps in v0.2.)
