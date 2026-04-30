# 03 — Billing — Example: SvelteKit + Supabase + Multi-Provider

The Grabit reference: a single `BillingAdapter` interface with three
implementations (Barion for HU, Paddle for global, Lemon for USD-billed
EU/US). Code patterns generalized — replace tenant fields and table names
with your own, but keep the structure.

---

## 1. Package layout

```
packages/billing/
├── package.json              # name: @yourorg/billing
├── pricing.json              # source of truth for plans + addons + currencies
├── types.ts                  # BillingAdapter interface + DTOs
├── currency.ts               # COUNTRY_CURRENCY, getPriceForCurrency, FX rates
├── index.ts                  # resolver: tenant → adapter
├── stripe.ts                 # StripeAdapter (added in §10)
├── lemon.ts                  # LemonAdapter
├── paddle.ts                 # PaddleAdapter
├── barion.ts                 # BarionAdapter (HU-only)
└── szamlazz.ts               # HU invoice integration
```

---

## 2. The adapter interface

```ts
// packages/billing/types.ts
export type BillingProviderId = 'stripe' | 'lemon' | 'paddle' | 'barion'

export interface BillingAdapter {
  readonly countryCode: string
  readonly currency: string
  readonly providerId: BillingProviderId

  createSubscription(p: CreateSubscriptionParams): Promise<SubscriptionResult>
  cancelSubscription(subscriptionId: string): Promise<void>
  chargeRecurring(p: ChargeRecurringParams): Promise<ChargeResult>
  getPaymentStatus(paymentId: string): Promise<PaymentStatus>
  createInvoice(p: CreateInvoiceParams): Promise<InvoiceResult>

  // Optional capabilities — providers that don't support them simply
  // omit these methods, and the caller checks `if (adapter.method)`.
  ensureCustomer?(p: EnsureCustomerParams): Promise<EnsureCustomerResult>
  syncPlanCatalog?(p: SyncCatalogParams): Promise<SyncCatalogResult>
  applyDiscountToSubscription?(p: ApplyDiscountParams): Promise<void>
  getInvoiceUrl?(p: GetInvoiceUrlParams): Promise<GetInvoiceUrlResult>
  verifyWebhookSignature?(rawBody: string, signature: string): boolean
  updateSubscriptionItems?(p: UpdateSubscriptionItemsParams): Promise<void>
}

export interface CreateSubscriptionParams {
  tenantId: string
  email: string
  planId: string
  amount: number
  planName: string
  redirectUrl: string
  callbackUrl: string
  trialDaysLeft?: number
  breakdown?: Array<{ name: string; price: number }>
  couponCode?: string | null
}

export interface SubscriptionResult {
  paymentId: string
  redirectUrl: string
  recurrenceId: string
  transactionId?: string   // Paddle/Lemon
}

export type PaymentStatus =
  | 'pending' | 'authorized' | 'succeeded'
  | 'failed' | 'cancelled' | 'expired'

export type ProrationBillingMode =
  | 'prorated_immediately'
  | 'prorated_next_billing_period'
  | 'full_immediately'
  | 'full_next_billing_period'
  | 'do_not_bill'

export interface UpdateSubscriptionItemsParams {
  subscriptionId: string
  productId: string
  amount: number
  currency: string
  planName: string
  description?: string
  prorationMode?: ProrationBillingMode
}
```

---

## 3. The resolver

```ts
// packages/billing/index.ts
import type { BillingAdapter, BillingProviderId } from './types'
import { BarionAdapter } from './barion'
import { PaddleAdapter } from './paddle'
import { LemonAdapter } from './lemon'

interface TenantBillingFields {
  billing_provider?: string | null
  country_code?: string | null
  barion_recurrence_id?: string | null
  paddle_subscription_id?: string | null
  lemon_subscription_id?: string | null
}

/** Decide provider for a *new* signup. */
export function resolveProviderForTenant(t: TenantBillingFields): BillingProviderId {
  if (t.billing_provider === 'lemon')  return 'lemon'
  if (t.billing_provider === 'paddle') return 'paddle'
  if (t.billing_provider === 'barion') return 'barion'

  const envDefault = (process.env.BILLING_PROVIDER ?? '').toLowerCase()
  if (envDefault === 'lemon')  return 'lemon'
  if (envDefault === 'paddle') return 'paddle'
  if (envDefault === 'barion') return 'barion'

  if (t.country_code === 'HU') return 'barion'
  return 'paddle'
}

export function getBillingAdapter(provider: BillingProviderId, countryCode: string): BillingAdapter {
  if (provider === 'lemon')  return new LemonAdapter(countryCode)
  if (provider === 'paddle') return new PaddleAdapter(countryCode)
  if (provider === 'barion') {
    if (countryCode.toUpperCase() !== 'HU') {
      throw new Error(`Barion is HU-only; got ${countryCode}`)
    }
    return new BarionAdapter()
  }
  throw new Error(`Unknown billing provider: ${provider}`)
}

/** For *existing* subs, prefer the provider that owns the active subscription. */
export function getBillingAdapterForTenant(t: TenantBillingFields): BillingAdapter {
  if (t.lemon_subscription_id)  return getBillingAdapter('lemon', t.country_code ?? 'HU')
  if (t.paddle_subscription_id) return getBillingAdapter('paddle', t.country_code ?? 'HU')
  if (t.barion_recurrence_id)   return getBillingAdapter('barion', t.country_code ?? 'HU')
  return getBillingAdapter(resolveProviderForTenant(t), t.country_code ?? 'HU')
}
```

