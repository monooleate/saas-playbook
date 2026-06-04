# GAPS — Item-level backlog

> **What this is.** A central rollup of every concrete gap-derived task
> across all 25 topics. One row per task. Mirror of the per-topic
> `## Open gaps & sprint-pickable tasks` sections at the end of each
> topic's `README.md`.
>
> **Why it exists.** Sprint planners scan one file, not 25. The matrix
> philosophy (see `../CLAUDE.md`) treats every topic as a row that
> produces three outputs: content, gap analysis, sprint-pickable tasks.
> This file is where output #3 collects.

**Pair this with:**

- [`MATRIX.md`](./MATRIX.md) — topic-level rollup (one row per topic,
  high-level "Remaining work" cell).
- [`sprints/*.md`](./sprints/) — active sprints; they pick rows from
  here into stories.
- `../NN-topic/README.md` (end-of-file `## Open gaps & sprint-pickable
  tasks` section) — the source of truth for that topic's gaps. This
  file mirrors them.

**Legend.** Priority: P0 must · P1 should · P2 nice-to-have (MoSCoW).
Effort: S ≤1d · M 2–4d · L 5–10d · XL >10d. Status: 📋 open · 🟡
in-progress · ✅ done · ❌ dropped.

**ID convention:** `G-<topic-number>-<sequence>` (e.g. `G-04-01` for the
first email gap, `G-14-03` for the third data-ops gap). The topic number
makes the source unambiguous from the ID alone.

---

## Open gaps

| ID | Topic | Gap | Priority | Effort | Status | Sprint |
|----|-------|-----|----------|--------|--------|--------|
| —  | —     | (none yet — first per-topic deep-dive will populate) | — | — | — | — |

---

## Done / dropped (kept for history)

| ID | Topic | Gap | Resolution | Sprint | Date |
|----|-------|-----|------------|--------|------|
| —  | —     | —   | —          | —      | —    |

---

## How to add a gap

When working on a topic and you discover a gap (something the topic
doesn't yet cover, an unclear pattern, a missing reference, an idea
that's "later"), follow this two-step add:

### Step 1 — Add to the topic doc (in context)

At the **end of the topic's `README.md`**, add or extend:

```markdown
## Open gaps & sprint-pickable tasks

- **G-NN-01** — <one-line gap description>. Effort: S/M/L. Priority: P0/P1/P2.
- **G-NN-02** — <one-line gap description>. Effort: M. Priority: P1.
```

Where `NN` is the topic number. Sequence within the topic.

### Step 2 — Add the same item to this file

Insert a new row in `## Open gaps` above. The Gap column should be the
same one-line description as in the topic doc. The two must stay in sync.

### When the gap moves into a sprint

- Set `Sprint` column to the sprint identifier (e.g. `2026-W21`).
- Set `Status` to 🟡.
- Mention the gap ID in the sprint file's relevant story.

### When the gap is done

- Status → ✅.
- Move the row to `## Done / dropped` with resolution + date.
- The topic doc's `## Open gaps` section still keeps the item but
  marked done (or remove it if the topic doc is being rewritten and
  the gap is no longer relevant in context).

### When the gap is dropped (no longer relevant)

- Status → ❌.
- Move to `## Done / dropped` with `Resolution = dropped: <reason>`.

---

## Sync rule (per `../CLAUDE.md`)

A topic-doc edit that adds, removes, or changes a gap **MUST** update
this file in the same session. If the two drift, sprint planning
fragments. The CLAUDE.md self-update checklist enforces this.
