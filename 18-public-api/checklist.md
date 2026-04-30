# 18 — Public API & Webhooks — Checklist

> **Status:** Outline.

## API surface

- API base URL (`/api/v1` or `api.example.com`).
- Auth: `Authorization: Bearer <key>`.
- API keys issuable from the tenant settings UI.
- Keys scoped (read / read-write) and revocable.
- All resources documented in OpenAPI.

## Reliability

- Rate limit per key.
- Idempotency-key header supported on all mutations.
- Pagination on all list endpoints.
- Errors return JSON with `code` + `message` + `request_id`.

## Outbound webhooks

- Customer can register URLs via API or UI.
- Every payload HMAC-signed.
- Retry schedule: 1s, 5s, 30s, 5m, 1h, 6h, 24h.
- Dead-letter queue with replay UI.
- Customer-facing delivery log.

## Docs

- OpenAPI auto-published.
- Postman collection generated.
- "Getting started" tutorial published.
- At least one SDK or one code sample per popular language (curl,
  JS, Python).
