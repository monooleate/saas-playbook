# 03 — Billing & Subscription

> **When to implement:** Before customer #1 pays you. Realistically: 2 weeks
> before public launch.
> **Cost:** 5–10 days for a working integration with one provider; another
> 3–5 days for invoices, dunning, and a second provider.
> **Belongs to:** *Foundations*.

## What this topic covers

1. **Pricing model** — flat plans, per-seat, addons, usage-based.
2. **Billing provider integration** — Stripe / Lemon Squeezy / Paddle / Barion,
   abstracted behind one adapter so the app code is provider-agnostic.
3. **Subscription lifecycle** — sign-up → trial → active → past-due → cancelled.
4. **Webhooks** — verifying, idempotency, retries, replaying failed deliveries.
5. **Invoices and tax** — VAT, MoR (Merchant-of-Record), Hungarian Számlázz.hu
   integration as a regional example.
6. **Dunning** — retry logic for failed payments, customer comms.
7. **Multi-currency** — country-driven pricing with FX fallback.
8. **Plan changes** — upgrades, downgrades, addon toggles, proration.
9. **Cancellation and refund flow.**

What this topic does NOT cover (and where to look):

- *Plan → feature mapping* (entitlements). That's topic **05 — Feature
  Flags & Addon Gating**. This topic owns "what plan does the tenant have."
  Topic 05 owns "what does that plan unlock."
- *Tax law* beyond pointing at the right tools. The MoR providers handle
  this for you; if you go direct you need a tax accountant, not a playbook.
- *Refund policy text* / consumer rights wording. That's topic **15 —
  Legal & Compliance**.

---

## Why it matters

Billing is the only piece of code where every bug shows up on a credit
card statement. Two failure modes dominate:

- **Customer is charged but doesn't have access.** Webhook didn't fire,
  database write failed silently, or the entitlement table wasn't updated.
  Revenue lost to refunds + churn.
- **Customer has access but isn't charged.** Provider session expired,
  card token wasn't reused, retry logic gave up too early. Revenue silently
  not collected.

Both are caused by the same root issue: billing state lives in two places
(your DB + the provider) and they drift. The architecture below treats the
provider as the source of truth for *did the money move*, and your DB as
the source of truth for *what should the money do next*.

---

## The two billing models

You have to pick one. Going both is a bigger project than picking one well.

### A. **Direct provider** (Stripe, Paddle, Adyen)

You handle taxation. Stripe Tax helps for VAT/sales tax in major
jurisdictions, but you're still the merchant of record.

- **Pros:** lower fees (~2.9% + 30¢), full control over checkout, larger
  feature surface, dispute handling, reporting.
- **Cons:** you owe taxes. EU OSS registration, US sales tax nexus,
  marketplace facilitator rules. Requires an accountant.

### B. **Merchant-of-Record (MoR)** (Lemon Squeezy, Paddle, FastSpring)

The provider sells *to* the customer; you sell *to* the provider. Tax,
fraud, chargebacks are theirs.

- **Pros:** zero tax operations. Friendly to solo founders.
- **Cons:** higher fees (5–8% all-in). Some control loss (checkout UI,
  customer data ownership). Some providers can't do all subscription
  patterns.

**Solo founder recommendation:** start with **MoR** (Lemon Squeezy is the
cleanest API). Migrate to direct (Stripe) only when (a) the fee delta
covers an accountant, *and* (b) you want features MoR can't give you
(custom invoices, SEPA mandates, complex usage-based billing).

The Grabit project supports **three** providers behind one adapter:
- **Barion** (HU-only direct provider, Hungarian VAT 27%).
- **Paddle** (MoR, EU + global).
- **Lemon Squeezy** (MoR, EU + global, USD-billed).

This was driven by regional needs (Barion for HU low-fee billing) and
international expansion (Lemon for ease of EU/US tax). The pattern below
shows the abstraction that made this manageable for one engineer.

---

## Pricing model: prefer one JSON file as the source of truth

The Grabit project uses a single `packages/billing/pricing.json` (300+
lines) that defines **every plan and addon**, with prices, currencies,
and per-currency overrides. The app, the marketing site, and the billing
adapters all read from it.

Why this works for solo founders:
- **Marketing pages stay in sync** with the app. No "the website says
  €19, the app charges €25" bugs.
- **A pricing change is a one-file change**, then a deploy.
- **Tests are trivial:** the billing layer's tests are smoke tests that
  walk every plan and addon, multiplying out totals in every currency.

A simplified shape:

