# 04 — Email & Communication — Gotchas

> **Status:** Outline. Categories listed; full incident write-ups
> forthcoming.

## Categories

1. **DKIM key rotation forgotten** — provider rotates keys, DNS not
   updated, all email starts going to spam. Fix: monitor DKIM with
   automated check.
2. **DMARC `p=reject` set too early** — legitimate forwarded mail
   (mailing lists, vacation auto-forwards) starts being rejected.
   Customers stop getting receipts. Fix: ramp through `p=none` →
   `p=quarantine` over weeks, watch reports.
3. **Sub-domain reputation poisoning** — sending marketing from same
   sub-domain as transactional, then a campaign hits the spam button
   100x → transactional starts going to spam too.
4. **Webhook for bounce events not processed** — bouncing addresses
   continue to receive sends, ratio degrades, IP reputation tanks.
5. **One email per cron tick** — cron job intended to send weekly
   digest fires twice within the same minute due to a deploy +
   restart. Customers get two digests. Fix: idempotency key on send.
6. **Using `apex@example.com` as reply-to** — replies to confirmation
   emails go nowhere. Customer support invisible.
7. **Plain-text part missing** — Gmail and Outlook deliver to spam at
   higher rate.
8. **Embedded images vs hosted images** — embedded inflates message
   size; hosted images on a slow CDN delay rendering.

(Each will get a real-incident write-up in v0.2.)
