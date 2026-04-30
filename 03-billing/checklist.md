# 03 — Billing & Subscription — Checklist

Sprint-ready. Order matters: sections later in the list depend on the
earlier ones.

## Decisions before any code

- [ ] Picked the billing model: **MoR** (Lemon, Paddle) or **direct**
      (Stripe, Adyen, Barion). See README for the trade-off.
- [ ] Picked the *first* provider. Plan to add a second only if you need
      regional reach.
- [ ] Picked the pricing structure: flat plans, per-seat, addon-driven.
      Wrote it down on a single page.
- [ ] Identified target jurisdictions for first 12 months. Confirmed your
      provider can sell to them.
- [ ] Read the provider's webhook docs in full. Bookmarked.

## Pricing as data

- [ ] Created `packages/billing/pricing.json` with **all** plans, addons,
      and per-currency overrides.
- [ ] Marketing site reads from `pricing.json` (not from a separate copy).
- [ ] Onboarding / upgrade UIs read from `pricing.json`.
- [ ] Wrote a smoke test that totals every plan + addon combination in
      every currency. Catches off-by-one and missing currency entries.

## Database schema

- [ ] `tenants.billing_provider` (nullable text — provider id, e.g.
      `stripe`).
- [ ] `tenants.country_code` (ISO 3166-1 alpha-2).
- [ ] `tenants.plan` (e.g. `trial`, `starter`, `pro`).
- [ ] `tenants.subscription_status` — canonical states (see README).
- [ ] `tenants.{stripe,lemon,paddle}_customer_id`,
      `tenants.{stripe,lemon,paddle}_subscription_id` — provider-specific
      IDs, all nullable. One of them populated per active tenant.
- [ ] `subscription_periods` table tracking each billing period (start,
      end, amount, currency, status). Insert on each successful charge.
- [ ] `webhook_events` table with `(provider, event_id)` UNIQUE for
      idempotency. See README for schema.
- [ ] `tenant_addons` table for active addons per tenant (slug + active
      flag + config jsonb + activated_at).
- [ ] `invoices` table linking to provider's invoice id and a stored PDF
      URL or bytes.

## Provider integration (per provider)

For each provider:

- [ ] Created **test mode** account/keys.
- [ ] Stored secrets: `*_API_KEY`, `*_WEBHOOK_SECRET` in the secret
      store / `.env` (production has its own copies).
- [ ] Implemented the `BillingAdapter` interface in
      `packages/billing/<provider>.ts`.
- [ ] Implemented webhook signature verification.
- [ ] Connected webhook URL in provider dashboard pointing to your
      `/api/webhooks/<provider>` endpoint.
- [ ] Tested *every* webhook event you care about by triggering it from
      the provider's test tools (Stripe CLI, Paddle's "send test event",
      Lemon's "test webhook" button).

### Stripe-specific

- [ ] Account verified (KYB). Can take 1–7 days.
- [ ] Stripe Tax enabled if you intend direct billing in EU/US.
- [ ] Customer Portal configured (cancellation, update card UI hosted by
      Stripe).
- [ ] Webhook endpoint signing secret pasted as `STRIPE_WEBHOOK_SECRET`.
- [ ] Tested with `stripe listen --forward-to localhost:3000/api/webhooks/stripe`.

### Lemon Squeezy-specific

- [ ] Store created.
- [ ] Products + variants created (one variant per plan).
- [ ] Webhook URL configured + signing secret stored.
- [ ] Test mode toggled on; placed a test order.

### Paddle-specific

- [ ] Sandbox account.
- [ ] Products + Prices created (Paddle separates them; non-catalog
      prices are common for per-tenant overrides).
- [ ] Webhook URL configured.
- [ ] Tax category set per product.

### Barion (HU only)

- [ ] Sandbox POS key.
- [ ] Tested `Payment/Start` with `InitiateRecurrence: true` to register
      the card token.
- [ ] Implemented Számlázz.hu integration for VAT-compliant invoices.

## Subscription lifecycle

- [ ] Sign-up flow: `createSubscription()` → redirect to provider's
      checkout → on return, save `subscription_id` to the tenant.
- [ ] Trial handling: `trial_ends_at` field on tenant; transition to
      `active` only after first successful charge.
- [ ] Past-due handling: webhook for failed charge → `subscription_status
      = 'past_due'`, send email, retry per provider's dunning schedule.
- [ ] Cancellation: `cancelSubscription()` → provider cancels at period
      end → tenant.status remains `active` until provider sends
      `subscription.cancelled`.
- [ ] Recovery: tenant clicks "reactivate" → re-enter checkout flow →
      new subscription.
- [ ] Plan change: `updateSubscriptionItems()` with proration mode.
      Default to `prorated_next_billing_period`.

## Webhook handler

