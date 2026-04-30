# 08 — Analytics & Reporting — Gotchas

> **Status:** Outline.

## Categories

1. **Event spam after a deploy.** New code path fires an event in a
   loop. PostHog quota burns; bills surprise.
2. **PII in event properties.** `email`, `full_name` end up in
   PostHog. GDPR violation; must purge.
3. **Anonymous → identified user split.** Same person has two profiles
   in PostHog because identity wasn't reconciled at signup.
4. **CSV export OOMs the server.** Streaming export = good; loading
   500k rows in memory = bad.
5. **PDF generation timeout.** WeasyPrint / Puppeteer takes 30s for a
   60-page report; serverless function times out.
6. **MRR formula off-by-one.** Annual subs not divided by 12; addons
   not summed.

(Real incidents in v0.2.)
