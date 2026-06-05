# Supabase Performance Checklist

> **Status:** Reference (generic playbook artifact).
> A portable, vendor-neutral-where-possible checklist distilled from Supabase
> changelog entries, blog posts, the official performance docs, and real
> production incidents on Supabase-backed SaaS. Use it as a review gate before
> shipping a Supabase-backed feature and as a periodic audit (quarterly, or
> whenever a slow-query alert fires).

Each item is a **decision or check**, not a paragraph. Items marked
**🔒 Supabase-specific** rely on a Supabase-managed feature; on vanilla Postgres
you get the equivalent capability differently (noted inline). Benchmark numbers
come from Supabase's own published tests — treat them as order-of-magnitude, not
guarantees.

---

## 1. Row Level Security (RLS) — the single biggest lever

RLS runs *per row*, so a careless policy is the most common cause of a slow
Supabase query. These moves are cumulative.

- [ ] **Index every column referenced in a policy.** A btree index on
      `user_id` / `tenant_id` used in `USING (...)` turns a seq scan into an
      index scan. → *over 100× on large tables.*
      ```sql
      create index on test_table using btree (user_id);
      ```
- [ ] **Wrap volatile functions in `(select …)`** so the planner runs them once
      (initPlan caching) instead of per row. Applies to `auth.uid()`,
      `auth.jwt()`, `current_setting(...)`, and any `SECURITY DEFINER` helper.
      → *11,000 ms → 10 ms.*
      ```sql
      -- slow:  auth.uid() = user_id
      -- fast: (select auth.uid()) = user_id
      ```
- [ ] **Always specify the role with `TO authenticated` / `TO anon`** so the
      policy is skipped entirely for roles it can't match. → *170 ms → <0.1 ms.*
      ```sql
      create policy "..." on tbl for select to authenticated using (...);
      ```
- [ ] **Rewrite join-in-policy to filter the row's column against a subquery**,
      not the other way round. → *9,000 ms → 20 ms.*
      ```sql
      -- slow:  auth.uid() in (select user_id from team_user where team_id = tbl.team_id)
      -- fast:  team_id in (select team_id from team_user where user_id = (select auth.uid()))
      ```
- [ ] **Push heavy joins into a `SECURITY DEFINER` function** that bypasses RLS
      on the join table. → *178,000 ms → 12 ms.*
      ```sql
      -- team_id in (select user_teams())
      ```
- [ ] **Add the explicit filter in the client query too** — never rely on RLS
      alone to narrow the result set. → *171 ms → 9 ms.*
      ```ts
      supabase.from('tbl').select().eq('user_id', userId)
      ```
- [ ] No `SELECT *` inside a policy subquery; select only the key column.

### Security-adjacent RLS traps (learned the hard way)

These don't show up as "slow" — they show up as *the policy you indexed never
runs at all*, which is worse. A correctness check that belongs in the same review
gate as the perf items above.

- [ ] **Every view over an RLS table carries `WITH (security_invoker = true)`.**
      A Postgres view defaults to running with the **view owner's** rights, which
      **silently bypasses RLS** on the underlying tables — your carefully indexed
      policies are skipped entirely. Re-assert the flag on *every*
      `CREATE OR REPLACE VIEW`; it is trivially lost in a later migration (one
      production project dropped it **4 times** across its history before it was
      caught).
      ```sql
      create view user_plan with (security_invoker = true) as select ...;
      ```
      Supabase's **Database Advisors** surfaces this as the `security_definer_view`
      lint (see §2) — keep it at zero.
- [ ] **Every `SECURITY DEFINER` function guards its caller.** The definer trick
      above runs with elevated rights and bypasses RLS, so each such function
      must (a) self-check the caller and (b) pin its `search_path`:
      ```sql
      -- inside the function body:
      if p_user_id is distinct from (select auth.uid()) then
        raise exception 'unauthorized';
      end if;
      -- and on the function definition:
      ...  security definer set search_path = public;
      ```
      Without the self-check, any authenticated user can pass an arbitrary id and
      act on someone else's rows; without the pinned `search_path` it is open to
      schema-injection.
