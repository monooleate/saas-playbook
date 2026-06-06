# SOURCES — source-consumption ledger

> **What this is.** A central ledger that tracks whether every piece of
> *ingested source material* — founder notes, interview transcripts,
> spreadsheet exports, a prior business plan, a competitor teardown — has
> actually been **mined into the topic docs and gaps**, or deliberately
> set aside as obsolete.

## Why it exists

`MATRIX.md` and `GAPS.md` track **output coverage** — which topics are
written, which gap-tasks remain. Neither tracks **input coverage**:
*have we squeezed everything out of the raw material we were handed?*

That gap is where work silently falls through. A founder pastes in a
15-page note; three pages get turned into gaps; the rest is forgotten
because nothing records that it was only *partially* consumed. Six weeks
later the topic docs all look "done" and the unused pages are invisible.

This ledger closes the loop. **As long as a row here is `⚪` or `🟡`
with remainder, a source is not yet exhausted — regardless of how
"finished" the topics look.** It's the input-side companion to the
output-side coverage tracker.

**Pair this with:**

- [`MATRIX.md`](./MATRIX.md) — topic-level output coverage.
- [`GAPS.md`](./GAPS.md) — item-level output backlog.
- This file — **input** coverage (sources → topics/gaps).

## Status legend

| Mark | Meaning |
|------|---------|
| 🟢 | **fully mined** — all relevant content lifted into topic docs / gaps |
| 🟡 | **partially mined** — some lifted, remainder identified (see "Remaining") |
| ⚪ | **not yet** — ingested/identified, but processing still pending |
| ⚫ | **historical / obsolete** — intentionally NOT a decision source (kept for provenance) |

**Target state:** every row is 🟢 or ⚫. Any `⚪` — or 🟡 with a non-empty
"Remaining" — means there is still source material to process.

---

## Ledger

> One row per source (or per section/page of a large source). Link the
> target topic + gap IDs so the trace is bidirectional.

| Source | Type | Status | Mined into (topic / `G-NN-NN`) | Remaining |
|--------|------|--------|-------------------------------|-----------|
| `<path/to/source>` | note / transcript / export / plan | ⚪/🟡/🟢/⚫ | `<NN-topic>` / `G-NN-NN` | `<what's left, or — >` |

For a large multi-section source (a long note, a multi-tab spreadsheet),
add a per-section sub-table so partial consumption is visible at the
right granularity:

```markdown
### <source name> (<N sections/pages>)

| Section | Layer | Content | Status | Target / gap |
|---------|-------|---------|--------|--------------|
| p.1–3   | theory | generic SDLC notes | ⚫ | framework-duplicate, not split out |
| p.10    | domain | event taxonomy | 🟢 | 14-data-ops / G-14-08 |
```

The "Layer" column is optional but useful when a source mixes *reusable
domain knowledge* with *obsolete early decisions* (e.g. an old stack
choice) — mark the obsolete layer ⚫ so it is never mistaken for a
live decision.

---

## Workflow

- **When you ingest a new source:** add a row (status `⚪`), so it can't
  be forgotten.
- **When you lift content from it** into a topic doc / gap: move `⚪`→`🟡`
  (note the remainder) or `⚪`→`🟢`, and record the target topic + gap ID.
- **When a source is obsolete** (superseded stack, old pricing, replaced
  by a richer artifact): mark it `⚫` with a one-line reason — do not
  delete the row; the provenance matters.
- **At sprint planning:** scan for `⚪`/🟡 rows. Unexhausted sources are
  candidate inputs before you go hunting for new ones.

The matrix philosophy (see [`../CLAUDE.md`](../CLAUDE.md)) gives each
topic three *outputs*. This ledger guarantees the *inputs* that feed
those outputs are fully spent.
