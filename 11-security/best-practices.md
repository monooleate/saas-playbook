# 11 — Security — Best Practices

> **Status:** Outline. Sources identified.

## Sources

- OWASP top 10: [owasp.org/Top10](https://owasp.org/Top10/).
- OWASP Cheat Sheets.
- Mozilla Web Security Guidelines.
- Cloudflare's WAF rule explanations.

## Practices to expand

- Defense in depth: framework defaults + RLS + rate limit + WAF.
- Trust no input: validate at every boundary.
- Use signed URLs for any public asset that should expire.
- Don't roll your own crypto. Use libraries (`crypto.subtle`, libsodium).
- Log security-relevant events (login attempt, privilege change).
- Have I Been Pwned API to check if a user's password is in a leak.
