# 03 — Billing — Example: Remix + PlanetScale + Lemon Squeezy

The MoR shape. Lemon Squeezy is the merchant of record; you don't worry
about VAT, sales tax, fraud or chargebacks. Higher fees (~5% all-in) are
the trade-off.

---

## 1. Stack

| Layer            | Choice                                  |
|------------------|-----------------------------------------|
| Framework        | Remix (`@remix-run/node`)               |
| ORM              | Drizzle ORM                             |
| DB               | PlanetScale (MySQL)                     |
| Provider         | Lemon Squeezy                           |
| Hosted checkout  | Lemon Squeezy hosted checkout (overlay) |

---

## 2. Lemon Squeezy concepts

- **Store** — your account.
- **Product** — a SaaS product (one per app, usually).
- **Variant** — a specific plan/price (e.g. "Pro Monthly", "Pro Annual").
  This is what you reference in checkout.
- **Order / Subscription** — created on customer purchase.
- **Customer** — an email + address.

Configure once in Lemon Squeezy dashboard:
1. Create a Store.
2. Create one Product per offering tier.
3. Each Product gets one or more Variants (monthly, annual, etc.).
4. Note the variant IDs — you'll reference them from code.

---

## 3. Drizzle schema

```ts
// apps/web/drizzle/schema.ts
import { mysqlTable, varchar, int, datetime, decimal, json, boolean, uniqueIndex } from 'drizzle-orm/mysql-core'

export const tenants = mysqlTable('tenants', {
  id: varchar('id', { length: 26 }).primaryKey(),
  email: varchar('email', { length: 255 }).notNull(),
  countryCode: varchar('country_code', { length: 2 }),
  currency: varchar('currency', { length: 3 }),
  plan: varchar('plan', { length: 32 }).notNull().default('trial'),
  subscriptionStatus: varchar('subscription_status', { length: 32 }).notNull().default('trial'),
  trialEndsAt: datetime('trial_ends_at'),
  lemonCustomerId: varchar('lemon_customer_id', { length: 64 }),
  lemonSubscriptionId: varchar('lemon_subscription_id', { length: 64 }),
}, (t) => ({
  emailIdx: uniqueIndex('tenants_email_idx').on(t.email),
}))

export const webhookEvents = mysqlTable('webhook_events', {
  id: varchar('id', { length: 26 }).primaryKey(),
  provider: varchar('provider', { length: 32 }).notNull(),
  eventId: varchar('event_id', { length: 128 }).notNull(),
  eventType: varchar('event_type', { length: 64 }).notNull(),
  payload: json('payload').notNull(),
  receivedAt: datetime('received_at').notNull(),
  processedAt: datetime('processed_at'),
  error: varchar('error', { length: 500 }),
}, (t) => ({
  uniq: uniqueIndex('webhook_events_uniq').on(t.provider, t.eventId),
}))
```

PlanetScale doesn't enforce foreign keys — application-level checks only.

---

## 4. Lemon Squeezy adapter

```ts
// apps/web/app/lib/billing/lemon.ts
import 'server-only'

const API_BASE = 'https://api.lemonsqueezy.com/v1'
const TOKEN = process.env.LEMON_API_KEY!
const STORE_ID = process.env.LEMON_STORE_ID!

interface LemonAPIResponse<T> { data: T; included?: any[] }

async function lemon<T>(path: string, init?: RequestInit): Promise<LemonAPIResponse<T>> {
  const res = await fetch(`${API_BASE}${path}`, {
    ...init,
    headers: {
      Accept: 'application/vnd.api+json',
      'Content-Type': 'application/vnd.api+json',
      Authorization: `Bearer ${TOKEN}`,
      ...(init?.headers ?? {}),
    },
  })
  if (!res.ok) throw new Error(`Lemon ${res.status}: ${await res.text()}`)
  return res.json()
}

export async function createCheckout(args: {
  variantId: string
  email: string
  tenantId: string
  successUrl: string
}): Promise<{ url: string; checkoutId: string }> {
  const body = {
    data: {
      type: 'checkouts',
      attributes: {
        checkout_options: { embed: false, dark: false },
        checkout_data: {
          email: args.email,
          custom: { tenant_id: args.tenantId },
        },
        product_options: {
          redirect_url: args.successUrl,
        },
      },
      relationships: {
        store:   { data: { type: 'stores',          id: STORE_ID } },
        variant: { data: { type: 'variants',        id: args.variantId } },
      },
    },
  }
  const res = await lemon<any>('/checkouts', { method: 'POST', body: JSON.stringify(body) })
  return { url: res.data.attributes.url, checkoutId: res.data.id }
}

export async function getSubscription(subscriptionId: string) {
  const res = await lemon<any>(`/subscriptions/${subscriptionId}`)
  return res.data
}

export async function cancelSubscription(subscriptionId: string) {
  await lemon(`/subscriptions/${subscriptionId}`, { method: 'DELETE' })
}
```

