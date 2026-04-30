# 03 — Billing & Subscription — Gotchas

Real production failures with billing impact. Many are from the Grabit
project's billing test suite, where each test name is a previous incident.

## G-01 Webhook fires before the redirect; the user sees "no plan"

**Symptom.** Customer completes checkout, returns to `/billing/success`,
the page shows "Your plan: Free" because the database still says they're
on trial. Refresh in 5 seconds shows the new plan.

**Root cause.** The webhook (`subscription.created`) and the redirect
both fire after checkout completes — but their order is not guaranteed.
On a slow network, the redirect arrives first.

**Fix.** Don't read the plan on the success page from your DB. Either:
1. Show a generic "All set! Activating..." message and poll the DB until
   the webhook has been processed (5–10 seconds of polling).
2. Pass the `subscription_id` in the success URL; on landing, call the
   provider's API once to fetch the latest state and update the DB
   synchronously.

The Grabit success page polls every 1s for up to 15s. If after 15s the
webhook hasn't landed, show "Your subscription is being activated; you'll
get an email shortly."

## G-02 Stripe webhook 200s but the row didn't update

**Symptom.** Stripe's "Events" tab shows `200 OK`. Your DB shows the
tenant still on `trial`. Webhook handler logs show no error.

**Root cause.** The handler caught all exceptions and returned 200
unconditionally. The actual DB write failed (e.g. unique constraint
violation, transaction conflict) but was swallowed.

**Fix.** Return 500 on internal errors. Stripe will retry. The pattern:

```ts
export async function POST(req: Request) {
  const rawBody = await req.text()
  const event = await verifyAndParse(rawBody, req.headers)
  if (!event) return new Response('bad signature', { status: 401 })

  // Idempotency record
  const inserted = await db.webhookEvents.insert({
    provider: 'stripe', event_id: event.id, event_type: event.type, payload: event,
  }).onConflictDoNothing().returning('*')
  if (inserted.length === 0) return new Response('duplicate', { status: 200 })

  try {
    await processEvent(event)   // <-- can throw
    await db.webhookEvents.update({ processed_at: new Date() }).where({ event_id: event.id })
    return new Response('ok', { status: 200 })
  } catch (err) {
    await db.webhookEvents.update({ error: String(err) }).where({ event_id: event.id })
    Sentry.captureException(err)
    return new Response('processing failed', { status: 500 })   // <-- crucial
  }
}
```

500 = Stripe retries. 200 = Stripe stops. Picking the right one is
half the job.

## G-03 Framework auto-parses JSON and breaks Stripe signature verification

**Symptom.** `Webhook signature verification failed: No signatures
found matching the expected signature`.

**Root cause.** Stripe signs the **raw body bytes**. SvelteKit /
Express / Fastify parse JSON by default; by the time your handler runs,
`req.body` is an object, not the raw string. Stringifying it produces
different whitespace and signing fails.

**Fix.** For the webhook route specifically, read the raw body before
any parsing.

```ts
// SvelteKit
export async function POST({ request }: RequestEvent) {
  const rawBody = await request.text()  // raw, before parsing
  const sig = request.headers.get('stripe-signature') || ''
  const event = stripe.webhooks.constructEvent(rawBody, sig, process.env.STRIPE_WEBHOOK_SECRET!)
  // ...
}
```

```ts
// Next.js App Router
export const dynamic = 'force-dynamic'
export async function POST(req: NextRequest) {
  const rawBody = await req.text()
  // ...
}
```

```ts
// Express
app.post('/api/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature']
  const event = stripe.webhooks.constructEvent(req.body, sig, secret)
  // ...
})
```

## G-04 Webhook delivered twice; subscription created twice

**Symptom.** Customer charged once, but DB shows two `subscription`
rows. Or: addon "extra_seat" toggled on once but billed twice.

**Root cause.** Provider retried the webhook (your endpoint timed out
on first attempt). Handler didn't check for duplicate event id.

**Fix.** Idempotency table with UNIQUE on `(provider, event_id)`. See
README and G-02 above. Always check.

## G-05 Customer charged for an addon they didn't enable

**Symptom.** Customer support ticket: "I never turned on the SMS
reminder addon, but my invoice has it." The audit log confirms: someone
*did* toggle it on, then off, but the addon remained billed for the
full month.