---

## 4. Currency helper

```ts
// packages/billing/currency.ts
export const COUNTRY_CURRENCY: Record<string, string> = {
  HU: 'HUF',
  DE: 'EUR', AT: 'EUR', FR: 'EUR', IT: 'EUR', NL: 'EUR', ES: 'EUR',
  RO: 'RON', PL: 'PLN', CZ: 'CZK',
  GB: 'GBP', US: 'USD', CA: 'CAD',
}

export function getCurrencyForCountry(cc: string | null | undefined): string {
  if (!cc) return 'EUR'
  return COUNTRY_CURRENCY[cc.toUpperCase()] ?? 'EUR'
}

// Reviewed: see top-of-file date comment. Update every 6 months.
export const FX_FALLBACK_RATES: Record<string, number> = {
  HUF: 1,
  EUR: 1 / 400,    // 1 HUF = 1/400 EUR
  USD: 1 / 365,
  GBP: 1 / 470,
  RON: 1 / 80,
  PLN: 1 / 95,
  CZK: 1 / 16,
}

/**
 * Look up a price in the requested currency.
 *  1. If currency is the HUF baseline → return basePrice.
 *  2. If `byCurrency[currency]` exists → use it.
 *  3. Else convert via FX_FALLBACK_RATES, round to 2dp.
 */
export function getPriceForCurrency(
  basePrice: number,
  byCurrency: Record<string, number> | undefined,
  currency: string,
): number {
  if (currency === 'HUF') return basePrice
  if (byCurrency?.[currency] != null) return byCurrency[currency]
  const rate = FX_FALLBACK_RATES[currency]
  if (rate == null) {
    throw new Error(`No FX rate or override for currency: ${currency}`)
  }
  return Math.round(basePrice * rate * 100) / 100
}

export const PROVIDER_CURRENCIES: Record<string, string[]> = {
  barion: ['HUF'],
  lemon:  ['USD', 'EUR', 'GBP'],
  paddle: ['HUF', 'EUR', 'USD', 'GBP', 'RON', 'PLN', 'CZK', 'CHF', 'DKK', 'SEK', 'NOK'],
  stripe: ['HUF', 'EUR', 'USD', 'GBP', 'RON', 'PLN', 'CZK', /* + 100 more */],
}

export function validateProviderCurrencyMatch(
  amount: number, currency: string, providerId: string,
): void {
  const allowed = PROVIDER_CURRENCIES[providerId]
  if (!allowed) throw new Error(`Unknown provider: ${providerId}`)
  if (!allowed.includes(currency)) {
    throw new Error(`${providerId} doesn't support ${currency}; allowed: ${allowed.join(', ')}`)
  }
  if (amount <= 0) throw new Error(`Amount must be positive; got ${amount}`)
}
```

---

## 5. Pricing JSON consumed by app

```ts
// apps/app/src/lib/server/addons.ts
import pricingJson from '@yourorg/billing/pricing.json'
import { getPriceForCurrency } from '@yourorg/billing/currency'

export interface AddonDefinition {
  name: string
  description: string
  price: number                                 // baseline (e.g. HUF)
  priceByCurrency?: Record<string, number>
  isPerUnit: boolean
  unitName?: string
  category: 'team' | 'marketing' | 'integrations' | 'communication' | 'core'
  addonType: 'core' | 'extra'
}

const extra = pricingJson.extraAddons as Record<string, any>
const core  = pricingJson.coreAddons  as Record<string, any>

