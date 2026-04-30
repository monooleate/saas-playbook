# 04 — Email — Example: Remix + PlanetScale + Postmark

> **Status:** Outline. Full implementation forthcoming.

Postmark is the better choice when deliverability is paramount (legal,
financial, healthcare-adjacent SaaS). Pattern: server-side `loader`/
`action` calls a `sendEmail()` helper that logs to PlanetScale and
posts to Postmark's `/email` endpoint.

References: [postmarkapp.com/developer](https://postmarkapp.com/developer).
