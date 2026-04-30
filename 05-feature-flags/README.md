# 05 — Feature Flags & Addon Gating

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** After billing (topic 03). Before pricing
> changes that affect existing customers.
> **Cost:** 2–4 days.

## Scope

Two mechanisms with the same query surface (`isEnabled('foo', { tenant
})`) but different motivations:

### Feature flags (gradual rollout, A/B, kill switch)

- Roll a feature out to 1% of tenants, then 10%, then 100%.
- A/B test variants of the onboarding flow.
- Kill switch for an experimental feature that misbehaves in prod.
- Per-employee/per-tenant overrides for support and demos.

Tools: PostHog Feature Flags (free up to ~1M evaluations), LaunchDarkly
(enterprise), GrowthBook (open-source), or roll your own (a JSON file
+ a deploy is enough at small scale).

### Addon / plan gating (entitlements)

- Plan A unlocks features X and Y; Plan B unlocks X, Y, and Z.
- Addons turn individual features on/off at extra cost.
- Limits: max 5 users on Starter, unlimited on Pro.

This is *contractual* state — driven by what the tenant pays for, not by
a rollout decision. Unlike feature flags, customers can see and toggle
their own addons (when allowed) and the changes affect their bill
(topic 03).

## The shared abstraction

```ts
async function isEnabled(
  key: string,
  ctx: { tenantId: string; userId?: string }
): Promise<boolean>
```

Implementations:
- For feature flags: query PostHog (or your config) with the tenant
  ID as the distinct key.
- For addons: query the `tenant_addons` table.

Most code only cares "is X on for this tenant?" The two backends
disappear behind one function.

## The Grabit pattern (addon gating)

The Grabit project uses a `tenant_addons` table (see topic 03 example):

```ts
async function isAddonActive(tenantId: string, addonSlug: string) {
  const { data } = await supabase
    .from('tenant_addons')
    .select('is_active')
    .eq('tenant_id', tenantId)
    .eq('addon_slug', addonSlug)
    .eq('is_active', true)
    .single()
  return !!data
}
```

Guard usage:

```ts
async function requireAddon(tenantId: string, slug: string, opts) {
  if (await isAddonActive(tenantId, slug)) return
  if (opts.mode === 'redirect') throw redirect(303, '/admin/subscription')
  throw error(opts.status ?? 403, opts.message)
}
```

Two short-cuts: `requireResourcePillar()`, `requireEventPillar()` for
the most common ones.

## Why it matters

Without gating, someone on the free plan can hit a paid endpoint and
use it. Without flags, every code change goes to 100% of users on
deploy — there's no soft-launch mechanism.

The mistake to avoid: hard-coding `if (tenant.plan === 'pro')` in 50
places. When the plan structure changes (new tier, addon split),
you're searching the codebase. Centralize behind `isEnabled()`.

## What goes in v0.2

- Decision matrix: when to use a flag vs an addon vs a plan.
- PostHog Feature Flags integration walkthrough.
- The `entitlements` table pattern (denormalized cache of "what does
  this tenant have access to").
- Per-user vs per-tenant flags.
- Rollout strategies: percentage, allow-list, by country, by signup
  cohort.
- Cleanup discipline: remove flags after rollout completes.

## Sources to mine

- PostHog feature flags: [posthog.com/docs/feature-flags](https://posthog.com/docs/feature-flags).
- LaunchDarkly's "Effective Feature Management":
  [launchdarkly.com/effective-feature-management/](https://launchdarkly.com/effective-feature-management/).
- GrowthBook: [growthbook.io](https://www.growthbook.io).
- Stripe's plan/feature mapping: their internal pattern is published
  in talks; we'll cite the most relevant ones.
