# 12 — Testing — Gotchas

> **Status:** Outline.

## Categories

1. **Tests pass in CI, fail locally.** Nondeterministic ordering, time
   zone, locale.
2. **Flaky E2E tests.** Mostly: not waiting for content to be visible
   before clicking. Use Playwright's auto-wait correctly.
3. **Test DB pollution.** One test's data leaks to another. Fix:
   transactions per test, rolled back at end.
4. **Mock drift.** Mocked Stripe API doesn't match real Stripe API
   anymore. Fix: pin SDK, run against actual sandbox in integration.
5. **Coverage chasing.** 100% coverage with meaningless tests. Worse
   than 50% with sharp tests.
6. **Snapshot drift.** Snapshot test failures get auto-updated without
   review.

(Real incidents in v0.2.)