- [ ] **Atomic counter/ledger mutations inside a definer RPC use `… FOR UPDATE`**
      to row-lock before decrementing (credit balances, quota counters). Without
      the lock, concurrent calls double-spend.

## 2. Indexing & query analysis

- [ ] **`pg_stat_statements`** reviewed for highest total-time and most-called
      queries (enabled by default on Supabase). This is the entry point for any
      slow-query hunt.
- [ ] 🔒 **Query Performance report** (Studio) checked after major schema/query
      changes.
- [ ] 🔒 **`index_advisor`** run on the slow queries it surfaces (Query
      Performance → a query → *Indexes* tab). It builds virtual indexes and
      reports the winning DDL.
- [ ] 🔒 **Database Advisors** (Performance + Security lints) have **zero
      outstanding warnings**: unindexed foreign keys, unused indexes, duplicate
      indexes, multiple permissive policies, RLS-enabled-no-policy,
      `security_definer_view` (§1).
- [ ] **Every foreign key is indexed** (Postgres does not do this
      automatically; unindexed FKs slow joins and cascade deletes).
- [ ] **Explicit table `GRANT`s shipped in a migration** for the `authenticated`
      / `anon` roles — don't rely on Supabase's implicit auto-grants. 🔒 The Data
      API is moving to **enforce** explicit privileges; the symptom of a missing
      grant is **`42501 permission denied`** in Postgres Logs *even with a correct
      RLS policy*. Pair every new user-facing table with its `GRANT`.
- [ ] Composite indexes ordered by selectivity / match the query's `WHERE` +
      `ORDER BY`.
- [ ] Partial indexes for soft-delete / status-filtered hot paths
      (`where deleted_at is null`).
- [ ] Drop unused indexes — they cost write throughput and disk.

## 3. Connections & pooling

- [ ] **Serverless / edge / short-lived clients use the transaction-mode pooler
      (port `6543`, Supavisor).** Direct port `5432` (session mode) is for
      long-lived servers and migrations only.
- [ ] **Prepared statements** kept on where possible — Supavisor supports named
      prepared statements; transaction mode no longer forces you to disable
      them.
- [ ] `default_pool_size` / per-user `pool_size` tuned to compute size, not left
      at default under load.
- [ ] Connection string for the app uses the **pooler host**, not the direct DB
      host, for IPv4 reachability and connection scaling.
- [ ] One client instance reused per process — no `createClient()` per request.

## 4. Compute & disk

- [ ] **Disk IO budget** monitored. Smaller instances *burst* above their gp3
      baseline (3,000 IOPS / 125 MB/s) then throttle — sustained budget use means
      upgrade compute.
- [ ] **Provisioned IOPS / throughput** raised on the disk *only after*
      confirming the compute tier can deliver it (effective IOPS is capped by
      instance size).
- [ ] 🔒 **High-performance disk / larger compute (4XL+)** considered for
      consistent, predictable disk latency on production-critical projects.
- [ ] Database size and bloat watched; `VACUUM`/autovacuum healthy on
      high-churn tables.

## 5. Read replicas & geography

- [ ] 🔒 **Read replicas** added when read load dominates, with reads
      load-balanced across primary + replicas via the pooler.
- [ ] Replicas placed **close to users** for a global audience to cut latency.
- [ ] Read-after-write paths kept on the primary (replicas are eventually
      consistent).

## 6. Caching & data movement

- [ ] 🔒 Storage / Smart CDN used for static assets and cacheable responses so
      they don't hit Postgres.
- [ ] 🔒 Image transformations done at the edge, not in app code.
- [ ] Expensive aggregates materialized (materialized view + scheduled refresh
      via `pg_cron`) instead of recomputed per request.
