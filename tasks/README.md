# tasks/

The single source of truth for *what's left to do* on the playbook,
shaped so you can plan sprints from it.

## Files in this folder

- [`MATRIX.md`](./MATRIX.md) — **start here**. One row per topic, with
  remaining work, effort estimate, priority (P0/P1/P2), and
  dependencies. Topic-level rollup; sort and slice from here to scope
  a sprint.
- [`GAPS.md`](./GAPS.md) — **item-level backlog**. One row per concrete
  gap-derived task across all topics; mirror of each topic's
  end-of-`README.md` `## Open gaps & sprint-pickable tasks` section.
  Sprint stories are picked from here.
- [`sprint-template.md`](./sprint-template.md) — copy this when
  starting a new sprint planning doc.
- [`sprints/`](./sprints/) — actual sprint plans live here, one
  markdown file per sprint (e.g. `sprints/2026-W19-billing.md`).

## Workflow

The playbook is meant to drive a real SaaS build. Use it like this:

### Planning a sprint

1. Open [`MATRIX.md`](./MATRIX.md). Filter by:
   - **Lifecycle stage** (Stage 0–4 in `../ROADMAP.md`) — what stage is
     your product in?
   - **Priority** (P0 first).
   - **Dependencies** — pick rows whose dependencies are already done.
2. Pick 3–8 rows totalling ~1–2 weeks of effort.
3. Copy [`sprint-template.md`](./sprint-template.md) to
   `sprints/YYYY-WNN-<theme>.md`.
4. Fill in the sprint goal + concrete user stories per row.
5. Work the sprint. Mark items off in `MATRIX.md` as you go.

### When you ship a topic

1. Update the topic's status in `MATRIX.md` (📋 → 🟡 in-progress → ✅ done).
2. Update the same status in `../README.md` matrix.
3. If new gaps appear inside the topic during work, add them to
   `MATRIX.md` rather than carrying them in your head.
4. At sprint end: short retro at the bottom of the sprint file.

## Why this shape

GitHub Issues is the natural place for granular tasks at scale, but:

- This repo is a knowledge base, not an active product, so issues are
  overhead.
- A single `MATRIX.md` table works as a database when scanned, sorted,
  and grep'd. You can paste rows into a spreadsheet for offline
  prioritization.
- Sprint plans as markdown files live alongside the knowledge they
  reference; cross-links work.

If the project ever moves to GitHub Issues, the MATRIX rows map
1:1 to issues — but most solo founders don't need that overhead.

## Convention

- **Priority:** P0 = must, P1 = should, P2 = nice-to-have.
- **Effort:** S = ≤1 day, M = 2–4 days, L = 5–10 days, XL = >10 days.
- **Status:** ✅ done · 🟡 in-progress · 📋 outline · ❌ blocked.
- **Stage:** matches `../ROADMAP.md` Stage 0–4.

## Granularity

Three levels, by design:

- **`MATRIX.md`** — *topic-level*. One row per topic, one-sentence
  "Remaining work" cell. Use to filter by priority / dependencies.
- **`GAPS.md`** — *item-level*. One row per concrete sprint-pickable
  task (e.g. "DKIM/SPF/DMARC walkthrough", "RLS policy patterns").
  Mirror of the per-topic `## Open gaps & sprint-pickable tasks`
  section at the end of each topic `README.md`.
- **`sprints/YYYY-WNN-*.md`** — *story-level*. Picked from `GAPS.md`
  into stories with acceptance criteria and a daily log.

The matrix philosophy (see [`../CLAUDE.md`](../CLAUDE.md)) is that each
topic produces three outputs: content, gap analysis, sprint-pickable
tasks. `GAPS.md` is where output #3 collects centrally; the topic doc's
end-of-file section is the same items in context.