**Root cause.** Toggling an addon updated the local `tenant_addons`
table immediately, but only called `provider.updateSubscriptionItems`
on save. The intermediate states leaked. *Or:* the addon was toggled
off but the provider call failed and there was no retry.

**Fix.**
1. Toggling addons in the UI is *committed* on save, not on toggle.
2. The save calls `provider.updateSubscriptionItems` *and* the local DB
   write inside one transaction (or with proper rollback semantics).
3. If the provider call fails, the local change is reverted and the user
   sees an error. No partial state.
4. Default proration = `prorated_next_billing_period`. So even if a
   wrong toggle goes through, the customer doesn't see an immediate
   charge.

## G-06 Past-due tenant kept full access for weeks

**Symptom.** A tenant had `subscription_status = 'past_due'` for 6
weeks. They kept using the product. Provider's dunning gave up after
14 days but never told us.

**Root cause.** The webhook `customer.subscription.deleted` (Stripe's
end-of-life event after dunning failure) was unhandled. Status stayed
`past_due` forever.

**Fix.** Handle the full set of subscription events. Minimum:
- `customer.subscription.created` → set status `active` or `trial`.
- `customer.subscription.updated` → update status from event payload.
- `customer.subscription.deleted` → set status `expired`, revoke access.
- `invoice.payment_failed` → set status `past_due`, log retry attempt.
- `invoice.payment_succeeded` → set status `active` (recover from
  past-due).

Backstop: a daily cron that lists `past_due` tenants older than N days,
calls `provider.subscriptions.retrieve(id)` for each, and reconciles.

## G-07 Test mode webhook secret leaked into prod

**Symptom.** Webhook handler returns 401 (signature mismatch) for every
event in production. Test mode was working fine.

**Root cause.** Each provider has **separate signing secrets** for test
and live mode. The deploy used the test secret in prod.

**Fix.**
- Name secrets explicitly: `STRIPE_WEBHOOK_SECRET_LIVE` vs
  `STRIPE_WEBHOOK_SECRET_TEST`.
- The app uses one canonical name (`STRIPE_WEBHOOK_SECRET`); the live
  env populates from `_LIVE`, the test env from `_TEST`. Naming saves
  you.
- Verify on first deploy: trigger a test event from the provider's
  dashboard and check it's accepted (200, not 401).

## G-08 EU customer charged VAT incorrectly because country wasn't validated

**Symptom.** German customer with valid German VAT ID was charged 27%
VAT (your home country) instead of 0% reverse-charge.

