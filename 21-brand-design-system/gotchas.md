# 21 — Brand & Design System — Gotchas

> **Status:** Outline.

## Categories

1. **Hex codes scattered everywhere.** No tokens; renaming requires
   global find-replace.
2. **Marketing site looks different from app.** Two designers, two
   token sets.
3. **Dark mode added late.** Components hardcoded for light; full
   audit needed.
4. **Logo file is 5 MB PNG.** Use SVG wordmark.
5. **Custom font kills page speed.** 200kb font; users see fallback
   for 2s.
6. **shadcn/ui copy-paste drifts.** Customizations diverge from
   upstream; updates painful.
7. **Component zoo.** 7 button variants because nobody said no.

(Real incidents in v0.2.)
