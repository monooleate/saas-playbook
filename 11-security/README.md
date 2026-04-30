# 11 — Security

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Day 1, then continuously.

## Scope

The security topic covers what other topics defer to it:

1. **Threat model** — what the realistic attackers want from a 1-person
   SaaS, and what doesn't matter.
2. **Secrets management** — `.env` discipline, password manager,
   rotation cadence.
3. **Dependency scanning** — Dependabot, Renovate, manual reviews.
4. **OWASP top-10 self-audit** — the 10 most common web vulns, what
   to check for in your code.
5. **CSRF / XSS / SQL injection** — framework defaults usually handle
   these, but verifying is the topic's job.
6. **Rate limiting** — at the proxy and at the app.
7. **Brute-force defense** — fail2ban (topic 01 has the install
   recipe), in-app login throttling.
8. **Public asset access control** — signed URLs, expiring tokens.
9. **Data isolation verification** — tenant A cannot read tenant B's
   data. Same as topic 02 but tested, repeatedly.
10. **Incident response** — what to do when a breach is suspected.

## Threat model for solo SaaS

The realistic attackers, in priority order:

1. **Drive-by automated scanners** — most attacks. Find an exposed
   `.git`, a leaked API key, an unpatched WordPress. Defense: standard
   hygiene (don't commit secrets, keep deps updated, don't run
   WordPress on the same VPS as your SaaS).
2. **A disgruntled or curious customer** — tries to access another
   tenant's data via URL guessing or API parameter tampering. Defense:
   tenant isolation + RLS (topic 02).
3. **Credential stuffing** — leaked-password lists used against your
   login. Defense: rate limit + 2FA for owners + Have-I-Been-Pwned
   integration.
4. **Targeted attacker** — you're a popular SaaS with valuable customer
   data. Defense: full security review by someone external, beyond
   playbook scope.

## What goes in v0.2

- Full OWASP top-10 walkthrough with framework-specific defaults.
- Secrets rotation procedure.
- Dependabot / Renovate config.
- Rate limiting patterns: at Caddy/Cloudflare vs in-app.
- Security headers checklist (CSP, HSTS, X-Frame-Options, etc.).
- Incident response template.

## Sources

- OWASP top 10: [owasp.org/Top10](https://owasp.org/Top10/).
- OWASP Cheat Sheets: [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/).
- Mozilla Web Security Guidelines:
  [infosec.mozilla.org/guidelines/web_security](https://infosec.mozilla.org/guidelines/web_security).
- Have I Been Pwned API:
  [haveibeenpwned.com/API/v3](https://haveibeenpwned.com/API/v3).
