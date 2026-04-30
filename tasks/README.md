# tasks/

The single source of truth for *what's left to do* on the playbook,
shaped so you can plan sprints from it.

## Files in this folder

- [`MATRIX.md`](./MATRIX.md) — **start here**. One row per topic, with
  remaining work, effort estimate, priority (P0/P1/P2), and
  dependencies. This is the file you sort and slice from to plan a
  sprint.
- [`sprint-template.md`](./sprint-template.md) — copy this when
  starting a new sprint planning doc.
- [`sprints/`](./sprints/) — actual sprint plans live here, one
  markdown file per sprint (e.g. `sprints/2026-W19-billing.md`).
  Empty until you start.

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

`MATRIX.md` rows are *topic-level*. Each row points at concrete files
to fill in (the topic's `checklist.md`, `best-practices.md`, etc.).
That granularity is enough for sprint pickup; finer granularity goes
into the sprint plan itself.
