# 18 — Public API & Webhooks — Best Practices

> **Status:** Outline.

## Sources

- Stripe API docs.
- Zalando RESTful API guidelines.
- Hookdeck / Svix engineering blogs.

## Practices

- Idempotency on every mutation.
- Versioning from day 1.
- Pagination is mandatory; default limits.
- Errors are typed; provide a `code` you commit to.
- Sign webhooks; document the signature scheme.
- Treat the API as a product: track usage, request docs feedback.
