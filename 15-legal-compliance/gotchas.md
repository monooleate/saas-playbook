# 15 — Legal & Compliance — Gotchas

> **Status:** Outline.

## Categories

1. **Cookie banner blocks all cookies including session.** User can't
   log in. Fix: separate "essential" (always on) from "non-essential"
   (require consent).
2. **GDPR delete didn't propagate to backups.** Backups are excluded
   from deletion by GDPR allowance, but you must document the
   retention period and final-purge cycle.
3. **Subprocessor added without notification.** Some contracts require
   30-day notice.
4. **ToS update silently changes pricing terms.** Need re-consent.
5. **Privacy Policy version history not preserved.** Forensic
   challenge: which version did the user agree to?
6. **`/terms` and `/privacy` routes serve different versions on
   different deploys.** Cache the rendered HTML; serve from a stable
   URL per version.

(Real incidents in v0.2.)
