# 11 — Example: Next.js + Prisma

> **Status:** Outline.

Vercel sets reasonable defaults but doesn't add CSP. Use
`next.config.mjs` `headers()` to add security headers. Auth.js
provides CSRF tokens by default. Prisma queries naturally parameterize
(no SQL injection from string concat) but raw queries (`$queryRaw`)
bypass — use `$queryRawUnsafe` only with sanitized input.
