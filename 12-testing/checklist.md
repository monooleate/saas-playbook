# 12 — Testing — Checklist

> **Status:** Outline.

- Test runner installed (Vitest preferred for Vite-based stacks).
- Unit tests for: pricing math, currency conversion, key validators.
- Integration tests for: webhook handler (idempotency, signature
  rejection), signup endpoint.
- Playwright installed; one E2E spec for signup → payment → addon
  toggle.
- CI runs all of the above on every PR.
- A test that asserts tenant A cannot read tenant B's data.
- Smoke test in production: a `/healthz` integration test that runs
  post-deploy.
- Coverage thresholds set (only for the `billing` package; the rest is
  fine without).
- Snapshot tests are deliberate; not auto-generated.