export const ADDON_REGISTRY: Record<string, AddonDefinition> = {
  resource_pillar: {
    name: 'Resource scheduling',
    description: 'Book rooms, courts, equipment.',
    price: core.resource_pillar.monthlyPrice,
    priceByCurrency: core.resource_pillar.monthlyPriceByCurrency,
    isPerUnit: false,
    category: 'core',
    addonType: 'core',
  },
  extra_seat: {
    name: 'Additional team member',
    description: '1 seat included; each extra seat is billed monthly.',
    price: extra.extra_seat.price,
    priceByCurrency: extra.extra_seat.priceByCurrency,
    isPerUnit: true,
    unitName: 'seat',
    category: 'team',
    addonType: 'extra',
  },
  // ... rest
}

// Compute total for tenant
export function computeMonthlyTotal(
  planMonthly: number,
  activeAddons: Array<{ slug: string; quantity?: number }>,
  currency: string,
): number {
  let total = planMonthly
  for (const a of activeAddons) {
    const def = ADDON_REGISTRY[a.slug]
    if (!def) continue
    const price = getPriceForCurrency(def.price, def.priceByCurrency, currency)
    total += price * (a.quantity ?? 1)
  }
  return total
}
```

---

## 6. Database tables

```sql
-- tenants (relevant columns only)
ALTER TABLE tenants
  ADD COLUMN country_code TEXT,
  ADD COLUMN currency TEXT,
  ADD COLUMN billing_provider TEXT,
  ADD COLUMN plan TEXT NOT NULL DEFAULT 'trial',
  ADD COLUMN subscription_status TEXT NOT NULL DEFAULT 'trial',
  ADD COLUMN trial_ends_at TIMESTAMPTZ,
  ADD COLUMN barion_recurrence_id TEXT,
  ADD COLUMN paddle_customer_id TEXT,
  ADD COLUMN paddle_subscription_id TEXT,
  ADD COLUMN lemon_customer_id TEXT,
  ADD COLUMN lemon_subscription_id TEXT,
  ADD COLUMN stripe_customer_id TEXT,
  ADD COLUMN stripe_subscription_id TEXT;

-- tenant_addons
CREATE TABLE tenant_addons (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  addon_slug TEXT NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  quantity INT NOT NULL DEFAULT 1,
  config JSONB NOT NULL DEFAULT '{}'::jsonb,
  activated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deactivated_at TIMESTAMPTZ,
  UNIQUE (tenant_id, addon_slug)
);

-- subscription_periods (one row per billing cycle)
CREATE TABLE subscription_periods (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  provider TEXT NOT NULL,
  amount NUMERIC(12,2) NOT NULL,
  currency TEXT NOT NULL,
  period_start TIMESTAMPTZ NOT NULL,
  period_end TIMESTAMPTZ NOT NULL,
  charge_status TEXT NOT NULL,
  payment_id TEXT,
  invoice_id TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX subscription_periods_tenant_idx ON subscription_periods(tenant_id, period_start DESC);

-- webhook_events (idempotency)
CREATE TABLE webhook_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider TEXT NOT NULL,
  event_id TEXT NOT NULL,
  event_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  processed_at TIMESTAMPTZ,
  error TEXT,
  UNIQUE (provider, event_id)
);

-- invoices
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  provider TEXT NOT NULL,
  provider_invoice_id TEXT NOT NULL,
  invoice_number TEXT NOT NULL,
  amount NUMERIC(12,2) NOT NULL,
  currency TEXT NOT NULL,
  status TEXT NOT NULL,  -- draft, paid, void
  pdf_url TEXT,
  issued_at TIMESTAMPTZ NOT NULL,
  UNIQUE (provider, provider_invoice_id)
);
```

---

## 7. SvelteKit checkout endpoint

```ts
// apps/app/src/routes/api/billing/start-subscription/+server.ts
import { json, error } from '@sveltejs/kit'
import { getBillingAdapterForTenant } from '@yourorg/billing'
import { getCurrencyForCountry } from '@yourorg/billing/currency'
import { computeMonthlyTotal, ADDON_REGISTRY } from '$lib/server/addons'
import { supabaseAdmin } from '$lib/server/supabase'
import pricing from '@yourorg/billing/pricing.json'

