# 10 — Example: Next.js + Prisma

> **Status:** Outline.

`/admin/*` (or any chosen path) protected by middleware checking
`session.user.isSuperadmin`. UI built with Server Components + Server
Actions for mutations. Audit log via Prisma `AuditLog` model written
in a `withAudit` wrapper around mutations.
