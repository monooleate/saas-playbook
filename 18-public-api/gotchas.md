# 18 — Public API & Webhooks — Gotchas

> **Status:** Outline.

## Categories

1. **Breaking change without versioning.** All integrators broken
   overnight.
2. **Unsigned webhook payload.** Customers can't trust origin; could
   be spoofed.
3. **Webhook timeouts cascade into mass retries.** Without backoff
   ramp, a slow customer endpoint amplifies.
4. **Rate limits per IP, not per key.** Shared infra clients
   throttled.
5. **Pagination via offset on huge tables.** Use cursor-based.
6. **N+1 in list endpoints.** Especially when including related
   objects.
7. **OpenAPI schema drifts from real behaviour.** Generate, don't
   write by hand.

(Real incidents in v0.2.)