export async function POST({ request, locals, url }) {
  const tenant = locals.tenant   // resolved by hook
  if (!tenant) throw error(401, 'unauthorized')

  const body = await request.json()
  const planId = body.planId as 'starter' | 'pro'

  const currency = getCurrencyForCountry(tenant.country_code)
  const plan = pricing.plans[planId]
  if (!plan) throw error(400, 'unknown plan')

  const amount = computeMonthlyTotal(
    plan.monthlyPriceByCurrency?.[currency] ?? plan.monthlyPrice,
    [],   // no addons at signup
    currency,
  )

  const adapter = getBillingAdapterForTenant(tenant)

  const result = await adapter.createSubscription({
    tenantId: tenant.id,
    email: tenant.email,
    planId,
    amount,
    planName: plan.name,
    redirectUrl: `${url.origin}/billing/success`,
    callbackUrl: `${url.origin}/api/webhooks/${adapter.providerId}`,
    trialDaysLeft: plan.trialDays,
  })

  await supabaseAdmin.from('tenants').update({
    plan: planId,
    currency,
    [`${adapter.providerId}_subscription_id`]: result.recurrenceId,
  }).eq('id', tenant.id)

  return json({ redirectUrl: result.redirectUrl })
}
```

---

## 8. Webhook handler (provider-shaped)

```ts
// apps/app/src/routes/api/webhooks/stripe/+server.ts
import { error, text } from '@sveltejs/kit'
import { supabaseAdmin } from '$lib/server/supabase'
import Stripe from 'stripe'
import * as Sentry from '@sentry/sveltekit'

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)
const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET!

export async function POST({ request }) {
  const rawBody = await request.text()       // raw, not parsed
  const sig = request.headers.get('stripe-signature') || ''

  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(rawBody, sig, WEBHOOK_SECRET)
  } catch (err) {
    return new Response(`signature error: ${err}`, { status: 401 })
  }

  // Idempotency: short-circuit duplicates
  const { data: inserted } = await supabaseAdmin
    .from('webhook_events')
    .insert({
      provider: 'stripe',
      event_id: event.id,
      event_type: event.type,
      payload: event as any,
    })
    .select('id')
    .maybeSingle()

  if (!inserted) {
    return text('duplicate', { status: 200 })
  }

  try {
    await processStripeEvent(event)
    await supabaseAdmin
      .from('webhook_events')
      .update({ processed_at: new Date().toISOString() })
      .eq('event_id', event.id)
    return text('ok', { status: 200 })
  } catch (err) {
    Sentry.captureException(err, { extra: { event } })
    await supabaseAdmin
      .from('webhook_events')
      .update({ error: String(err) })
      .eq('event_id', event.id)
    return text('processing failed', { status: 500 })
  }
}

async function processStripeEvent(event: Stripe.Event) {
  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated': {
      const sub = event.data.object as Stripe.Subscription
      const tenantId = sub.metadata?.tenantId
      if (!tenantId) throw new Error('missing tenantId in subscription metadata')

      const status = mapStripeStatus(sub.status)
      await supabaseAdmin
        .from('tenants')
        .update({
          subscription_status: status,
          stripe_subscription_id: sub.id,
        })
        .eq('id', tenantId)
      break
    }

    case 'customer.subscription.deleted': {
      const sub = event.data.object as Stripe.Subscription
      const tenantId = sub.metadata?.tenantId
      if (!tenantId) return
      await supabaseAdmin
        .from('tenants')
        .update({ subscription_status: 'expired' })
        .eq('id', tenantId)
      break
    }

    case 'invoice.payment_succeeded': {
      const inv = event.data.object as Stripe.Invoice
      // record into subscription_periods + invoices tables
      // (left as exercise; the pattern is straightforward)
      break
    }

    case 'invoice.payment_failed': {
      const inv = event.data.object as Stripe.Invoice
      const tenantId = (inv.subscription_details?.metadata as any)?.tenantId
      if (!tenantId) return
      await supabaseAdmin
        .from('tenants')
        .update({ subscription_status: 'past_due' })
        .eq('id', tenantId)
      // Trigger past-due email — see topic 04
      break
    }
  }
}

function mapStripeStatus(s: Stripe.Subscription.Status): string {
  switch (s) {
    case 'trialing': return 'trial'
    case 'active': return 'active'
    case 'past_due': return 'past_due'
    case 'canceled': return 'cancelled'
    case 'unpaid':
    case 'incomplete_expired': return 'expired'
    case 'paused': return 'paused'
    default: return 'expired'
  }
}
```

---

## 9. Addon toggle endpoint (with proration)

```ts
// apps/app/src/routes/api/billing/toggle-addon/+server.ts
import { json, error } from '@sveltejs/kit'
import { getBillingAdapterForTenant } from '@yourorg/billing'
import { getCurrencyForCountry } from '@yourorg/billing/currency'
import { ADDON_REGISTRY, computeMonthlyTotal } from '$lib/server/addons'
import { supabaseAdmin } from '$lib/server/supabase'
import pricing from '@yourorg/billing/pricing.json'

