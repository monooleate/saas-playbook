# Backlog template

How to organize a product backlog so you can plan sprints from it without it
rotting into a wishlist.

## Structure

Split the backlog by **epic / module**, one markdown file per epic under a
`backlog/` folder:

```
agile/
├── README.md
├── roles-and-rbac.md
├── backlog/
│   ├── <epic-a>.md        ← stories US-<A>-001…
│   ├── <epic-b>.md
│   └── <epic-c>.md
└── process/
```

Within an epic file, group stories by **sub-module** with `##` headings, and one
`###` heading per story (see [`user-story-template.md`](./user-story-template.md)).

## Epic file skeleton

```markdown
# Backlog — <Epic name>

> Source: <where these came from>. Roles → ../roles-and-rbac.md.
> Open questions → ../open-questions.md.

## <Sub-module 1>

### US-<EPIC>-001 — <title> · *<role>* · `<priority>` · MVP: <Yes/No> · Status: <status>
As a <role>, I want <capability>, so that <outcome>.
- [ ] <acceptance criterion>
- [ ] <acceptance criterion>

### US-<EPIC>-002 — …

## <Sub-module 2>

### US-<EPIC>-003 — …
```

## Prioritization

- **Priority** (`High`/`Medium`/`Low` or P0/P1/P2) — what breaks without it.
- **MVP flag** (`Yes`/`No`) — is it in the first shippable slice? Be ruthless;
  most stories are `No` for MVP.
- Order within a file roughly by priority, but don't over-sort — the topic
  [`../tasks/MATRIX.md`](../tasks/MATRIX.md) and sprint planning do the real sequencing.

## Keep two views, linked

| View | Lives in | Granularity |
|---|---|---|
| **Product backlog** (user value) | `agile/backlog/` | user stories with acceptance criteria |
| **Engineering gaps** (work to do) | [`../tasks/GAPS.md`](../tasks/GAPS.md) | concrete, estimable tasks (`G-NN-NN`) |

A story points at the gaps it needs; a gap points back at the stories it serves.
Sprint stories ([`../tasks/sprints/`](../tasks/sprints/)) are picked from the gap
list, justified by the user stories.

## Hygiene

- **One source of truth.** If stories start in a spreadsheet, convert once and
  maintain the markdown. Don't keep both live.
- **Park open questions.** Contradictions and undecided scope go in a single
  `open-questions.md`, not buried in story notes — decide them before coding.
- **Don't duplicate tasks across stories** (e.g. the same "mandatory field
  validation" in five stories) — factor shared work into one story or a gap.
- **Stories describe intent, specs describe mechanics.** Keep API shapes,
  schemas, and validation rules in the topic docs, not in the story.