- [ ] Realtime used **deliberately** — prefer polling for low-frequency data;
      every Realtime subscription with RLS adds per-message policy evaluation.

## 7. Vectors / AI (if using pgvector)

- [ ] **HNSW index** chosen over IVFFlat for query speed/recall on vector
      columns.
- [ ] pgvector kept current (0.6 parallel builds ≈ 30× faster index build;
      0.7 `halfvec`; 0.8 iterative scans) — newer = faster builds and search.
- [ ] Index build run on a temporarily larger compute, then scaled down.

## 8. Postgres version & maintenance

- [ ] On a **current major Postgres** (newer planner, better performance);
      upgrade path (in-place or pause/restore) tested in staging first.
- [ ] Extensions that block an upgrade identified before the upgrade window.
- [ ] `EXPLAIN (ANALYZE, BUFFERS)` is part of the review for any new hot-path
      query.

---

## Quick triage order when "Supabase is slow"

1. `pg_stat_statements` → find the offending query.
2. Is it RLS-bound? → apply §1 (index + `(select …)` + role + rewrite).
3. `EXPLAIN ANALYZE` → missing index? → `index_advisor` (§2).
4. Connection errors / saturation? → pooler mode & pool size (§3).
5. Disk IO budget flat-lining? → compute/disk upgrade (§4).
6. Read-heavy & global? → read replicas (§5).

## Quick triage when "RLS returns rows it shouldn't" (or `42501`)

1. Is the read going through a **view**? → check `WITH (security_invoker = true)`
   (§1) — the #1 silent RLS bypass.
2. Is it a `SECURITY DEFINER` RPC? → confirm the `auth.uid()` self-check (§1).
3. `42501 permission denied` despite a valid policy? → missing `GRANT` (§2).

## Sources

- [RLS Performance and Best Practices — Discussion #14576](https://github.com/orgs/supabase/discussions/14576)
- [Row Level Security | Supabase Docs](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supavisor 1.0: a scalable connection pooler for Postgres](https://supabase.com/blog/supavisor-postgres-connection-pooler)
- [Connection management | Supabase Docs](https://supabase.com/docs/guides/database/connection-management)
- [Read Replicas | Supabase Docs](https://supabase.com/docs/guides/platform/read-replicas)
- [pg_stat_statements | Supabase Docs](https://supabase.com/docs/guides/database/extensions/pg_stat_statements)
- [index_advisor | Supabase Docs](https://supabase.com/docs/guides/database/extensions/index_advisor)
- [Performance and Security Advisors | Supabase Docs](https://supabase.com/docs/guides/database/database-advisors)
- [Compute and Disk | Supabase Docs](https://supabase.com/docs/guides/platform/compute-and-disk)
- [High Performance Disk | Supabase Blog](https://supabase.com/blog/high-performance-disks)
- [pgvector v0.6.0: 30x faster with parallel index builds](https://supabase.com/blog/pgvector-fast-builds)
- [HNSW indexes | Supabase Docs](https://supabase.com/docs/guides/ai/vector-indexes/hnsw-indexes)
- [Upgrade to Postgres 17 | Supabase Docs](https://supabase.com/docs/guides/platform/upgrading)

### Production-incident sources (the §1 "learned the hard way" items)

The `security_invoker`, `SECURITY DEFINER` self-check, `FOR UPDATE` ledger, and
explicit-`GRANT` items are distilled from a real Supabase SaaS migration history
(CutOptim): the `user_plan` view lost its `security_invoker` flag across four
migrations before it was caught; two `SECURITY DEFINER` RPCs shipped without an
`auth.uid()` self-check (later `014_security_definer_self_check.sql`); a credit
ledger needed `FOR UPDATE` for FIFO decrement; and the Data API GRANT-enforcement
rollout required explicit `GRANT`s (`019_explicit_grants.sql`).