---

## 5. Webhook signature verification

Lemon signs the raw request body with HMAC-SHA256 using your webhook
signing secret.

```ts
// apps/web/app/lib/billing/lemon-signature.ts
import crypto from 'node:crypto'

export function verifyLemonSignature(rawBody: string, signature: string): boolean {
  const secret = process.env.LEMON_WEBHOOK_SECRET!
  const expected = crypto
    .createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex')

  // Use timing-safe comparison
  const a = Buffer.from(expected, 'hex')
  const b = Buffer.from(signature, 'hex')
  if (a.length !== b.length) return false
  return crypto.timingSafeEqual(a, b)
}
```

Lemon's docs:
[https://docs.lemonsqueezy.com/help/webhooks#signing-requests](https://docs.lemonsqueezy.com/help/webhooks#signing-requests).

---

## 6. Webhook handler

```ts
// apps/web/app/routes/api.webhooks.lemon.ts
import type { ActionFunctionArgs } from '@remix-run/node'
import { db } from '@yourorg/db'
import { webhookEvents, tenants } from '~/drizzle/schema'
import { eq, and } from 'drizzle-orm'
import { verifyLemonSignature } from '~/lib/billing/lemon-signature'
import { ulid } from 'ulid'
import * as Sentry from '@sentry/remix'

export async function action({ request }: ActionFunctionArgs) {
  const rawBody = await request.text()
  const signature = request.headers.get('x-signature') ?? ''
  const eventName = request.headers.get('x-event-name') ?? ''

  if (!verifyLemonSignature(rawBody, signature)) {
    return new Response('invalid signature', { status: 401 })
  }

  const event = JSON.parse(rawBody)
  const eventId = event.meta?.event_id ?? `${eventName}-${event.data.id}-${event.meta?.test_mode ? 'test' : 'live'}`

  // Idempotency: ON DUPLICATE KEY UPDATE not natural in Drizzle; use try/catch
  try {
    await db.insert(webhookEvents).values({
      id: ulid(),
      provider: 'lemon',
      eventId,
      eventType: eventName,
      payload: event,
      receivedAt: new Date(),
    })
  } catch (e: any) {
    if (e.message?.includes('Duplicate entry')) {
      return new Response('duplicate', { status: 200 })
    }
    throw e
  }

  try {
    await processLemonEvent(eventName, event)
    await db.update(webhookEvents)
      .set({ processedAt: new Date() })
      .where(and(eq(webhookEvents.provider, 'lemon'), eq(webhookEvents.eventId, eventId)))
    return new Response('ok', { status: 200 })
  } catch (err) {
    Sentry.captureException(err, { extra: { eventName, eventId } })
    await db.update(webhookEvents)
      .set({ error: String(err) })
      .where(and(eq(webhookEvents.provider, 'lemon'), eq(webhookEvents.eventId, eventId)))
    return new Response('processing failed', { status: 500 })
  }
}

async function processLemonEvent(eventName: string, event: any) {
  const data = event.data
  const tenantId = event.meta?.custom_data?.tenant_id
  if (!tenantId) {
    // No tenant association; many Lemon events are store-level. Skip safely.
    return
  }

  switch (eventName) {
    case 'subscription_created':
    case 'subscription_updated': {
      const status = data.attributes?.status as string
      await db.update(tenants).set({
        subscriptionStatus: mapLemonStatus(status),
        lemonSubscriptionId: data.id,
        lemonCustomerId: String(data.attributes?.customer_id ?? ''),
      }).where(eq(tenants.id, tenantId))
      break
    }
    case 'subscription_cancelled':
    case 'subscription_expired': {
      await db.update(tenants)
        .set({ subscriptionStatus: 'expired' })
        .where(eq(tenants.id, tenantId))
      break
    }
    case 'subscription_payment_failed': {
      await db.update(tenants)
        .set({ subscriptionStatus: 'past_due' })
        .where(eq(tenants.id, tenantId))
      break
    }
    case 'subscription_payment_success': {
      // record into a periods table; mark tenant active
      await db.update(tenants)
        .set({ subscriptionStatus: 'active' })
        .where(eq(tenants.id, tenantId))
      break
    }
  }
}

function mapLemonStatus(s: string): string {
  switch (s) {
    case 'on_trial': return 'trial'
    case 'active':   return 'active'
    case 'past_due': return 'past_due'
    case 'cancelled': return 'cancelled'
    case 'expired':   return 'expired'
    case 'paused':    return 'paused'
    default:          return 'expired'
  }
}
```

---

## 7. Starting a checkout from Remix

```ts
// apps/web/app/routes/api.billing.checkout.ts
import { json } from '@remix-run/node'
import type { ActionFunctionArgs } from '@remix-run/node'
import { requireTenant } from '~/lib/auth.server'
import { createCheckout } from '~/lib/billing/lemon'

const VARIANT_BY_PLAN: Record<string, string> = {
  starter: process.env.LEMON_VARIANT_STARTER!,
  pro: process.env.LEMON_VARIANT_PRO!,
}

export async function action({ request }: ActionFunctionArgs) {
  const tenant = await requireTenant(request)
  const body = await request.json()
  const variantId = VARIANT_BY_PLAN[body.planId]
  if (!variantId) throw new Response('unknown plan', { status: 400 })

  const url = new URL(request.url)
  const checkout = await createCheckout({
    variantId,
    email: tenant.email,
    tenantId: tenant.id,
    successUrl: `${url.origin}/billing/success`,
  })

  return json({ url: checkout.url })
}
```

In the UI (Remix client component), redirect:

```tsx
// apps/web/app/routes/billing.tsx
const buy = async (planId: string) => {
  const res = await fetch('/api/billing/checkout', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ planId }),
  })
  const { url } = await res.json()
  window.location.href = url
}
```

Lemon's hosted checkout opens in a new browser tab/window; on success it
redirects to the `successUrl` you passed.

---

## 8. Lemon-specific notes

- **One subscription per checkout.** Lemon doesn't bundle multiple
  variants in a single subscription. For "plan + addon" billing, model
  addons as separate subscriptions or use Lemon's *usage-based billing*
  (limited) and a custom monthly invoice. For most solo SaaS, one
  variant = one plan-and-everything-included works.
- **Currency.** Lemon bills in USD by default; EUR and GBP available.
  Customer is shown the currency you configured on the variant.
  Multi-currency requires multiple variants.
- **Customer Portal.** Lemon hosts a customer portal at
  `https://your-store.lemonsqueezy.com/billing` — generate a signed
  customer URL via `/customers/:id/portal` API.
- **Test mode.** Toggle on in dashboard. Test card: `4242 4242 4242 4242`.

---

## 9. Test-mode webhook locally

Lemon doesn't have a CLI like Stripe's. Two options:

1. **ngrok / cloudflared:** expose `localhost:3000` to the internet,
   point Lemon webhook URL at `https://<random>.ngrok.app/api/webhooks/lemon`.
2. **Replay from dashboard:** make a real test purchase, then in
   Lemon dashboard's **Webhooks → Recent Events**, hit Resend on any
   event. The signature is regenerated.

---

## 10. Smoke test

```ts
// apps/web/__tests__/lemon-signature.test.ts
import { describe, it, expect, vi } from 'vitest'
import { verifyLemonSignature } from '~/lib/billing/lemon-signature'
import crypto from 'node:crypto'

describe('verifyLemonSignature', () => {
  it('accepts a valid signature', () => {
    const body = '{"meta":{},"data":{}}'
    const secret = 'test-secret'
    const sig = crypto.createHmac('sha256', secret).update(body).digest('hex')
    vi.stubEnv('LEMON_WEBHOOK_SECRET', secret)
    expect(verifyLemonSignature(body, sig)).toBe(true)
  })
  it('rejects a tampered body', () => {
    const body = '{"meta":{},"data":{}}'
    const secret = 'test-secret'
    const sig = crypto.createHmac('sha256', secret).update(body).digest('hex')
    vi.stubEnv('LEMON_WEBHOOK_SECRET', secret)
    expect(verifyLemonSignature('{"meta":{},"data":{"x":1}}', sig)).toBe(false)
  })
})
```

---

## 11. What this setup does NOT cover

- **VAT / sales tax invoices to your accountant.** Lemon issues invoices
  to your customers; you receive a payout statement only. Reconcile
  monthly via Lemon's reports.
- **Per-tenant custom prices.** Lemon doesn't support arbitrary
  unit_amount per checkout (you reference variants). Use Stripe or
  Paddle if you need this.
- **Complex usage-based billing.** Possible via Lemon's usage records
  but rough. Stripe is the better choice if usage billing is core.