```json
{
  "currency": "USD",
  "supportedCurrencies": ["USD", "EUR", "HUF"],
  "fxRates": { "USD_to_EUR": 0.92, "USD_to_HUF": 360 },
  "plans": {
    "starter": {
      "monthlyPrice": 19,
      "monthlyPriceByCurrency": { "EUR": 18, "HUF": 6900 },
      "annualMonthlyPrice": 15,
      "trialDays": 14,
      "staffIncluded": 1
    },
    "pro": { ... }
  },
  "coreAddons": {
    "advanced_analytics": {
      "monthlyPrice": 9,
      "monthlyPriceByCurrency": { "EUR": 8, "HUF": 3200 }
    }
  },
  "extraAddons": {
    "extra_seat": {
      "price": 5,
      "priceByCurrency": { "EUR": 5, "HUF": 1900 },
      "billingType": "per_unit",
      "unitName": "seat"
    },
    "remove_branding": {
      "price": 9,
      "billingType": "monthly"
    }
  },
  "coreAddonDiscount": {
    "threshold": 3,
    "discountPercent": 20,
    "appliesTo": "core"
  }
}
```

Lookup precedence for a price in a given currency:
1. If the currency is the baseline (here `USD`) → return base price.
2. Else, if `priceByCurrency[X]` exists → use it.
3. Else, FX-convert from baseline → round to a marketing-friendly number.

The Grabit `getPriceForCurrency()` helper does exactly this. Code is in
the example files.

---

## Provider-agnostic adapter

Goal: app code never says `if (provider === 'stripe') ... else if ...`.
There is one interface; multiple implementations; a resolver that picks
the right one per tenant.

```ts
// packages/billing/types.ts
export type BillingProviderId = 'stripe' | 'lemon' | 'paddle' | 'barion'

export interface BillingAdapter {
  countryCode: string
  currency: string
  providerId: BillingProviderId

  createSubscription(params: CreateSubscriptionParams): Promise<SubscriptionResult>
  cancelSubscription(subscriptionId: string): Promise<void>
  chargeRecurring(params: ChargeRecurringParams): Promise<ChargeResult>
  getPaymentStatus(paymentId: string): Promise<PaymentStatus>
  createInvoice(params: CreateInvoiceParams): Promise<InvoiceResult>

  // Optional — providers that support these implement, others don't
  ensureCustomer?(params: EnsureCustomerParams): Promise<EnsureCustomerResult>
  syncPlanCatalog?(params: SyncCatalogParams): Promise<SyncCatalogResult>
  applyDiscountToSubscription?(params: ApplyDiscountParams): Promise<void>
  getInvoiceUrl?(params: GetInvoiceUrlParams): Promise<GetInvoiceUrlResult>
  verifyWebhookSignature?(rawBody: string, signatureHeader: string): boolean
  updateSubscriptionItems?(params: UpdateSubscriptionItemsParams): Promise<void>
}
```

The resolver decides which adapter to use:

```ts
// packages/billing/index.ts
export function getBillingAdapterForTenant(tenant: Tenant): BillingAdapter {
  // 1. Active subscription? Use the provider that owns it.
  if (tenant.stripe_subscription_id) return new StripeAdapter(tenant.country_code)
  if (tenant.lemon_subscription_id) return new LemonAdapter(tenant.country_code)
  if (tenant.paddle_subscription_id) return new PaddleAdapter(tenant.country_code)

  // 2. Tenant override (set by superadmin if needed)
  if (tenant.billing_provider) return providerFromId(tenant.billing_provider, tenant.country_code)

  // 3. Env default for new tenants
  const envDefault = process.env.BILLING_PROVIDER
  if (envDefault) return providerFromId(envDefault, tenant.country_code)

  // 4. Country fallback
  if (tenant.country_code === 'HU') return new BarionAdapter()
  return new StripeAdapter(tenant.country_code)
}
```

This is the single resolver function used everywhere — `startSubscription`,
`cancelSubscription`, `retryBilling`. Adding a fourth provider means
implementing the interface and adding a branch in this resolver. Nothing
else changes.

---

## Webhook handling: the four invariants

Whatever provider, whatever framework, every webhook handler must respect
these four:

### 1. **Verify the signature before reading the body**

Stripe, Lemon, Paddle all sign webhook bodies. An unverified webhook is
the easiest forgery in SaaS — a malicious actor sends a fake
`subscription.updated` event and bumps their plan to enterprise.

```ts
const isValid = adapter.verifyWebhookSignature(rawBody, headers['stripe-signature'])
if (!isValid) return new Response('invalid signature', { status: 401 })
```

