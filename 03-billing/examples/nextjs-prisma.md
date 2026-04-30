# 03 — Billing — Example: Next.js + Prisma + Stripe

The "direct provider" shape. Stripe is the merchant of record for you,
which means lower fees, more control, and tax responsibility on your side
(use Stripe Tax + an accountant).

---

## 1. Stack

| Layer            | Choice                                |
|------------------|---------------------------------------|
| Framework        | Next.js 15 (App Router)               |
| ORM              | Prisma 5                              |
| DB               | Postgres (Neon)                       |
| Provider         | Stripe (with Stripe Tax + Customer Portal) |
| Hosted checkout  | Stripe Checkout Sessions              |
| Webhooks         | Stripe webhook endpoint, signed       |

---

## 2. Prisma schema additions

```prisma
// apps/web/prisma/schema.prisma

model Tenant {
  id                       String   @id @default(cuid())
  email                    String   @unique
  countryCode              String?  @map("country_code")
  currency                 String?
  plan                     String   @default("trial")
  subscriptionStatus       String   @default("trial") @map("subscription_status")
  trialEndsAt              DateTime? @map("trial_ends_at")
  stripeCustomerId         String?  @unique @map("stripe_customer_id")
  stripeSubscriptionId     String?  @unique @map("stripe_subscription_id")

  addons        TenantAddon[]
  periods       SubscriptionPeriod[]
  invoices      Invoice[]
}

model TenantAddon {
  id              String   @id @default(cuid())
  tenantId        String   @map("tenant_id")
  tenant          Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  addonSlug       String   @map("addon_slug")
  isActive        Boolean  @default(true) @map("is_active")
  quantity        Int      @default(1)
  config          Json     @default("{}")
  activatedAt     DateTime @default(now()) @map("activated_at")
  deactivatedAt   DateTime? @map("deactivated_at")

  @@unique([tenantId, addonSlug])
}

model SubscriptionPeriod {
  id           String   @id @default(cuid())
  tenantId     String   @map("tenant_id")
  tenant       Tenant   @relation(fields: [tenantId], references: [id])
  amount       Decimal  @db.Decimal(12, 2)
  currency     String
  periodStart  DateTime @map("period_start")
  periodEnd    DateTime @map("period_end")
  chargeStatus String   @map("charge_status")
  paymentId    String?  @map("payment_id")
  invoiceId    String?  @map("invoice_id")
  createdAt    DateTime @default(now()) @map("created_at")
}

model WebhookEvent {
  id           String   @id @default(cuid())
  provider     String
  eventId      String   @map("event_id")
  eventType    String   @map("event_type")
  payload      Json
  receivedAt   DateTime @default(now()) @map("received_at")
  processedAt  DateTime? @map("processed_at")
  error        String?

  @@unique([provider, eventId])
}

model Invoice {
  id                String   @id @default(cuid())
  tenantId          String   @map("tenant_id")
  tenant            Tenant   @relation(fields: [tenantId], references: [id])
  provider          String
  providerInvoiceId String   @map("provider_invoice_id")
  invoiceNumber     String   @map("invoice_number")
  amount            Decimal  @db.Decimal(12, 2)
  currency          String
  status            String
  pdfUrl            String?  @map("pdf_url")
  issuedAt          DateTime @map("issued_at")

  @@unique([provider, providerInvoiceId])
}
```

Migrate:

```bash
pnpm prisma migrate dev --name add_billing
```

---

## 3. Stripe SDK + Checkout

```ts
// apps/web/lib/billing/stripe.ts
import 'server-only'
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-11-20.acacia' as Stripe.LatestApiVersion,
  typescript: true,
})

export async function createCheckoutSession(args: {
  tenantId: string
  email: string
  planId: 'starter' | 'pro'
  amount: number      // major units
  currency: string
  trialDays: number
  origin: string
}): Promise<{ url: string; sessionId: string }> {
  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    customer_email: args.email,
    success_url: `${args.origin}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url:  `${args.origin}/billing?cancelled=1`,
    line_items: [{
      price_data: {
        currency: args.currency.toLowerCase(),
        product_data: { name: args.planId === 'pro' ? 'Pro Plan' : 'Starter Plan' },
        unit_amount: toMinor(args.amount, args.currency),
        recurring: { interval: 'month' },
      },
      quantity: 1,
    }],
    subscription_data: {
      trial_period_days: args.trialDays,
      metadata: { tenantId: args.tenantId, plan: args.planId },
    },
    automatic_tax: { enabled: true },
    tax_id_collection: { enabled: true },
    customer_update: { address: 'auto', name: 'auto' },
    metadata: { tenantId: args.tenantId },
  }, {
    idempotencyKey: `checkout-${args.tenantId}-${args.planId}`,
  })

  return { url: session.url!, sessionId: session.id }
}