export async function POST({ request, locals }) {
  const tenant = locals.tenant
  if (!tenant) throw error(401, 'unauthorized')

  const { addonSlug, enabled, quantity } = await request.json()
  const def = ADDON_REGISTRY[addonSlug]
  if (!def) throw error(400, 'unknown addon')

  const currency = getCurrencyForCountry(tenant.country_code)

  // Toggle in DB inside a transaction-equivalent block.
  // Supabase JS doesn't have first-class TX; emulate with a single RPC if you
  // want strict atomicity. For solo SaaS, sequential calls + rollback on error:
  await supabaseAdmin.from('tenant_addons').upsert({
    tenant_id: tenant.id,
    addon_slug: addonSlug,
    is_active: enabled,
    quantity: quantity ?? 1,
    deactivated_at: enabled ? null : new Date().toISOString(),
  })

  // Recompute total, push to provider
  const { data: addons } = await supabaseAdmin
    .from('tenant_addons')
    .select('addon_slug, quantity')
    .eq('tenant_id', tenant.id)
    .eq('is_active', true)

  const plan = pricing.plans[tenant.plan as 'starter' | 'pro']
  const newTotal = computeMonthlyTotal(
    plan.monthlyPriceByCurrency?.[currency] ?? plan.monthlyPrice,
    addons ?? [],
    currency,
  )

  const adapter = getBillingAdapterForTenant(tenant)
  if (adapter.updateSubscriptionItems) {
    try {
      await adapter.updateSubscriptionItems({
        subscriptionId: tenant.stripe_subscription_id
                     ?? tenant.paddle_subscription_id
                     ?? tenant.lemon_subscription_id!,
        productId: tenant.plan,
        amount: newTotal,
        currency,
        planName: plan.name,
        prorationMode: 'prorated_next_billing_period',
      })
    } catch (err) {
      // Roll back the toggle to keep DB and provider in sync
      await supabaseAdmin.from('tenant_addons').upsert({
        tenant_id: tenant.id,
        addon_slug: addonSlug,
        is_active: !enabled,
        quantity: quantity ?? 1,
      })
      throw error(502, 'provider sync failed; addon reverted')
    }
  }

  return json({ ok: true, newMonthlyTotal: newTotal, currency })
}
```

---

## 10. Stripe adapter implementation (sketch)

```ts
// packages/billing/stripe.ts
import Stripe from 'stripe'
import type {
  BillingAdapter, BillingProviderId,
  CreateSubscriptionParams, SubscriptionResult,
  ChargeRecurringParams, ChargeResult,
  PaymentStatus, CreateInvoiceParams, InvoiceResult,
  UpdateSubscriptionItemsParams,
} from './types'

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)

export class StripeAdapter implements BillingAdapter {
  readonly providerId: BillingProviderId = 'stripe'
  readonly currency: string

  constructor(public readonly countryCode: string) {
    // Naive mapping; the full COUNTRY_CURRENCY map lives in currency.ts
    this.currency = ({ HU: 'HUF', GB: 'GBP', US: 'USD' } as Record<string, string>)[countryCode] ?? 'EUR'
  }

  async createSubscription(p: CreateSubscriptionParams): Promise<SubscriptionResult> {
    // Use Checkout Sessions for hosted UI
    const session = await stripe.checkout.sessions.create({
      mode: 'subscription',
      customer_email: p.email,
      success_url: p.redirectUrl + '?session_id={CHECKOUT_SESSION_ID}',
      cancel_url: p.redirectUrl + '?cancelled=1',
      line_items: [{
        price_data: {
          currency: this.currency.toLowerCase(),
          product_data: { name: p.planName },
          unit_amount: this.toMinor(p.amount),
          recurring: { interval: 'month' },
        },
        quantity: 1,
      }],
      subscription_data: {
        trial_period_days: p.trialDaysLeft,
        metadata: { tenantId: p.tenantId },
      },
      automatic_tax: { enabled: true },
      tax_id_collection: { enabled: true },
      metadata: { tenantId: p.tenantId },
    }, {
      idempotencyKey: `tenant-${p.tenantId}-checkout-${p.planId}`,
    })

    return {
      paymentId: session.id,
      redirectUrl: session.url!,
      recurrenceId: session.subscription as string ?? '',
      transactionId: session.id,
    }
  }

