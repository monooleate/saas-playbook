# 12 — Testing

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** When pain > pleasure.
> **Cost:** 2–10 days for the first useful suite.

## Scope

Tests for a solo SaaS — pragmatic, not aspirational.

1. **Unit tests** — pure functions, formatters, calculators.
2. **Integration tests** — DB queries, API endpoints with a real DB.
3. **End-to-end tests** — Playwright walking through critical flows.
4. **Smoke tests** — production-side health checks.
5. **Visual regression** — only for marketing-page-heavy products.

## The solo-founder anti-overtest rule

You are not Google. You don't have an SRE team. Tests have a cost
(write, run, maintain). A test that fails for the wrong reason costs
more than the bug it would have caught.

**Pareto rule for solo testing:**

- **High ROI:** unit tests for billing math, currency conversion, plan
  computation. Integration tests for the webhook handler and the
  signup flow. E2E for the trial-to-paid path. ~50 tests, ~5s to run.
- **Medium ROI:** unit tests for key validators (email, slug, country
  code). Integration for any RLS policy.
- **Low ROI / skip:** snapshot tests of UI markup. Tests of trivial
  CRUD with no business logic. Mocking everything just to assert
  function calls happen in order.

The Grabit test setup, after 1 year of production:
- 1 unit suite (`packages/billing/tests`) of ~80 tests covering plan
  math, currency, FX, addon math, idempotency keys.
- 1 integration suite testing webhook handlers against a Supabase
  test instance.
- 5 Playwright E2E specs covering signup, billing, addon toggle,
  cancellation, superadmin impersonation.

That's it. ~200 tests total. Runs in ~30s on CI.

## What goes in v0.2

- Choosing a test runner (Vitest, Bun test, Jest).
- DB-touching tests: per-test transactions vs per-suite reset.
- Playwright setup (one-time pain, then easy).
- CI runner: GitHub Actions concurrency, caching.
- Visual regression with Chromatic / Percy when needed.

## Sources

- Kent C. Dodds' testing trophy:
  [kentcdodds.com/blog/the-testing-trophy-and-testing-classifications](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications).
- Vitest docs.
- Playwright docs.
