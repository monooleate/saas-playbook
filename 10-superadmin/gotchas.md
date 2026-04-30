# 10 — Superadmin — Gotchas

> **Status:** Outline.

## Categories

1. **Impersonation banner can be dismissed.** Support staff peek at
   customer data with no trail. Fix: banner is non-dismissible.
2. **Superadmin SQL leaks tenant data.** Founder runs `SELECT * FROM
   bookings` and accidentally screenshots customer PII for support
   reply. Fix: read-only views with masking; never raw SQL in chat.
3. **Audit log table grows unbounded.** Solution: partitioned + archive
   to cold storage > 1 year.
4. **Plan override without billing change.** Tenant gets premium
   features but isn't billed. Fix: only `is_active` toggles via
   superadmin; billing always goes through topic 03 flow.
5. **Locked out of own superadmin.** Single admin account; password
   forgotten; no recovery path. Fix: two accounts always.
6. **Superadmin URL discovered via robots.txt or sitemap.** Fix:
   exclude from sitemap; `noindex` header; basic-auth on the route
   group.

(Real incidents in v0.2.)
