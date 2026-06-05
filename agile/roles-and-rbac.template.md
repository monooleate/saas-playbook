# Roles & RBAC — template

Define your product's roles **once**, here, and reference them from every user
story. The permission matrix this produces is the source for your access-control
implementation (→ topic `02-auth-multitenancy`).

## 1. Role hierarchy

Draw the org/permission tree. Most B2B SaaS has 3–6 roles; don't invent more than
the customer actually distinguishes.

```
<Org admin>
├── <Manager role>
│     └── <Lead / supervisor role>
│           ├── <Operator / member role>
│           └── <Read-only / viewer role>
└── <Support / cross-cutting role>
```

| Role | Responsibility | Reports to | Typical actions |
|---|---|---|---|
| `<admin>` | … | — | manage users, billing, settings |
| `<manager>` | … | `<admin>` | … |
| `<member>` | … | `<manager>` | create/edit own records |
| `<viewer>` | … | — | read-only |

> Tip: separate **who a person is** (identity / login) from **what they may do**
> (role). In some domains many people share one device or role account — then the
> audit anchor is an explicit "acting as" selection, not the login. Decide this
> early; it shapes the audit model.

## 2. Permission matrix (role × action)

Fill one row per role, one column per protected action. Use ✓ / ✕ / conditional
(e.g. "own only", "active record only").

| Action ▸ \\ ▾ Role | `admin` | `manager` | `member` | `viewer` |
|---|---|---|---|---|
| View records | ✓ | ✓ | ✓ | ✓ |
| Create record | ✓ | ✓ | ✓ (own) | ✕ |
| Edit record | ✓ | ✓ (any) | ✓ (own/active) | ✕ |
| Delete record | ✓ | ✓ | ✕ | ✕ |
| Manage users/roles | ✓ | ✕ | ✕ | ✕ |
| Export / report | ✓ | ✓ | ✓ | ✓ (read) |

## 3. Rules to write down

- **Default deny.** Anything not explicitly granted is forbidden.
- **Enforce server-side.** The UI hides buttons; the API/database enforces. In a
  multi-tenant DB, prefer row-level security keyed on tenant + role.
- **Audit privileged actions.** Log who/what/when for every modification and every
  permission change — exportable for compliance.
- **Roles are data, not code.** Avoid `if (user.role === 'admin')` scattered in
  routes; resolve permissions through one policy layer.

> See a filled, real example (10-role industrial hierarchy + matrix) in
> [`examples/industrial-shift-log.md`](./examples/industrial-shift-log.md).
