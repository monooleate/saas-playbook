# 18 — Public API & Webhooks

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** When the first integration request lands.
> **Cost:** 5–10 days for v1 of API; ongoing.

## Scope

When customers (or their integrators) want to read/write your data
programmatically:

1. **API design** — REST vs RPC vs GraphQL.
2. **API keys** — issuance, scoping, rotation.
3. **Versioning** — date-based (Stripe) or semver.
4. **Rate limits** — per-key, per-tenant.
5. **Outbound webhooks** — letting customers subscribe to events.
6. **Webhook delivery** — retries, signing, replay.
7. **OpenAPI / docs** — auto-generated from the schema.
8. **Auth modes** — bearer token, OAuth (for third-party integrations).
9. **SDKs** — when to build, when to just publish OpenAPI and let
   tools generate them.

## Why it matters

Even if "we don't need an API" is true today, the architecture you
choose now shapes what's possible later. Stripe's API is its product;
your API will be a feature long before it's a product.

## REST is fine

Your customers' integrators are familiar with REST. GraphQL is great
for complex internal APIs; less great as a public surface for
infrequent users. Stick with REST + JSON.

URL shape:

```
GET    /api/v1/bookings
POST   /api/v1/bookings
GET    /api/v1/bookings/:id
PATCH  /api/v1/bookings/:id
DELETE /api/v1/bookings/:id
```

Authentication: `Authorization: Bearer <api_key>`. The key embeds
the tenant.

## Versioning

Two patterns:

- **Date-based** (Stripe): `Stripe-Version: 2024-11-20.acacia`. Each
  date is a snapshot. Old behaviour preserved indefinitely.
- **URL semver**: `/api/v1`, `/api/v2`. Cleaner conceptually, but
  forces all changes to a v2 even when small.

For solo SaaS, **URL semver** is sufficient. Move to date-based when
you have integrators who'll scream when you change behaviour.

## Outbound webhooks (your customers receive your events)

Mirror image of receiving Stripe webhooks (topic 03):

- Customer registers a URL via your API or settings UI.
- You sign every payload with HMAC.
- You retry on non-2xx with exponential backoff (e.g. 1s, 5s, 30s,
  5m, 1h, 6h, 24h, then dead-letter).
- Customer can replay from a UI.
- Failure rate per endpoint visible to the customer.

This is essentially what Stripe does for *you*; you become Stripe to
your integrators.

## What goes in v0.2

- Full API design walkthrough.
- API key model + issuance UI.
- Rate limit patterns.
- Webhook delivery service (Inngest, Trigger.dev, or DIY).
- OpenAPI generation (Hono / tRPC / Zod-to-OpenAPI).
- SDKs and code samples in docs.

## Sources

- Stripe API docs (the canonical model).
- [Zalando RESTful API guidelines](https://opensource.zalando.com/restful-api-guidelines/).
- Vercel API docs.
- Hookdeck / Svix as managed webhook delivery services worth
  evaluating.
