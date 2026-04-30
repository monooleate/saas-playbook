# 05 — Feature Flags & Addon Gating — Gotchas

> **Status:** Outline.

## Categories

1. **Stale flags.** A flag that was 100% rolled out 8 months ago is
   still in the codebase as `if (flag) { ... }`. Multiplied by 30
   flags = unreadable code.
2. **Flag eval inside a tight loop.** Each `isEnabled()` is a network
   call to PostHog → page hangs. Fix: cache per request.
3. **Server says yes, client says no.** Flag value evaluated on server
   render, then re-evaluated client-side with different cookie/user
   context. UI flicker. Fix: pass server eval to client as initial state.
4. **Addon disabled in DB but feature still works.** Code uses `if
   (tenant.plan === 'pro')` instead of `isAddonActive()`. Fix: lint
   rule banning string-equality checks on `tenant.plan`.
5. **Rollout to 1% picks the same 1% every time.** Random function not
   seeded by tenant ID; user gets feature on Monday, doesn't on
   Tuesday.
6. **Coupon disables addon billing but not addon access.** Customer
   keeps using the addon for free forever. Fix: status reconciliation
   cron.

(Real incidents to be documented in v0.2.)
