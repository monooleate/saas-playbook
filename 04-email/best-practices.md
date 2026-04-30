# 04 — Email & Communication — Best Practices

> **Status:** Outline. Source pointers identified.

## Sources

- **Resend deliverability guide** —
  [resend.com/docs/dashboard/emails/deliverability-status](https://resend.com/docs/dashboard/emails/deliverability-status).
- **Postmark — The Rebel's Guide to Email Marketing** — free PDF.
- **DMARC.org** — [dmarc.org/overview](https://dmarc.org/overview/).
- **Google's bulk sender requirements** (Feb 2024) — required for
  anyone sending >5k/day to Gmail. SPF + DKIM + DMARC + one-click
  unsubscribe.
- **Microsoft's bulk sender requirements** (May 2025) — similar.

## Practices to expand

- Write transactional emails in plain text. HTML is fine but the
  text/plain part is what spam filters read.
- Single-purpose emails: one CTA per email.
- From-name: your product name, not "noreply." `Grabit <hello@mail.grabit.hu>`,
  not `noreply@grabit.hu`.
- Reply-To set to a monitored inbox even on auto-sends.
- Subject lines under 60 characters (mobile preview).
- Don't use URL shorteners — Gmail flags them.
- Tracking pixels degrade deliverability; turn off click tracking on
  transactional unless you must measure.
- Warm-up isn't required for small SaaS volumes (<10k/mo) on a
  reputable provider.

(Full quotes and examples in v0.2.)
