# 10 — Superadmin — Best Practices

> **Status:** Outline.

## Practices to expand

- Don't share superadmin accounts. One per human.
- 2FA on every superadmin account.
- Banner on impersonation, always. No silent peek.
- Audit log immutable (append-only, no UPDATE, no DELETE).
- Backup superadmin path: a CLI tool you can run when the web UI is
  broken (you broke a deploy, can't get into the admin to fix it).
- Don't expose superadmin URLs to public crawlers; `noindex` headers.

## Sources

(Mostly internal tools across the industry are not public; the
playbook will cite GitLab's open-source admin panel as a reference
implementation.)
