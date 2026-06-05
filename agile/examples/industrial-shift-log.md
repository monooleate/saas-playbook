# Worked example — industrial shift-log SaaS

A real, lightly anonymized backlog excerpt from a B2B **shift-diary / shift-log**
product for continuous-operation industrial sites (chemical, refining, energy).
Use it to see what filled-in roles, modules, and stories look like — not as a spec
to copy verbatim.

> This example shows the *shape*: a deep role hierarchy, module-grouped stories,
> acceptance criteria, MVP flags, and how a domain quirk (shared role accounts)
> drives the access model.

---

## Roles (filled hierarchy)

```
Site Manager
├── Production Manager
│     └── Plant Group Manager  (a.k.a. Block Manager)
│           ├── Supervisor  (a.k.a. Data Supervisor)
│           └── Shift Leader  (= Shift Supervisor)
│                 ├── Board Operator
│                 └── Outside Operator
├── Maintenance Manager
│     └── Plant Group Maintenance Engineer
└── Operational Excellence Team
```

Plus a cross-cutting **System Administrator** (manages roles, never edits records).

### Permission matrix (excerpt)

| Action ▸ \\ ▾ Role | Site Mgr | Block Mgr / Data Sup | Shift Leader | All users |
|---|---|---|---|---|
| View any shift log | ✓ | ✓ | ✓ | ✓ |
| Edit shift log | ✓ (any, anytime) | ✓ (corrections) | ✓ (active only) | ✕ |
| Define recurring tasks | ✓ | ✓ | ✕ | ✕ (OpEx Team) |
| Manage roles | — System Administrator only — | | | |

> **Domain quirk that shaped the design:** shift leaders **share one role-based
> account** (a common control-room terminal), so login ≠ person. The audit anchor
> is a **name picked from a dropdown** and verified against the shift schedule —
> not the login identity. This pushed identity/attribution into the data model,
> not just auth.

---

## Backlog excerpt (module-grouped stories)

### Module: Basic

#### US-SD-001 — Shift report recording · *Shift Leader* · `High` · MVP: Yes · Status: Draft
As a shift leader, I want to record a structured shift report, so that all critical
shift information is documented consistently for analysis and compliance.
- [ ] Form follows the predefined QA-approved structure (mandatory fields: production data, headcount, technological parameters, incidents).
- [ ] Fields are validated (numbers within range; strings from a controlled list) for comparable, structured data.
- [ ] Reports are stored centrally and available for review; format is identical across shifts.
- [ ] The shift leader can edit a report only within their own shift window.

**Frontend:** modern modular form; modals for validation; scheduled POST on a secure channel.
**Backend:** schema; accept POST, validate, persist.

#### US-SD-002 — Automated shift-diary handoff · *Shift Leader* · `High` · MVP: Yes
As a shift leader, I want the system to auto-close the previous diary and open mine
on duty, so that I can start immediately.
- [ ] New diary loads automatically; an incomplete previous diary does **not** block it.
- [ ] Diaries are editable only within a time window (until the next shift starts).
- [ ] Past the deadline an unfinished diary auto-locks, marked "incomplete".
- [ ] Auto-lock is logged and a "black mark" is recorded against the responsible leader (accountability).

### Module: Tasks

#### US-SD-017 — Assign recurring tasks · *Operational Excellence Team* · `High` · MVP: Yes
As an OpEx team member, I want to define recurring tasks, so that maintenance and
safety procedures happen on schedule.
- [ ] Recurring tasks with fixed intervals (daily/weekly/monthly).
- [ ] System auto-adds them to the relevant shift's task list.
- [ ] Shift leaders cannot modify recurring tasks but must complete them.
- [ ] Each completion is logged; responsible personnel are notified when due.

#### US-SD-018 — Track task status · *Block Manager* · `High` · MVP: Yes
As a block manager, I want real-time task status, so that I can spot delays.
- [ ] Statuses: Not Started / In Progress / Completed; start & finish timestamped.
- [ ] Dashboard summarizes completion rates; filter by date range, type, person.

### Module: Admin panel

#### US-SD-022 — Customize diary layout (drag-and-drop) · *Supervisor* · `Low` · MVP: No
As a supervisor, I want to rearrange which predefined modules appear in the diary,
so that each site sees the layout it needs.
- [ ] Drag-and-drop admin panel reorders **predefined** modules (custom modules not allowed).
- [ ] Mandatory core modules can't be removed; changes apply instantly to shift leaders.
- [ ] All layout changes are logged.

---

## What to copy from this example

- **Group stories by module/epic**, with stable IDs (`US-SD-NNN`).
- **Every story has acceptance criteria** as checkable conditions.
- **MVP flag is honest** — most stories are `No`; the `Yes` set is the first slice.
- **A domain quirk can drive architecture** — surface it in the roles doc, not just
  in one story's notes.
- **Park the contradictions.** The real backlog had an `open-questions.md` for
  things like "is submission mandatory or is auto-save enough?" — decided before
  coding, not guessed.