- [ ] Raw-body middleware **disabled** for the webhook route (parse JSON
      *after* signature verification).
- [ ] Signature verified before any other processing.
- [ ] Event written to `webhook_events` table; if duplicate, return 200
      without re-processing.
- [ ] Event handler is split per event type (one function per
      `customer.subscription.updated`, etc.).
- [ ] All DB writes inside the handler are inside a transaction.
- [ ] On success: `processed_at = NOW()`, return 200.
- [ ] On failure: log, store `error`, return 500. Provider retries
      automatically.

## Idempotency on outbound calls

- [ ] Every `createSubscription` / `chargeRecurring` call passes an
      `idempotency-key` header (e.g. `tenant-${id}-${plan}-${month}`).
- [ ] If the call fails after the provider charged but before our DB
      saved the result, the next retry produces the same result, not a
      duplicate charge.

## Multi-currency

- [ ] Country → currency mapping in code (`COUNTRY_CURRENCY`).
- [ ] FX fallback rates committed in code, with a comment "review every
      6 months" and a date.
- [ ] `getPriceForCurrency(basePrice, byCurrency, currency)` helper used
      everywhere.
- [ ] Tested every supported currency in dev (smoke tests).
- [ ] Provider/currency match validated at sub creation
      (`validateProviderCurrencyMatch`) — Barion is HUF-only, Lemon is
      USD/EUR/GBP only.

## Invoices & VAT (jurisdiction-specific)

- [ ] Customer-facing invoices include: invoice number, date, period,
      amount, VAT (or "VAT included if MoR"), seller details, customer
      details.
- [ ] **EU / VAT:** capture customer VAT number at checkout. Validate
      via VIES. If VAT-valid B2B in another EU country → reverse-charge
      (zero-rate, note on invoice).
- [ ] **HU specific:** Számlázz.hu integration for NAV-compliant
      e-invoices. Push every invoice via API.
- [ ] **MoR providers (Lemon, Paddle):** they invoice the customer; you
      get a *commission/payout* statement. Skip self-invoicing.

## Dunning (failed payment recovery)

- [ ] Provider's default dunning enabled (Stripe Smart Retries, Paddle
      Recover, Lemon's Dunning).
- [ ] Past-due email day-1, day-3, day-7 (template in topic 04).
- [ ] After N retries (provider-defined or your override), transition
      to `expired` and revoke access.
- [ ] In-app banner on past-due showing "Update payment method" CTA.

## Cancellation flow

- [ ] User-initiated cancel from app: confirmation modal, "what happens
      now" copy ("you keep access until DATE").
- [ ] Save reason (free text + optional structured reason). Use it in
      monthly retention review.
- [ ] Confirmation email sent.
- [ ] Webhook from provider triggers `tenant.subscription_status =
      'cancelled'`. Access continues until period end.
- [ ] On period end webhook → `expired`, lock app routes (redirect to
      `/upgrade`).
- [ ] Reactivation path: button on `/upgrade` → new checkout.

## Refunds

- [ ] Refund button in superadmin (topic 10). Logs to `refunds` table.
- [ ] Customer-facing email when refunded.
- [ ] Refund webhook handled (`charge.refunded` etc.) — update internal
      records.

## Testing

- [ ] Unit tests for `getPriceForCurrency` over the matrix of plans ×
      currencies.
- [ ] Unit tests for the resolver (`getBillingAdapterForTenant`) — all
      branches.
- [ ] Integration test that simulates: signup → trial → first charge →
      addon toggle → cancellation → period end → expired. Use provider's
      sandbox.
- [ ] Snapshot test of the webhook event handler outputs (DB diff
      before/after).

## Operational

- [ ] Slack/Discord channel `#billing-events` posting:
      `subscription.created`, `subscription.cancelled`, `payment.failed`.
      Catches problems faster than reading logs.
- [ ] Daily cron checks for tenants in `past_due` > 14 days; alerts.
- [ ] Monthly cron computes MRR/ARR and posts to a dashboard.
- [ ] Webhook delivery monitor: graph of received events per day.
      Sudden drop = provider misconfig.

## Migration safety

- [ ] If switching providers later: dual-running plan documented.
      Existing tenants stay on old provider; new tenants on new. No
      forced migration mid-cycle.
- [ ] `tenants.billing_provider` field used to dispatch — never
      hardcoded checks.

## Pre-launch verification

- [ ] One real charge made on a real card (you, your co-founder).
      Refunded.
- [ ] One real cancellation walked through end-to-end.
- [ ] One real failed charge tested (provider's test card for declines).
- [ ] One downgrade and one upgrade tested.
- [ ] Customer Portal (or your custom portal) shows correct plan, next
      bill, payment method.
- [ ] Tax invoice for one paid customer reviewed by an accountant or
      lawyer.