  async cancelSubscription(subscriptionId: string): Promise<void> {
    await stripe.subscriptions.update(subscriptionId, { cancel_at_period_end: true })
  }

  async chargeRecurring(_p: ChargeRecurringParams): Promise<ChargeResult> {
    // Stripe handles recurring automatically
    return { paymentId: '', status: 'noop' as any }
  }

  async getPaymentStatus(paymentId: string): Promise<PaymentStatus> {
    const session = await stripe.checkout.sessions.retrieve(paymentId)
    return session.payment_status === 'paid' ? 'succeeded' : 'pending'
  }

  async createInvoice(_p: CreateInvoiceParams): Promise<InvoiceResult> {
    // Stripe creates invoices automatically when subscriptions renew.
    // For custom one-off invoices, use stripe.invoices.create.
    throw new Error('not used: Stripe auto-invoices')
  }

  async updateSubscriptionItems(p: UpdateSubscriptionItemsParams): Promise<void> {
    const sub = await stripe.subscriptions.retrieve(p.subscriptionId)
    const item = sub.items.data[0]
    await stripe.subscriptions.update(p.subscriptionId, {
      items: [{
        id: item.id,
        price_data: {
          currency: p.currency.toLowerCase(),
          product: item.price.product as string,
          unit_amount: this.toMinor(p.amount),
          recurring: { interval: 'month' },
        },
      }],
      proration_behavior: this.mapProration(p.prorationMode ?? 'prorated_next_billing_period'),
    }, {
      idempotencyKey: `update-${p.subscriptionId}-${p.amount}-${p.currency}`,
    })
  }

  verifyWebhookSignature(rawBody: string, signature: string): boolean {
    try {
      stripe.webhooks.constructEvent(rawBody, signature, process.env.STRIPE_WEBHOOK_SECRET!)
      return true
    } catch {
      return false
    }
  }

  private toMinor(major: number): number {
    // HUF and JPY have no decimals; everything else uses 100 minor units
    if (this.currency === 'HUF' || this.currency === 'JPY') return Math.round(major)
    return Math.round(major * 100)
  }

  private mapProration(mode: string): 'create_prorations' | 'none' | 'always_invoice' {
    if (mode === 'prorated_immediately') return 'always_invoice'
    if (mode === 'prorated_next_billing_period') return 'create_prorations'
    if (mode === 'do_not_bill') return 'none'
    return 'create_prorations'
  }
}
```

---

## 11. Tests

```ts
// packages/billing/tests/currency.test.ts
import { describe, it, expect } from 'vitest'
import { getPriceForCurrency, validateProviderCurrencyMatch } from '../currency'

describe('getPriceForCurrency', () => {
  it('returns base when currency is HUF', () => {
    expect(getPriceForCurrency(3990, undefined, 'HUF')).toBe(3990)
  })
  it('returns explicit override when present', () => {
    expect(getPriceForCurrency(3990, { EUR: 10 }, 'EUR')).toBe(10)
  })
  it('FX-converts when no override', () => {
    expect(getPriceForCurrency(3990, undefined, 'EUR')).toBeCloseTo(9.98, 1)
  })
  it('throws on unknown currency', () => {
    expect(() => getPriceForCurrency(3990, undefined, 'XYZ'))
      .toThrowError(/No FX rate/)
  })
})

describe('validateProviderCurrencyMatch', () => {
  it('passes for Barion + HUF', () => {
    expect(() => validateProviderCurrencyMatch(1000, 'HUF', 'barion')).not.toThrow()
  })
  it('throws for Barion + EUR', () => {
    expect(() => validateProviderCurrencyMatch(10, 'EUR', 'barion'))
      .toThrowError(/doesn't support EUR/)
  })
})
```

---

## 12. Operations notes

- Sandbox first, always. Stripe / Lemon / Paddle / Barion all have test
  modes; never connect prod from your laptop.
- Webhook URL configured in each provider's dashboard, pointing to your
  domain's `/api/webhooks/<provider>` route. Use `ngrok` or `cloudflared`
  for local development.
- Run `stripe listen --forward-to localhost:3000/api/webhooks/stripe`
  during dev — this signs events with your test secret automatically.
- Reconciliation cron daily: list all subs in your DB, fetch latest
  status from each provider, log mismatches. If you see > 0 mismatches,
  there's a bug.