**Root cause.** App captured the VAT ID but never validated it against
[VIES](https://ec.europa.eu/taxation_customs/vies/). Treated all VAT IDs
as valid; applied home-country VAT.

**Fix.**
- At checkout, validate VAT ID via VIES *before* completing the sale.
  VIES returns valid/invalid + canonical name+address.
- If valid + B2B + cross-EU border → reverse-charge (zero VAT, line on
  invoice "Reverse charge per Art. 196 of Directive 2006/112/EC").
- If invalid or missing → home country VAT or destination country VAT
  per OSS rules.

Or just use a MoR provider and skip this entirely.

## G-09 Annual customer downgraded; refund expectation mismatch

**Symptom.** Annual customer paid €240 in January, downgrades to
monthly in March, expects ~€180 refund. You don't refund. Customer
churns + posts on Twitter.

**Root cause.** Your terms didn't address mid-cycle downgrade for
annual plans, and the system silently kept them on annual until renewal.

**Fix.**
- **Up front:** decide and document the refund policy for annual
  downgrades. Two viable choices: (a) no refund, downgrade applies at
  renewal; (b) prorated refund of unused months.
- The downgrade UI states the policy explicitly: "Your annual plan
  continues until [date]; you'll be on the lower plan from [date+1d]."
- Or, refund and prorate immediately if you can afford the goodwill.

## G-10 PlanetScale connection limit exhausted by webhook bursts

**Symptom.** During a Stripe replay (provider redelivered ~500 events
in 2 minutes), the app's MySQL connection pool exhausted. Returned 502s.
PlanetScale dashboard shows `Too many connections`.

**Root cause.** Each webhook handler created a new connection without
pooling. PlanetScale's free tier caps at 1000 concurrent connections.

**Fix.**
- Use a connection pooler (PgBouncer for Postgres, ProxySQL for MySQL).
  Neon and Supabase have built-in pooling URLs.
- For PlanetScale: prefer their pre-pooled HTTP-based driver
  (`@planetscale/database`) over raw `mysql2`.
- Webhook endpoint: serial processing via a single worker behind a queue
  if bursts are common.

## G-11 Currency mismatch: subscription in EUR, addon priced in HUF

**Symptom.** Tenant in Czechia signed up with EUR pricing. Months later,
toggled an addon. The system pulled the price from the HUF baseline
(3,200 HUF) and called `updateSubscriptionItems` with `amount = 3200,
currency = 'EUR'`. Provider charged €3,200.

**Root cause.** `getPriceForCurrency` was bypassed in the addon flow.
The HUF baseline was sent without conversion.

**Fix.**
- Always go through `getPriceForCurrency(basePrice, byCurrency,
  tenant.currency)`. Make it the single read path.
- Add a runtime assert: `validateProviderCurrencyMatch(amount, currency,
  provider)` rejects calls where the currency isn't supported by the
  provider (Barion: only HUF; Lemon: USD/EUR/GBP).
- Test: every addon × every supported currency, compute price, sanity-
  check the magnitude (e.g. between $1 and $1000 for typical SaaS).

## G-12 The "delete account" button cancelled the sub but didn't refund

**Symptom.** Customer used "Delete my account" two days into a fresh
month. Account deleted, sub cancelled at period end. Customer expected
prorated refund.

**Root cause.** "Cancel" and "Delete" were the same flow internally —
both called `cancelAtPeriodEnd: true` on the provider. No refund logic.

**Fix.**
- Decide whether "delete account" is a refund-bearing action. (For
  consumer SaaS in EU under 14-day cooling off, a fresh month → maybe.)
- In the UI, separate "cancel subscription" (no refund, access until
  period end) from "delete account" (refund + immediate revoke if
  desired).
- Document the policy in the ToS (topic 15) and reference it in the
  dialog.

## G-13 Stripe Tax disabled mid-flow

**Symptom.** Stripe Tax was enabled on the account but `subscription.create`
calls didn't include `automatic_tax: { enabled: true }`. Subscriptions
were created without tax. Found out 6 weeks later when reconciling.

**Root cause.** Account-level Stripe Tax doesn't apply automatically;
each API call must opt in.

**Fix.** Pass `automatic_tax: { enabled: true }` on every
`subscriptions.create` and `checkout.sessions.create`. Add a code
comment explaining why. Add a test that asserts the parameter is
present.

## G-14 Provider catalog drift: pricing.json says €19, Paddle still says €15

**Symptom.** Marketing site updated to €19. Customer signed up via
Paddle Checkout, got charged €15. Discrepancy noticed when reconciling
revenue.

**Root cause.** `pricing.json` is the source of truth in code, but
provider-side product/price IDs are configured separately. They drifted.

**Fix.**
- A `syncPlanCatalog` adapter method (Paddle, Stripe, Lemon all support
  this) that pushes the canonical prices from `pricing.json` to the
  provider on deploy.
- The Grabit deploy workflow does this: post-deploy, hits an internal
  endpoint that runs `syncPlanCatalog` and reports drift via Slack.
- Tests assert that `pricing.json` shape matches the adapter's expected
  inputs.

## G-15 Refund button refunded the wrong charge

**Symptom.** Support clicked refund on an invoice. The provider's API
refunded the *most recent* charge on the customer, which was a
different invoice (a re-bill after a card update).

**Root cause.** The refund endpoint took only the customer ID, not the
charge ID. It refunded "latest charge."

**Fix.** Always refund by `charge_id` (Stripe) or `transaction_id`
(others), not by customer. Display the specific charge in the UI before
the refund button.

## G-16 Trial ends, no email, customer caught off-guard

**Symptom.** Customer's trial ended Sunday, card was charged €19.
Customer contacted support Monday: "I never agreed to a paid plan."
Refund granted; trust damaged.

**Root cause.** The "trial ending in 3 days" email was scheduled
correctly, but the cron that sent it had a silent failure for 2 weeks
(no error, no retry, no alert).

**Fix.**
- Critical comms (trial ending, payment failed, subscription cancelled)
  are sent **synchronously** as part of the state transition, not via a
  cron. The webhook handler queues the email; the email sender is
  monitored.
- Backstop: a daily cron that lists tenants whose trial ends in 3 days
  AND no `trial_ending_email_sent_at` and sends them. If the primary
  path missed, the cron catches it.
- Topic 04 covers the email infra and topic 13 the alerting.
