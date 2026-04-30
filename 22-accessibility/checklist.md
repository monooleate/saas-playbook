# 22 — Accessibility — Checklist

> **Status:** Outline.

## Foundation

- HTML uses semantic elements (`<button>`, `<nav>`, `<main>`,
  `<header>`, `<footer>`, `<h1>`–`<h6>`).
- Heading hierarchy is logical (no `h1` skipped to `h3`).
- Page has one `<h1>`.
- Page has a `<main>` landmark.
- "Skip to content" link first focusable element.
- `<html lang="...">` set.

## Interaction

- Every interactive element reachable by Tab.
- Visible focus ring (don't `outline: none` without replacement).
- Logical tab order matches visual order.
- Modals trap focus; ESC closes.
- Tooltips dismissable with ESC.
- No keyboard traps anywhere else.

## Forms

- Every `<input>` has an associated `<label>`.
- Errors associated via `aria-describedby`.
- `autocomplete` hints (`email`, `current-password`, etc.).
- Required fields marked with `aria-required`.

## Visual

- Color contrast: 4.5:1 body, 3:1 large + UI components.
- Don't convey info by color alone.
- Respect `prefers-reduced-motion` for animations.
- Text resizes to 200% without horizontal scroll.

## Media

- Images have `alt` attributes (empty for decorative).
- Videos have captions.
- Icons used as buttons have `aria-label`.

## Automated

- axe-core run in CI; fails build on known issues.
- Lighthouse a11y score 90+ on key pages.

## Manual

- Screen reader walkthrough quarterly (VoiceOver / NVDA).
- Keyboard-only walkthrough quarterly.

(Each expanded in v0.2.)
