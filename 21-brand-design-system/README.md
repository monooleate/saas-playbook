# 21 — Brand & Design System

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Before any production UI work.
> **Cost:** 3–7 days for v1; iterate forever.
> **Belongs to:** *Foundations*.

## Scope

The minimum visual + identity foundation a solo SaaS needs:

1. **Brand identity** — name, logo, wordmark, tagline.
2. **Color tokens** — primary, secondary, neutrals, semantic
   (success/warning/danger), shades.
3. **Typography scale** — heading sizes, body, mono. Font choice.
4. **Spacing scale** — 4px or 8px base.
5. **Radius, shadow, motion tokens** — consistent across components.
6. **Component primitives** — Button, Input, Card, Modal, Toast,
   Tooltip, etc.
7. **Icon system** — Lucide, Heroicons, Phosphor — pick one.
8. **Light + dark mode** — token-driven, not duplicated.
9. **Accessibility built-in** — color contrast, focus rings,
   keyboard support (cross-link to topic 22).

## Why it matters

Solo founders ship without a design system, then redesign every
component every time. By month 6 the codebase has 7 button variants
and 12 shades of grey. The cleanup cost compounds.

A design system is not a Figma file. It's tokens in code, components
in code, and one source of truth for "what does our product look like."

## Recommended stack

- **shadcn/ui** for React (Next.js, Remix). Owned components, not a
  dependency. The closest thing to a community standard.
- **shadcn-svelte** for SvelteKit.
- **Tailwind CSS** for the token layer. Define tokens in
  `tailwind.config.ts` once, use everywhere.
- **Radix UI** primitives (used by shadcn) for accessibility.
- **Lucide** for icons.

For non-developers / fast prototyping: **Framer** or **Webflow** for
the marketing site visuals, then port the design tokens into code.

## Tokens before components

The right order:

1. Define color tokens (e.g. `--color-primary-500`, `--color-bg`).
2. Define spacing, radius, shadow, motion tokens.
3. Define typography tokens.
4. Build components using tokens — never use raw colors.
5. Build pages using components — never use raw HTML primitives where
   a component exists.

This makes design changes cheap. Renaming `--color-primary-500` =
one PR. Going dark mode = swap token values, no component changes.

## What goes in v0.2

- `tailwind.config.ts` token reference.
- shadcn/ui setup walkthrough.
- Logo design tips for solo founders (wordmark > illustration).
- Dark mode token strategy.
- Component contribution rules.
- Design / dev handoff (when there's no designer).

## Sources

- shadcn/ui docs: [ui.shadcn.com](https://ui.shadcn.com).
- Radix UI: [radix-ui.com](https://www.radix-ui.com).
- Tailwind: [tailwindcss.com](https://tailwindcss.com).
- Refactoring UI (Adam Wathan, Steve Schoger) — best book for solo
  developer-as-designer.
- Linear's design system writeups.
- The Tailwind UI / Catalyst components for inspiration.
