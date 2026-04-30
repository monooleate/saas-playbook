# 03 — Billing & Subscription — Best Practices

What Stripe / Lemon Squeezy / Paddle and the SaaS ops community publicly
recommend, with sources. Solo-founder annotations where they apply.

## 1. The provider is the source of truth for "did money move," your DB
   is the source of truth for "what should the customer have"

Stripe documents this exact pattern in their [Build a subscriptions
integration guide](https://stripe.com/docs/billing/subscriptions/build-subscriptions):

> "The Stripe API is the canonical record of the customer's subscription.
> Your database stores customer state for fast access; Stripe webhooks
> keep them in sync."

Practical rule: **never call `stripe.subscriptions.list()` on every
request to check status.** That's API spam and slow. Webhooks update your
DB; reads come from the DB.

## 2. Idempotency keys on every mutation

Every Stripe POST that creates or modifies a resource accepts an
`Idempotency-Key` header. Same key = same response, even if the request
is retried. Source: [Stripe — Idempotent Requests](https://stripe.com/docs/api/idempotent_requests).

Pattern:

```ts
await stripe.subscriptions.create(params, {
  idempotencyKey: `tenant-${tenantId}-create-${planId}-${ymd()}`,
})
```

Lemon Squeezy and Paddle have the same concept under different headers
(`X-Idempotency-Key` for Lemon). Use them everywhere. Cost of forgetting:
duplicate charges when a request retries on network blip.

## 3. Webhook signatures are non-optional

Every major provider signs webhook payloads. Verifying that signature is
the only thing standing between you and an attacker forging
`subscription.updated` events.

- **Stripe:** `Stripe-Signature` header, HMAC-SHA256, verified via
  `stripe.webhooks.constructEvent(rawBody, sig, secret)`. Source:
  [Stripe Webhook Verification](https://stripe.com/docs/webhooks#verify-events).
- **Lemon Squeezy:** `X-Signature` header, HMAC-SHA256. Source:
  [Lemon Squeezy webhook signing](https://docs.lemonsqueezy.com/help/webhooks#signing-requests).
- **Paddle:** `Paddle-Signature` header. Source:
  [Paddle Webhook Signature](https://developer.paddle.com/webhooks/signature-verification).
- **Barion:** **does not sign webhooks**. You must call back to Barion's
  API to verify each event. The Grabit codebase does this in
  `packages/billing/hu.ts`.

## 4. Use the Customer Portal (or build the equivalent)

Stripe's [Customer Portal](https://stripe.com/docs/billing/subscriptions/customer-portal)
gives customers a hosted page to update card, change plan, cancel,
download invoices. It's free; you configure it once.

For solo founders this is **a strict win**: you don't build the
"update payment method" UI, you don't handle the PCI scope of card
input.

If you're not on Stripe, check whether your provider has the equivalent:
- Lemon Squeezy has [Customer Portal](https://docs.lemonsqueezy.com/help/customers/customer-portal).
- Paddle has the [retain](https://developer.paddle.com/build/subscriptions/customer-portal).
- Barion: no portal; build your own (the Grabit project does).

## 5. Trial without card vs trial with card

- **Trial without card** (most consumer SaaS): higher signup conversion,
  lower trial-to-paid conversion. Friction at the top, friction at the
  bottom.
- **Trial with card** (most B2B): lower signup conversion, higher
  trial-to-paid. Friction up front, smooth conversion.

Linear, Notion, Figma are no-card-trial. Stripe Atlas, AWS, and most
B2B tools are card-on-file.

Source: [Lenny's Newsletter — How to design a free trial](https://www.lennysnewsletter.com/p/how-to-design-a-free-trial)
covers the trade-off with industry data.

**Solo founder default:** trial-without-card if your product is
self-serve consumer; trial-with-card if it's B2B requiring setup. Either
way, ask for the card *before* the trial-end notification, not at the
last minute.

## 6. Dunning: don't lose customers to a dead card

Stripe's data ([Stripe Smart Retries](https://stripe.com/blog/smart-retries))
shows that ~13% of recurring charges fail and 70% of those can be
recovered with smart retry timing.

The recovery sequence Stripe recommends:
- Day 0 (first failure): retry immediately, then in 1 day.
- Day 3: retry; email customer with self-service link.
- Day 7: retry; second email.
- Day 14: final retry; revoke access.

Lemon Squeezy and Paddle do similar dunning automatically.

**For Barion / direct providers without dunning:** implement this
yourself with a daily cron. Pattern in the example file.

## 7. Don't store card data; don't even pass it through your servers

PCI-DSS scope balloons the moment a card number touches your server.
Use the provider's hosted checkout (Stripe Checkout, Stripe Elements,
Lemon Squeezy hosted checkout, Paddle Checkout) so the card never
reaches you.

Source: [PCI Security Standards — SAQ-A](https://www.pcisecuritystandards.org/document_library/)
defines the lightest-scope SAQ for merchants who outsource card capture.
You want this scope.

## 8. Tax: pick "MoR" or "Stripe Tax + accountant"

Halfway is the worst place. Either:
- The provider is the merchant of record (Lemon, Paddle, FastSpring) →
  taxes are theirs. Higher fees, simpler life.
- Or you are MoR + use [Stripe Tax](https://stripe.com/docs/tax) for
  calculation, file your own returns. Lower fees, accountant required.

Source: [Stripe Tax — overview](https://stripe.com/docs/tax) covers what
Stripe automates and what you still owe.

For HU founders: VAT 27% direct billing requires you to file VAT returns
monthly via NAV. The Számlázz.hu API is the standard tool — Grabit uses
it from `packages/billing/szamlazz.ts`.

## 9. Plan changes: prefer the next-period proration model

Most subscription providers offer "prorate immediately" (charge the
delta now) or "prorate next period" (apply at next billing). Stripe
default is prorate-immediately; Paddle exposes both.

[Patrick McKenzie's "Salesy Sales Tax" article](https://www.kalzumeus.com/2010/10/27/sales-tax-and-saas/)
notes the customer-experience cost of surprise mid-cycle charges. They
trigger support tickets disproportionate to the dollar amount.

**Default:** `prorated_next_billing_period`. Customers get exactly what
they expect: "this month I pay X, next month I pay Y." Easy to support.

Exception: when customer is upgrading and explicitly wants more capacity
*now*. UI copy should say "Pay $X now to upgrade immediately." Then
prorate-immediately is correct.

## 10. Currency is per-tenant, set at signup, immutable

Stripe and Paddle both lock subscription currency at creation. You cannot
change `subscription.currency` later. Source: [Stripe Subscription
Currency](https://stripe.com/docs/billing/subscriptions/multiple-currencies)
notes this explicitly.

So: pick the right currency at signup, store it on the tenant, and don't
expose a "change currency" feature. If a tenant moves countries, they
create a new subscription (with help from support).

## 11. The single-page checkout still wins

Lemon Squeezy's [research](https://www.lemonsqueezy.com/blog/best-practices-for-saas-checkout)
and Stripe's own [checkout funnel
data](https://stripe.com/blog/payment-form-best-practices) both say:

- One page, one form.
- Address autocomplete (Stripe Address Element does this).
- Apple Pay / Google Pay buttons above the form for mobile.
- No required fields beyond email + card.

For solo founders, **just use the hosted checkout** — Stripe Checkout,
Lemon Squeezy Checkout. They've A/B-tested this for you.

## 12. Webhooks fail; replay tooling is required

Even with retries, webhooks can be lost (server down for an hour, IP
change). Your provider's dashboard has an **Events** view where you can
manually retry/replay. Make sure your handler is idempotent (point 2)
so replays are safe.

Source: [Stripe — Receiving events](https://stripe.com/docs/webhooks#testing)
documents the replay mechanism.

For solo SaaS at small scale, a *daily reconciliation cron* that
compares your DB's `tenants.subscription_status` against the provider's
API state and flags mismatches is enough. ~50 lines of code, saves you
when a webhook silently fails.

## 13. MRR / ARR: compute from periods, not subscriptions

Naive MRR = sum of active subscription monthly amounts. Wrong: you
miss annual subs (need to divide by 12), addons (separate line items),
custom prices (overrides), proration deltas.

Correct: store every billing period in `subscription_periods` with
amount + start + end. MRR for month M = sum of (amount × days_in_M /
period_days) over all periods overlapping M.

Source: [ChartMogul — MRR calculation](https://chartmogul.com/help/getting-started/mrr-calculation/)
covers the canonical formula. Stripe's [MRR docs](https://stripe.com/docs/revenue-recognition)
do too.

For solo founders: ChartMogul's free tier (or Baremetrics) plugs into
your provider and computes MRR for you. Skip building this until you
need custom slicing.

## 14. Make refunds easy internally; refund tickets are 80% of support

The faster you can refund, the less time refunds take. Build the refund
button into superadmin (topic 10). Store the reason. Don't make it a
multi-step approval — for solo founder, you ARE the approval.

Source: implicit; the data is in any support metrics dashboard.

## 15. Public pricing. No "Contact us" before you have enterprise customers

Public pricing is a sales accelerator at small scale. "Contact us"
loses 60–80% of self-serve buyers (industry rule of thumb; not formally
sourced).

Stripe, Linear, Vercel, Notion — all publish per-tier pricing. They
add "Enterprise: Contact us" only as the top tier where the value is
genuinely custom.

**Solo founder rule:** publish prices unless your customer-segment
research says otherwise. The friction reduction is worth more than the
"competitor sees our prices" risk.
