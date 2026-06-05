# agile/

A reusable starting point for the **product-management layer** of a SaaS build:
user stories, roles & access control, and the development process around them.

Where [`../tasks/`](../tasks/) answers *"what's left to do"* (gap-driven sprint
planning) and the topic folders answer *"how do I build X"*, this folder answers
**"what are we building, for whom, and in what order"** — the backlog discipline.

## Files in this folder

- [`user-story-template.md`](./user-story-template.md) — the user-story format,
  fields, status legend, and INVEST checklist. Copy a block per story.
- [`backlog-template.md`](./backlog-template.md) — how to organize a backlog
  (epics / modules), priorities, MVP flagging; a fillable skeleton.
- [`roles-and-rbac.template.md`](./roles-and-rbac.template.md) — role hierarchy
  template + a role × module × action permission matrix to fill in.
- [`process/sdlc-phases.md`](./process/sdlc-phases.md) — the six SDLC phases
  (who / what / which docs) as a generic reference.
- [`process/ba-po-methodology.md`](./process/ba-po-methodology.md) — BA vs PO,
  Product Backlog, User Story, MVP — the methodology in brief.
- [`examples/industrial-shift-log.md`](./examples/industrial-shift-log.md) — a
  **worked example**: a real (anonymized) backlog excerpt from an industrial
  shift-log SaaS, so you can see filled-in stories, modules, and RBAC.

## How it connects

```
BUSINESS NEED ─► USER STORY (agile/) ─► GAP (tasks/GAPS.md) ─► SPRINT STORY (tasks/sprints/) ─► CODE
                      ▲                        ▲
               roles-and-rbac           per-topic README "Open gaps"
```

- A **user story** is the *what & why* (user's view, with acceptance criteria).
  It lives here, in the backlog.
- A **gap** (`G-NN-NN`) is a doc/implementation hole, mirrored in the topic's
  `README.md` and in [`../tasks/GAPS.md`](../tasks/GAPS.md).
- A **sprint story** is the concrete picked task in [`../tasks/sprints/`](../tasks/sprints/).

> A story and a gap are **not** 1:1 — one story can touch several gaps (auth +
> data model + UI), and one gap can serve several stories. Keep the backlog
> (user value) and the gap list (engineering work) as separate but linked views.

## Using this in a new project

1. Copy this `agile/` folder into your repo.
2. Replace [`roles-and-rbac.template.md`](./roles-and-rbac.template.md) with your
   product's actual roles; fill the permission matrix.
3. Start a `backlog/` from [`backlog-template.md`](./backlog-template.md), one
   file per epic/module; write stories with [`user-story-template.md`](./user-story-template.md).
4. As stories reveal engineering work, file gaps in [`../tasks/GAPS.md`](../tasks/GAPS.md)
   and pick them into sprints.
5. Keep the markdown the **master** — if stories start life in a spreadsheet,
   convert once and maintain here (git-diffable, linkable, reviewable).

> See [`examples/industrial-shift-log.md`](./examples/industrial-shift-log.md)
> for what a filled-in backlog looks like in practice.
