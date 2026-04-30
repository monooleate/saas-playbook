# 22 — Accessibility — Gotchas

> **Status:** Outline.

## Categories

1. **`<div onClick>` instead of `<button>`.** Not keyboard reachable;
   not announced as button.
2. **`outline: none` without focus replacement.** Keyboard users
   can't see where they are.
3. **Modal doesn't trap focus.** Tab leaves to underlying page.
4. **Toast dismisses by hover only.** Keyboard users can't dismiss.
5. **Color contrast just below threshold.** Looks fine to designer's
   monitor; fails axe.
6. **Form errors shown but not announced.** Screen reader misses
   them; user can't submit.
7. **Animation plays even with `prefers-reduced-motion`.** Triggers
   vestibular issues.
8. **Decorative SVG with `aria-label`.** Screen reader announces
   meaningless name.
9. **PDF reports inaccessible** — separate problem if you generate
   PDFs (cross-link to topic 08).

(Real incidents in v0.2.)
