---
name: responsive-layout
description: Breakpoint strategy, container widths, and mobile-first spacing/layout rules.
---
# responsive-layout.md

## Purpose

Define a systematic approach to multi-device support, ensuring the UI remains functional, readable, and aesthetic across all screen sizes.

---

## Conventions

### Breakpoint Strategy

- **Mobile First**: Write base styles for small screens, then use `min-width` queries for larger ones.
- **Standard Breakpoints**:
  - `sm`: 640px (Mobile Landscape)
  - `md`: 768px (Tablets)
  - `lg`: 1024px (Laptops)
  - `xl`: 1280px (Desktops)
  - `2xl`: 1536px (Large Monitors)

### Container Widths

- Use max-width containers (`max-w-7xl`, etc.) to prevent content from stretching too far on wide screens.
- Standardize horizontal padding (e.g., `px-4` on mobile, `px-8` on desktop).
- Use `auto` margins to center layouts.

### Spacing and Layout Rules

- **Flex/Grid**: Use CSS Grid for page layouts and Flexbox for component-level alignment.
- **Typography**: Scale font sizes using breakpoints (e.g., `text-xl` on mobile, `text-3xl` on desktop).
- **Touch Targets**: Ensure buttons are at least 44x44px on mobile devices.

---

## Anti-Patterns

- Never use fixed pixel widths (`width: 800px`) for containers.
- Never use `max-width` media queries (e.g., `@media (max-width: 600px)`) as the primary logic.
- Never hide content on mobile that is critical for the task just to "save space."
- Never force horizontal scrolling on any device.
- Never use "magic numbers" for positioning that break when content changes.

---

## Ready-to-Use Prompt

```
Task: Make [component/page] Responsive
Skill: frontend/responsive-layout, frontend/web-ui

REQUIREMENTS:
- Target: [e.g., Dashboard Sidebar, Pricing Table]
- Strategy: Mobile-first
- Behavior: Stack on mobile, grid on desktop

CONSTRAINTS:
- Use Tailwind responsive prefixes (md:, lg:)
- Maintain container max-widths
- Adjust typography scale for smaller screens
- Ensure touch targets are mobile-friendly

DONE WHEN:
- Layout is fluid and doesn't break at any width
- No horizontal scrollbars appear
- Content is readable on a 320px screen
```

