# 21 — Brand & Design System — Checklist

> **Status:** Outline.

## Brand

- Product name decided.
- Domain owned.
- Logo (or wordmark) — one SVG, commits forever.
- Favicon set (16/32/180/512px) generated.
- One-sentence tagline.

## Design tokens

- Color tokens defined; primary palette has 9 steps.
- Typography scale with at most 6 sizes.
- Spacing scale (4px or 8px base).
- Radius, shadow, motion tokens.
- Light + dark mode by swapping token values.

## Components

- 10–15 component primitives implemented (Button, Input, Card, Modal,
  Toast, Dropdown, Table, Tooltip, Avatar, Badge, Tabs, Switch,
  Checkbox, Radio).
- Each accessible by default (Radix primitives or equivalent).
- Storybook or Ladle to preview (optional but useful).

## Marketing-app consistency

- Marketing site uses the same tokens.
- Logo + wordmark consistent.
- Colors match.

## Operational

- Token rename procedure documented.
- "When to add a new component" decision rule.
- Lint rule: no inline colors / spacing.

(Each expanded in v0.2.)
