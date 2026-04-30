# 11 — Security — Checklist

> **Status:** Outline.

## Secrets

- `.env` not committed; verified with `git log -p` search for known
  prefixes (`sk_live`, `whsec_`, `eyJ`).
- Password manager vault for prod secrets.
- Rotation calendar: quarterly for billing/email keys; annually for
  others.

## Dependencies

- Dependabot or Renovate enabled with weekly cadence.
- Direct deps reviewed manually; transitive deps trusted.
- `npm audit` or `pnpm audit` on every CI run; fail on high+.

## Headers (set at proxy or app)

- `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`.
- `Content-Security-Policy` (per app, restrictive).
- `X-Content-Type-Options: nosniff`.
- `X-Frame-Options: DENY` (or CSP `frame-ancestors`).
- `Referrer-Policy: strict-origin-when-cross-origin`.

## Rate limiting

- 5 login attempts per IP per minute.
- 100 requests per IP per minute on `/api/*`.
- Higher limit for authenticated users with valid session.

## Auth & isolation

- 2FA available for users; required for superadmin.
- Tenant isolation tested with two accounts, cross-tenant URL
  manipulation.
- API endpoints accept `tenant_id` from session, never from body/query.

## Operational

- `robots.txt` excludes `/superadmin/*`, `/api/internal/*`.
- Sentry configured (link to topic 13).
- Incident response template in `internal-docs/INCIDENT.md`.

(Each item expanded with verification steps in v0.2.)
