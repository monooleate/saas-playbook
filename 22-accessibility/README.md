# 22 — Accessibility (a11y)

> **Status:** Outline only. Full content scheduled for v0.2.
> **When to implement:** Built into the design system from day 1.
> **Cost:** Continuous; ~10% extra effort if done from start, 200% if
> retrofitted.

## Scope

The minimum a solo SaaS needs to be usable by people with disabilities
and to clear common legal thresholds:

1. **WCAG 2.2 AA target** — the global de facto standard.
2. **Keyboard navigation** — every action reachable without mouse.
3. **Focus management** — visible focus rings, logical order, focus
   trap in modals.
4. **Screen reader support** — semantic HTML, ARIA where unavoidable,
   live regions.
5. **Color contrast** — 4.5:1 for body text, 3:1 for large + UI.
6. **Reduced motion** — respect `prefers-reduced-motion`.
7. **Form accessibility** — labels, error association, autocomplete
   hints.
8. **Image alt text** — every meaningful image; decorative ones with
   `alt=""`.
9. **Document structure** — landmarks, headings hierarchy.

## Why it matters

- **Ethical:** the web should be usable by everyone.
- **Legal:** EU EAA (European Accessibility Act, in force June 2025),
  US ADA + Section 508. Lawsuits are real and rising; small SaaS gets
  hit too.
- **Practical:** a11y improvements help everyone — keyboard shortcuts
  are loved by power users; high contrast helps in sunlight; semantic
  HTML helps SEO.

## Solo founder approach

Build it in, don't bolt it on. Two practices catch ~80% of issues:

1. **Use accessible primitives.** Radix UI, React Aria, shadcn-based
   components. They handle focus management, ARIA, keyboard.
2. **Run automated checks on every PR.** axe-core via Playwright or a
   CI step. Catches missing labels, contrast issues, ARIA misuse.

The remaining 20% requires manual testing with a screen reader
(VoiceOver on Mac, NVDA free on Windows) and keyboard-only navigation.
Do this for the 5 most-used flows once a quarter.

## What goes in v0.2

- Component-by-component a11y checklist (Button, Input, Modal, etc.).
- Keyboard testing script.
- Screen reader testing playbook.
- Common ARIA patterns and which to use vs. avoid.
- Forms: labels, errors, aria-describedby.
- Color contrast tooling.
- The EU EAA scope and what solo SaaS must comply with.

## Sources

- WCAG 2.2 quick reference:
  [w3.org/WAI/WCAG22/quickref](https://www.w3.org/WAI/WCAG22/quickref/).
- Web Content Accessibility Guidelines authoritative:
  [w3.org/TR/WCAG22](https://www.w3.org/TR/WCAG22/).
- ARIA Authoring Practices Guide (APG):
  [w3.org/WAI/ARIA/apg](https://www.w3.org/WAI/ARIA/apg/).
- axe-core: [github.com/dequelabs/axe-core](https://github.com/dequelabs/axe-core).
- Adrian Roselli's blog (long-form a11y deep dives).
- WebAIM: [webaim.org](https://webaim.org/).