function toMinor(major: number, currency: string): number {
  if (['HUF', 'JPY', 'KRW'].includes(currency)) return Math.round(major)
  return Math.round(major * 100)
}
```

---

## 4. Checkout API route

```ts
// apps/web/app/api/billing/checkout/route.ts
import { NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { db } from '@yourorg/db'
import { createCheckoutSession } from '@/lib/billing/stripe'
import pricing from '@yourorg/billing/pricing.json'
import { getCurrencyForCountry, getPriceForCurrency } from '@yourorg/billing/currency'

export async function POST(req: Request) {
  const session = await auth()
  if (!session?.tenantId) return NextResponse.json({ error: 'unauthorized' }, { status: 401 })

  const { planId } = await req.json()
  const tenant = await db.tenant.findUniqueOrThrow({ where: { id: session.tenantId } })

  const currency = getCurrencyForCountry(tenant.countryCode)
  const plan = (pricing as any).plans[planId]
  if (!plan) return NextResponse.json({ error: 'unknown plan' }, { status: 400 })

  const amount = getPriceForCurrency(plan.monthlyPrice, plan.monthlyPriceByCurrency, currency)

  const origin = new URL(req.url).origin
  const checkout = await createCheckoutSession({
    tenantId: tenant.id,
    email: tenant.email,
    planId,
    amount,
    currency,
    trialDays: plan.trialDays ?? 14,
    origin,
  })

  return NextResponse.json({ url: checkout.url })
}
```

---

## 5. Webhook handler

```ts
// apps/web/app/api/webhooks/stripe/route.ts
import { NextResponse } from 'next/server'
import Stripe from 'stripe'
import { stripe } from '@/lib/billing/stripe'
import { db } from '@yourorg/db'
import * as Sentry from '@sentry/nextjs'

export const dynamic = 'force-dynamic'
export const runtime = 'nodejs'      // not edge

const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET!

export async function POST(req: Request) {
  const rawBody = await req.text()
  const sig = req.headers.get('stripe-signature') ?? ''

  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(rawBody, sig, WEBHOOK_SECRET)
  } catch (err) {
    return new NextResponse(`signature: ${err}`, { status: 401 })
  }

  // Idempotency
  let inserted
  try {
    inserted = await db.webhookEvent.create({
      data: {
        provider: 'stripe',
        eventId: event.id,
        eventType: event.type,
        payload: event as any,
      },
    })
  } catch (e: any) {
    if (e.code === 'P2002') {
      // unique constraint = duplicate, OK
      return new NextResponse('duplicate', { status: 200 })
    }
    throw e
  }

  try {
    await processEvent(event)
    await db.webhookEvent.update({
      where: { id: inserted.id },
      data: { processedAt: new Date() },
    })
    return new NextResponse('ok', { status: 200 })
  } catch (err) {
    Sentry.captureException(err, { extra: { eventId: event.id, type: event.type } })
    await db.webhookEvent.update({
      where: { id: inserted.id },
      data: { error: String(err) },
    })
    return new NextResponse('processing failed', { status: 500 })
  }
}

async function processEvent(event: Stripe.Event) {
  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated': {
      const sub = event.data.object
      const tenantId = sub.metadata?.tenantId
      if (!tenantId) throw new Error('subscription missing tenantId metadata')

      await db.tenant.update({
        where: { id: tenantId },
        data: {
          subscriptionStatus: mapStatus(sub.status),
          stripeSubscriptionId: sub.id,
          stripeCustomerId: sub.customer as string,
          plan: sub.metadata?.plan ?? undefined,
        },
      })
      break
    }

    case 'customer.subscription.deleted': {
      const sub = event.data.object
      const tenantId = sub.metadata?.tenantId
      if (!tenantId) return
      await db.tenant.update({
        where: { id: tenantId },
        data: { subscriptionStatus: 'expired' },
      })
      break
    }

    case 'invoice.payment_succeeded': {
      const inv = event.data.object
      const tenantId = await tenantFromInvoice(inv)
      if (!tenantId) return
      await db.subscriptionPeriod.create({
        data: {
          tenantId,
          amount: (inv.amount_paid ?? 0) / 100,
          currency: (inv.currency ?? 'usd').toUpperCase(),
          periodStart: new Date((inv.period_start ?? 0) * 1000),
          periodEnd:   new Date((inv.period_end ?? 0) * 1000),
          chargeStatus: 'succeeded',
          paymentId: inv.payment_intent as string ?? null,
          invoiceId: inv.id ?? null,
        },
      })
      // also persist Invoice record
      break
    }

    case 'invoice.payment_failed': {
      const inv = event.data.object
      const tenantId = await tenantFromInvoice(inv)
      if (!tenantId) return
      await db.tenant.update({
        where: { id: tenantId },
        data: { subscriptionStatus: 'past_due' },
      })
      // queue past-due email — see topic 04
      break
    }
  }
}

