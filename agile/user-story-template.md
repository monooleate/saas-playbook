# User story template

Copy one block per story into your backlog files. Keep stories small, testable,
and written from the **user's** point of view — not the implementation's.

## Format

```
### US-<EPIC>-<NNN> — <short title> · *<role>* · `<priority>` · MVP: <Yes/No> · Status: <status>

As a <role>, I want <capability>, so that <business/user outcome>.

**Acceptance criteria**
- [ ] <verifiable condition 1>
- [ ] <verifiable condition 2>
- [ ] <verifiable condition 3>

**Frontend:** <UI-side tasks, if useful>
**Backend:** <API/DB-side tasks, if useful>
**Related:** <US-… ids>   ·   **Gaps:** <G-NN-NN ids>
_Notes:_ <constraints, edge cases, open questions>
```

## Fields

| Field | Purpose |
|---|---|
| **ID** | `US-<EPIC>-<NNN>` — stable, referenceable (e.g. `US-AUTH-003`). Epic prefix groups stories. |
| **Title** | 3–6 words, the capability. |
| **Role** | One of the defined roles → [`roles-and-rbac.template.md`](./roles-and-rbac.template.md). |
| **Priority** | `High` / `Medium` / `Low` (or P0/P1/P2). What breaks if it's missing? |
| **MVP?** | `Yes` / `No` — is it in the first shippable slice? |
| **Status** | see legend below. |
| **Acceptance criteria** | The definition of done, as checkable conditions. The contract with QA. |
| **FE / BE tasks** | Optional split; keeps the story self-contained for estimation. Don't duplicate the same task across stories. |
| **Related / Gaps** | Links to sibling stories and to engineering gaps (`tasks/GAPS.md`). |

## Status legend

| Status | Meaning |
|---|---|
| **Draft** | Being written; wording may change. |
| **In development** | Engineers are working on it. |
| **Accepted** | Finalized, ready for development; team approved, scheduled. |
| **Blocked** | Waiting on another feature or on more information. |
| **Done** | Shipped and verified. |

## INVEST check (write good stories)

- **I**ndependent — minimal coupling to other stories.
- **N**egotiable — describes intent, not a rigid contract; details emerge in refinement.
- **V**aluable — delivers value to a user or buyer; if you can't name the value, it's a task, not a story.
- **E**stimable — the team can size it (split if not).
- **S**mall — fits comfortably in a sprint.
- **T**estable — the acceptance criteria are verifiable.

## Example (filled)

```
### US-AUTH-003 — Reset password · *Member* · `High` · MVP: Yes · Status: Accepted

As a member, I want to reset my password from the login screen, so that I can
regain access without contacting support.

**Acceptance criteria**
- [ ] "Forgot password" link on the login page sends a reset email.
- [ ] Reset link expires after 60 minutes and is single-use.
- [ ] On success the user is signed in and the old password no longer works.
- [ ] Rate-limited to N requests per email per hour.

**Frontend:** request form + reset form; inline validation.
**Backend:** token issue/verify; email send; password hash update.
**Related:** US-AUTH-001   ·   **Gaps:** G-04-02 (email), G-02-04 (session)
_Notes:_ reuse the transactional-email adapter; don't leak whether an email exists.
```

> A longer, real, multi-module example: [`examples/industrial-shift-log.md`](./examples/industrial-shift-log.md).
