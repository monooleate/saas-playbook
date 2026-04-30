# 01 — Infrastructure & DevOps — Best Practices

What the operators of large SaaS publicly recommend, and where this playbook
agrees, disagrees, or simplifies for solo founders.

## 1. Treat servers as cattle, not pets — but only when you have more than one

The "cattle not pets" mantra (Bill Baker, Microsoft, ~2012) is an SRE
principle for fleets. With one VPS, you have a pet, and that's fine. The
principle to apply at every scale is the *reproducibility* part: anything
done by hand on the server must also be doable by re-running a script.

**Source:** [Pets vs. Cattle: The Elastic Cloud Crash Course](https://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/),
Randy Bias, 2016.

**Solo founder application:** keep a `internal-docs/VPS-SETUP.md` that
contains every command run during initial setup. The Grabit reference
implementation does this — it's how setup time on the next VPS dropped from
~2 days to ~2 hours. [Grabit's VPS-SETUP doc has 33 sections including
fail2ban tuning, Caddyfile evolution, and post-mortems for every gotcha.]

## 2. Use TLS everywhere; obtain certificates with DNS-01 if you have wildcards

Let's Encrypt's HTTP-01 challenge requires a working HTTP listener at the
exact hostname. Wildcard certificates (`*.example.com`) cannot use HTTP-01;
they require DNS-01.

**Source:** [Let's Encrypt — Challenge Types](https://letsencrypt.org/docs/challenge-types/).

Caddy's documentation on the [Cloudflare DNS plugin](https://github.com/caddy-dns/cloudflare)
shows the `xcaddy build --with github.com/caddy-dns/cloudflare` recipe and
the `tls { dns cloudflare ... }` Caddyfile directive. This is the de facto
solo-founder pattern.

Vercel and Fly handle this internally if you stay on their platform.

## 3. Don't terminate TLS in your application

Stripe's engineering blog notes that they terminate TLS at NLBs/HAProxy
specifically so the application can stay simple and stateless. The same
principle scales down to one VPS: terminate at Caddy/Nginx, never inside
Node/Bun.

**Source:** [Online migrations at scale](https://stripe.com/blog/online-migrations) — Stripe Engineering, 2017 (architecture diagrams).

**Why it matters at solo scale:** if you upgrade Node and your TLS stack
breaks, you've taken down both your app and HTTPS. Separating them means
each can be replaced independently.

## 4. Build the dependency graph; let the build system invalidate caches

Turbo (`turbo.json`), Nx, Lerna, and Bazel all do the same thing: declare
which task depends on which artifact, hash inputs, skip work on cache hit.

**Source:** [Turborepo — How tasks work](https://turborepo.com/docs/crafting-your-repository/running-tasks).

The Grabit `turbo.json` is famously short: four tasks (`dev`, `build`,
`typecheck`, `lint`), each declaring outputs or dependencies. That's enough
for a solo founder.

**What to skip:** remote build caches (Vercel Remote Cache, Nx Cloud) for
solo projects. The single-machine cache is enough; remote cache complicates
debugging.

## 5. Zero-downtime deploys come from process supervision, not orchestration

PM2's `startOrReload` does a graceful reload: spin up new process, pass
traffic, drain old. Same model as `systemctl reload nginx`, `unicorn -USR2`,
or Kubernetes rolling updates. You do not need Kubernetes for this.

**Source:** [PM2 — Process management](https://pm2.keymetrics.io/docs/usage/process-management/).

**For Next.js / Astro static-only:** zero-downtime is automatic with a
reverse proxy serving the new directory atomically (rename trick). No
process manager needed.

## 6. Pin runtime versions; let dependency versions float by default

Pin Node version (`.nvmrc`, `engines` in package.json, Hetzner-side install
script). Pin pnpm via `packageManager` in `package.json`. Don't pin every
npm dependency to an exact version — let `^` ranges flow, lock with
`pnpm-lock.yaml`.

**Source:** [Node.js — Version management](https://nodejs.org/en/learn/getting-started/how-to-install-nodejs).
**Counter-source:** Some teams (notably Slack engineering) pin everything
exactly. For solo founders, this is over-engineering until you've been bitten.

## 7. Use a monorepo when packages must be in lockstep, not because it's trendy

Linear, Vercel, and Stripe all use monorepos. None of them do it because
"monorepo good" — they do it because their billing/api/sdk/docs share types
that must move together.

**Source:** [Why we use a monorepo](https://vercel.com/blog/why-we-use-a-monorepo) — Vercel Engineering, 2021.

**Solo founder rule:** monorepo if and only if you have a `db` or `billing`
package shared between two apps (e.g. marketing site and product app).
Otherwise a single repo with one `package.json` is faster to navigate.

## 8. Wildcard subdomains for tenant routing — but only one cert, not N

For multi-tenant SaaS where each tenant gets `tenant.app.com`:

- **One** wildcard cert `*.app.com` (DNS-01).
- **One** Caddy block `*.app.com { reverse_proxy localhost:3000 }`.
- App reads `Host` header to identify the tenant.

This is how Vercel, Linear, Notion all work — you don't see N certs being
issued, you see one wildcard. The exception is "custom domain per tenant"
(e.g. Webflow, Carrd), which requires per-domain certs and on-the-fly
issuance. That's topic 02 territory.

**Source:** [Caddy — On-Demand TLS](https://caddyserver.com/docs/automatic-https#on-demand-tls)
covers the per-tenant-domain case if you need it later.

## 9. Server-side rendering needs *server-side* persistence; client-side
   does not

If your app SSRs (Next.js App Router, SvelteKit, Remix), the server is doing
DB queries on every navigation. Latency between app process and database
matters: same-region only.

**Source:** [Vercel — Edge Functions](https://vercel.com/docs/functions/edge-functions)
docs explicitly warn about cold-region DB access.

**Solo founder rule:** put the app and the DB in the same region. Hetzner
Falkenstein + Supabase EU = good. Hetzner Falkenstein + Supabase US =
sluggish. For a global audience, Cloudflare Workers + a globally-replicated
DB (PlanetScale, Turso) is the alternative — see Remix + PlanetScale example.

## 10. Cloudflare proxy: ON for static assets, OFF for cert challenges

Cloudflare's "orange cloud" is great for free DDoS protection and CDN, but
the proxy intercepts HTTP-01 challenges from Let's Encrypt — and DNS-01
challenges break if the API token is misconfigured. The Grabit setup turned
the proxy OFF (grey cloud) on the A records and let Caddy handle TLS
directly. Cloudflare's [Universal SSL](https://developers.cloudflare.com/ssl/edge-certificates/universal-ssl/)
is the alternative — proxy ON, Cloudflare terminates TLS, origin can be
HTTP. Pick one model and document it.

## 11. Health endpoints should fail correctly

`/healthz` returning 200 unconditionally is theatre. It must check the
critical downstream — typically the database. Otherwise the load balancer
keeps sending traffic to a process that can't serve requests.

**Source:** [Kubernetes — Liveness, Readiness probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
introduced the formal liveness/readiness distinction. Same idea applies
to UptimeRobot.

```ts
// Healthcheck pattern that has saved many incidents
app.get('/healthz', async (req, res) => {
  try {
    await db.raw('SELECT 1');
    res.status(200).send('ok');
  } catch (err) {
    res.status(503).send('db unreachable');
  }
});
```

## 12. Deploy in 5 minutes or it doesn't get done

Vercel's whole pitch is "git push, it deploys." Linear, Stripe, and Notion
all have similar internal speed. The reason is psychological: a slow deploy
discourages small fixes. Fast deploys are a line of defense, not a luxury.

**Source:** [Accelerate](https://itrevolution.com/product/accelerate/) by
Forsgren / Humble / Kim — DORA metrics, "deploy frequency" and "lead time
for changes" predict business performance.

**Solo founder version:** GitHub Actions to a single VPS via SSH is
typically 60–90 seconds. If yours is over 5 minutes, optimize:
- Cache `pnpm` store between runs.
- Build only the changed app (Turbo handles this).
- Skip CI on doc-only commits (`if: !contains(github.event.head_commit.message, '[skip ci]')`).

## 13. There is no rollback; only forward to a known-good revision

"Rollback" in production is `git reset --hard <prev-commit>` followed by
running the same build pipeline. There is no magic snapshot. This is why:

- Migrations must be backwards-compatible (topic 14).
- `git push --force` to main is a near-incident in itself (topic 11).
- The CI pipeline must work for old commits too. Don't rewrite it
  destructively.

**Source:** [GitOps principles](https://opengitops.dev/) make this explicit:
the desired state is whatever git says.

## 14. Cost reviews are an engineering practice

The single biggest infrastructure cost surprise for solo SaaS in 2025–26
was Vercel bandwidth. The second was OpenAI inference. Setting a billing
cap is non-optional.

**Source:** [Vercel — Spend Management](https://vercel.com/docs/pricing/manage-and-optimize-spend).

**Solo founder rule:** every paid service gets a hard cap. If a viral
spike would 100× your bill, you'd rather go down than be bankrupt. Stripe
has a [`metered billing alerts`](https://stripe.com/docs/billing/subscriptions/usage-based)
pattern; AWS has Budgets; Vercel has Spend Cap. Use them.
