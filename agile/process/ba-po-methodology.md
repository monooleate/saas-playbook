# BA / PO methodology — backlog, user stories, MVP

A brief on the product-management roles and artifacts behind a backlog. For a solo
founder, "BA" and "PO" are both you — but knowing the split helps you switch hats
deliberately.

## Product Owner vs Business Analyst

| | Product Owner (PO) | Business Analyst (BA) |
|---|---|---|
| Primary role | Owns product vision & backlog priority | Gathers/analyzes business requirements |
| Focus | Product, features, backlog | Processes, data flow, requirements |
| Decisions | Owns product decisions | Recommends; doesn't own the final call |
| Goal | Ship a product that meets user & business needs | Translate needs into accurate system requirements |

### Combine or split?

- **Combine** (one person) when: small team / MVP / startup; fast, flexible
  decision-making; the PO has strong analysis skills; the dev team understands the
  domain well.
- **Split** when: large/enterprise scope; complex requirements; many stakeholders;
  frequently changing requirements that need dedicated analysis.
- **Scrum Master** pays off mainly in larger, multi-team, long-running agile work.
  A small team can run the backlog without one.

## Product Backlog

A prioritized list of everything the product needs; owned by the PO; the single
source of work for the team.

- **Dynamic** — items added/removed, priorities change.
- **Ordered by priority** — highest value on top.
- **All work types** — user stories, features, bugs, tech debt, research.
- **Progressively refined** — top items detailed, lower items coarse.
- **Drives sprint planning** — the team pulls from the top.

**Item types:** User Story · Feature · Bug · Technical Debt · Spike (research/prototype).

> **Product Backlog ≠ Sprint Backlog.** The sprint backlog is a subset selected for
> one sprint, owned by the dev team.

## User Story

```
As a <role>, I want <capability>, so that <business/user outcome>.
```

Plus **acceptance criteria** (when it's done). A story captures the *business
context*, not the technical detail — that belongs in the functional spec / topic
docs. Format and INVEST checklist → [`../user-story-template.md`](../user-story-template.md).

## MVP

The first minimal-but-valuable version: only the technical detail needed for the
core functions; no future features. An MVP doc states:

- **Goal** — why the MVP, what problem it solves.
- **Must-have functions** — the minimum to be viable.
- **Explicitly out of scope** — what waits for later versions.
- **User flows** — the main paths through the MVP.
- **Validation plan** — how you'll measure success.

> Keep the MVP scope as an explicit, dated decision. The set of `MVP: Yes` stories
> in the backlog *is* your MVP definition — revisit it as you learn.