The body must be the **raw bytes**, not a parsed JSON. Most frameworks
parse JSON automatically; you have to opt out for the webhook route.
This is gotcha [G-03](./gotchas.md#g-03-framework-auto-parses-json-and-breaks-stripe-signature-verification).

### 2. **Idempotency: the same event delivered twice must not double-count**

Providers retry on non-2xx responses. Sometimes they retry on 2xx if the
ack times out. Webhook handlers must be idempotent.

```sql
-- table: webhook_events
CREATE TABLE webhook_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider TEXT NOT NULL,
  event_id TEXT NOT NULL,           -- provider's id (evt_..., we_...)
  event_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  processed_at TIMESTAMPTZ,
  error TEXT,
  UNIQUE (provider, event_id)
);
```

Flow:
1. Receive webhook → upsert into `webhook_events` (unique constraint
   guarantees no duplicates).
2. If `processed_at IS NOT NULL` → return 200, do nothing.
3. Else, process the event (update tenant's subscription state).
4. On success, set `processed_at = NOW()`.
5. On failure, set `error = ...`, return 500 — the provider will retry.

### 3. **The webhook updates DB; the DB triggers the user-visible state**

Don't email the customer or set entitlements *in* the webhook handler.
The webhook handler updates a single source of truth (e.g. `tenant.plan`
+ `tenant.subscription_status`). Other systems (email, entitlements,
analytics) react to that DB change.

This makes replay safe and tests easier.

### 4. **Acknowledge fast, process async**

If your processing takes > 5 seconds, respond 200 immediately and queue
the work. Many providers time out at 10–30 seconds and start retrying.

For a solo SaaS at small scale, synchronous handling is usually fine if
you can keep it under 1–2 seconds. Move to async when you start to see
timeouts.

---

## Subscription lifecycle states

Standardize the state machine in your DB. Don't trust provider-specific
state names. Map provider states to your own canonical set:

| Canonical state    | Meaning                                              | Trial?  | Has access? |
|--------------------|------------------------------------------------------|---------|-------------|
| `trial`            | In free trial period                                 | yes     | yes         |
| `active`           | Paying, current period covered                       | no      | yes         |
| `past_due`         | Last charge failed, dunning in progress              | no      | yes (grace period) |
| `cancelled`        | User cancelled; access until period ends             | no      | yes (until period end) |
| `expired`          | Cancelled period has ended                           | no      | no          |
| `paused`           | Provider paused (e.g. dispute)                       | no      | no          |

Store these in a `subscriptions.status` column. Every provider webhook
event maps to a single state transition. Document the map.

---

## Multi-currency

Country-driven, never user-chooseable. The currency is decided at
sign-up by `tenant.country_code` (which the user picks once during
onboarding).

Reasons:
- **Provider constraint:** Paddle locks subscription currency at creation
  time; you cannot change it later. Same for Stripe.
- **Tax constraint:** VAT calculation depends on customer location; the
  currency must match.
- **Pricing power constraint:** charging in EUR in Czechia loses you
  customers who expect CZK. The country-code mapping (e.g. `CZ → CZK`,
  `HU → HUF`, `DE → EUR`) gives you a default that works.

Keep an explicit `COUNTRY_CURRENCY` map in code. Default to `EUR` for
unknown countries.

---

## Plan changes & proration

Three modes worth knowing:

| Mode                              | Behaviour                                                          | When to use                                |
|-----------------------------------|--------------------------------------------------------------------|--------------------------------------------|
| `prorated_immediately`            | Charge the difference now; new price applies immediately           | Upgrades; when customer wants to use more right now |
| `prorated_next_billing_period`    | New price starts next period; no immediate charge                  | Downgrades; addon toggles                   |
| `full_immediately`                | Charge full new price now, refund unused old period                | Edge cases; not commonly recommended       |
| `full_next_billing_period`        | New price starts next period; no immediate charge                  | Equivalent of "prorated_next" for many providers |
| `do_not_bill`                     | Update items, don't charge/refund                                  | Internal corrections; admin overrides       |

Stripe, Paddle, and Barion all use these or close equivalents. The
adapter exposes a `prorationMode` parameter on `updateSubscriptionItems`.

**Default for solo SaaS:** `prorated_next_billing_period`. Customers
don't get surprise charges; you don't have to explain proration math
to support tickets.

---

## Read next

1. [`checklist.md`](./checklist.md) — sprint-ready setup.
2. [`best-practices.md`](./best-practices.md) — how Stripe / Paddle /
   Lemon Squeezy actually recommend you build this, with citations.
3. [`gotchas.md`](./gotchas.md) — webhooks, signatures, double-charges,
   tax fines.
4. Examples for SvelteKit + Supabase (Barion + Paddle + Lemon),
   Next.js + Prisma (Stripe), Remix + PlanetScale (Lemon Squeezy).
