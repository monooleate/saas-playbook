# 15 — Legal & Compliance

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before first paying customer.
> **Cost:** 1–3 days for the basics; ongoing.
> **Disclaimer:** This playbook is not legal advice. Hire a lawyer for
> jurisdiction-specific guidance. The patterns here are common practice.

## Scope

What every SaaS legally needs in a typical EU/US setting:

1. **Terms of Service (ToS)**.
2. **Privacy Policy**.
3. **Cookie banner** (only if you set non-essential cookies).
4. **Data Processing Agreement (DPA)** template — for B2B customers
   who ask.
5. **GDPR data export endpoint**.
6. **GDPR account deletion endpoint**.
7. **Consent record** — when, where, what version.
8. **Subprocessor list** — public list of third-party services that
   process customer data.
9. **ToS / Privacy versioning** — old versions accessible; users
   re-consent on material changes.

## Why it matters

- **EU GDPR:** maximum fine is 4% of global revenue or €20M, whichever
  is higher. Realistic risk for solo SaaS is small but non-zero.
  Compliance is also a sales requirement — many B2B buyers ask for
  DPA + subprocessor list before signing.
- **US CCPA / state laws:** lower stakes per-incident but compounding.
- **HU specific:** NAIH (data protection authority) actively
  investigates complaints.

## What's actually feasible for a solo founder

- Use **template ToS and Privacy Policy** (Termly, GetTerms,
  Iubenda) and customize. Don't write from scratch.
- Read what you publish; understand it. Templates have known holes
  for SaaS edge cases (e.g. user-generated content).
- Have a lawyer review once before launch. Cost: €300–€1500.
- Re-review when material changes happen (new subprocessor, new
  jurisdiction, new feature like AI processing of customer data).

## What goes in v0.2

- Template ToS structure (sections to customize).
- Privacy Policy structure.
- Cookie banner: when needed, when not (Plausible / Umami don't need
  it; Google Analytics + Facebook Pixel do).
- GDPR endpoint patterns.
- DPA template (Salesforce-style, scaled for solo).
- Consent record schema.

## Sources

- Iubenda blog (best free SaaS legal content out there).
- GDPR.eu guides.
- ICO (UK) guides.
- Stripe Terms of Service (publicly available; well-written).
- Notion's privacy policy (clear, recent, reasonable).
