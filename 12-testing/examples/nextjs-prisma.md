# 12 — Example: Next.js + Prisma

> **Status:** Outline.

Vitest with `@testing-library/react` for component tests. Playwright
for E2E. Prisma test setup uses a separate test database
(`DATABASE_URL_TEST`). Each test wraps in `prisma.$transaction(async
(tx) => { ... ; throw rollback })` to isolate.
