# 11 — Security — Gotchas

> **Status:** Outline.

## Categories

1. **Secret committed to git** — `.env.production` pushed by accident.
   Found by an automated scanner within an hour. Rotate immediately.
2. **API endpoint trusts `tenant_id` from request body** — IDOR.
   Customer A passes Customer B's ID, gets B's data. Fix: tenant from
   session only.
3. **CORS too permissive** — `Access-Control-Allow-Origin: *` lets any
   site call your API on behalf of users.
4. **Stripe webhook endpoint not signature-verified** (cross-link to
   topic 03 G-03).
5. **2FA bypass via password reset** — reset flow doesn't require 2FA.
6. **NoSQL injection via JSON** — `{ $gt: '' }` matches everything.
   Validate input shape.
7. **SSRF via user-supplied URL** — webhook subscriber, image URL,
   OAuth callback — attacker points at internal services.
8. **Rate limit by IP only** — distributed attackers (botnets) bypass
   easily. Combine with email + IP.
9. **CSRF on POST without SameSite cookies** — modern frameworks set
   `SameSite=Lax` by default, but legacy APIs may not.
10. **Public Supabase anon key with permissive RLS** — anon key + open
    policy = public DB.

(Real incidents in v0.2.)