async function tenantFromInvoice(inv: Stripe.Invoice): Promise<string | null> {
  const subId = (inv as any).subscription
  if (!subId) return null
  const tenant = await db.tenant.findFirst({ where: { stripeSubscriptionId: subId } })
  return tenant?.id ?? null
}

function mapStatus(s: Stripe.Subscription.Status): string {
  switch (s) {
    case 'trialing': return 'trial'
    case 'active': return 'active'
    case 'past_due': return 'past_due'
    case 'canceled': return 'cancelled'
    case 'paused': return 'paused'
    default: return 'expired'
  }
}
```

---

## 6. Customer Portal redirect

Saves you from building "update card / cancel / view invoices" UI.

```ts
// apps/web/app/api/billing/portal/route.ts
import { NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { db } from '@yourorg/db'
import { stripe } from '@/lib/billing/stripe'

export async function POST(req: Request) {
  const session = await auth()
  if (!session?.tenantId) return NextResponse.json({ error: 'unauthorized' }, { status: 401 })

  const tenant = await db.tenant.findUniqueOrThrow({ where: { id: session.tenantId } })
  if (!tenant.stripeCustomerId) return NextResponse.json({ error: 'no customer' }, { status: 400 })

  const portal = await stripe.billingPortal.sessions.create({
    customer: tenant.stripeCustomerId,
    return_url: new URL('/billing', req.url).toString(),
  })

  return NextResponse.json({ url: portal.url })
}
```

Configure the portal: Stripe Dashboard → Settings → Billing → Customer
portal. Enable: cancel, update card, update billing details, invoice
history. Disable plan switching unless your products are simple (mixed
plans + addons via the portal can confuse customers).

---

## 7. Local development with Stripe CLI

```bash
# Install
brew install stripe/stripe-cli/stripe

# Auth
stripe login

# Forward events to local dev
stripe listen --forward-to localhost:3000/api/webhooks/stripe
# Outputs: webhook signing secret whsec_...
# Put this in .env.local as STRIPE_WEBHOOK_SECRET

# Trigger a test event
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
```

The CLI signs events with your test webhook secret automatically.

---

## 8. Smoke tests

```ts
// apps/web/__tests__/billing.smoke.test.ts
import { describe, it, expect } from 'vitest'
import pricing from '@yourorg/billing/pricing.json'
import { getPriceForCurrency } from '@yourorg/billing/currency'

describe('pricing.json: every plan has positive price in every supported currency', () => {
  for (const planId of Object.keys((pricing as any).plans)) {
    for (const currency of (pricing as any).supportedCurrencies as string[]) {
      it(`${planId} × ${currency}`, () => {
        const plan = (pricing as any).plans[planId]
        const price = getPriceForCurrency(
          plan.monthlyPrice,
          plan.monthlyPriceByCurrency,
          currency,
        )
        expect(price).toBeGreaterThan(0)
        expect(price).toBeLessThan(100000)   // sanity ceiling
      })
    }
  }
})
```

---

## 9. Don't-forgets

- Stripe Tax: pass `automatic_tax: { enabled: true }` on every checkout
  and subscription create. Account-level enable is not enough.
- Customer email is the natural unique key. If a customer signs up twice
  with the same email, dedupe on email or use `tenant.email`.
- Currency is locked at subscription creation. Test mode lets you create
  with any currency; prod will reject some combinations.
- Migrations during build: `prisma migrate deploy` is fine for solo
  scale. At scale, run as a separate gated step (topic 14).
- Pre-launch: turn `automatic_tax` off only if Stripe Tax isn't
  registered in the right jurisdictions. Better: register first, then
  launch.
