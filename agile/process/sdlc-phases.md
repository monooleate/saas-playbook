# SDLC phases

A six-phase reference for how a SaaS feature (or a whole product) moves from idea
to maintenance. Generic and lightweight — adapt roles to your team size. A solo
founder wears most of these hats; the value is the **checklist of concerns**, not
the org chart.

| # | Phase | Who | Key activities | Artifacts |
|---|---|---|---|---|
| 1 | **Idea & business need** | PO/BA, stakeholders, customers | Define the problem & goal; gather needs; market & competitor scan; business model & value proposition | Business plan; stakeholder need list; competitor analysis |
| 2 | **Requirements & design** | PO/BA, UX/UI, architect, lead dev | Functional + non-functional requirements; **user stories**; wireframes & prototypes; system & data model; stack choice; **define MVP** | Functional spec; user stories + acceptance criteria; wireframes; architecture & schema; MVP doc |
| 3 | **Development** | Frontend, backend, DevOps, QA | Sprint planning; API & DB; UI & components; integration tests; CI/CD; keep docs current | API docs; code docs; sprint backlog |
| 4 | **Testing & QA** | QA, test automation, devs | Manual (UI/API/functional); automated (unit/integration); security & performance; UAT | Test plan & log; bug reports |
| 5 | **Deployment & release** | DevOps, ops/support | Deploy to staging; production infra; monitoring & logging; deploy & rollback strategy | Release notes; deployment guide |
| 6 | **Maintenance & iteration** | Support, devs | Support & bug intake; new features & iterations; performance tuning; version updates & scaling | User manual; bug list & roadmap |

## How it maps to this playbook

- Phase 1–2 concerns live in topics `19-discovery-validation`, `20-pricing-strategy`,
  and the **user stories** in [`../backlog-template.md`](../backlog-template.md).
- Phase 3–5 concerns live in the build topics (`01-infrastructure`, `02-auth…`,
  `12-testing`, `14-data-ops`, …) and are scheduled via [`../../tasks/`](../../tasks/).
- Phase 6 maps to `16-support-changelog`, `13-observability`, `08-analytics`.

## Notes for small teams

- You don't need every role — you need every **question answered**. A phase with
  no owner is a phase that silently gets skipped.
- The phases are not strictly sequential. In an iterative build you revisit 2–4
  every sprint; phase 1 is mostly up front, phase 6 is forever.
- **Watch for stale early decisions.** Stack, schema, and scope choices made in
  phase 1–2 go stale fast — date them and re-check before building.
