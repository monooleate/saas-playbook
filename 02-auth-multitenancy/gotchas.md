# 02 — Auth & Multi-tenancy — Gotchas

> **Status:** Outline. Categories listed; specific incidents to be
> documented in v0.2.

## Categories of failure to be expanded

1. **Tenant ID forgotten in a query** — the most common multi-tenancy
   bug. Symptom: customer reports seeing another company's data. RLS
   prevents this; ORM-only setups need extreme discipline.
2. **Subdomain spoofing via `Host` header** — proxy must forward a
   trusted `Host`, app must validate against allow-list.
3. **Magic link replay** — single-use tokens, but only if you mark them
   used *before* sending the success response.
4. **Session fixation after password reset** — old sessions remain
   valid. Must invalidate all sessions on reset.
5. **OAuth account takeover via email collision** — user signs up with
   email/password, attacker signs up via Google with the same email.
6. **Rate limit on the wrong key** — limiting by IP misses
   distributed attacks; limit by email + IP combined.
7. **Invite token reuse / sharing** — the invite link gets forwarded;
   anyone who clicks it joins the tenant.
8. **2FA bypass via password reset** — reset flow doesn't require 2FA,
   so it bypasses 2FA. Must be patched.
9. **Tenant deletion leaves orphan rows** — cascading deletes must be
   audited.
10. **"Sign in as user" without a banner** — support staff impersonating
    silently. Add a visible, non-dismissible banner.

(Each will be a full incident write-up in v0.2.)
