---
name: accessibility
description: Keyboard navigation, focus states, contrast thresholds, and semantic HTML + ARIA rules.
---
# accessibility.md

## Purpose

Ensure the application is usable by everyone, including people with visual, auditory, motor, or cognitive impairments, by adhering to WCAG 2.1 standards.

---

## Conventions

### Keyboard Navigation

- All interactive elements must be reachable via `Tab`.
- Use logical tab order (top-to-bottom, left-to-right).
- Implement spatial navigation for complex widgets (e.g., arrow keys for tabs or menus).
- Ensure `Enter` and `Space` act as click triggers for buttons.

### Focus States

- Never disable the default focus ring without providing a high-contrast custom alternative.
- Focus indicators must be clearly visible and have a contrast ratio of at least 3:1.
- Use "focus-visible" to only show rings for keyboard users if desired.

### Semantic HTML + ARIA

- Use the right tag for the job: `<button>` for actions, `<a>` for navigation, `<nav>` for menus.
- Use ARIA labels (`aria-label`) for icon-only buttons.
- Use ARIA states (`aria-expanded`, `aria-selected`, `aria-hidden`) to communicate UI state changes.
- Ensure images have descriptive `alt` text (or `alt=""` for decorative ones).

### Contrast Thresholds

- Text must meet a minimum contrast ratio of 4.5:1 (Level AA).
- Large text (18pt+) must meet 3:1.
- UI components and graphical objects must meet 3:1.

### Touch Targets (Mobile & Touch)

- Ensure all interactive elements (buttons, links, inputs) are large enough to be comfortably clicked.
- Minimum size: **44x44 pixels**.
- Provide sufficient spacing between adjacent targets to prevent accidental clicks.

---

## Anti-Patterns

- Never use `div` or `span` for buttons or links without full ARIA and keyboard handling.
- Never use `tabindex` greater than `0`.
- Never rely on color alone to convey meaning (e.g., use an icon or text for errors).
- Never use auto-playing media without a way to pause/stop it.
- Never hide focus rings globally.

---

## Ready-to-Use Prompt

```
Task: Audit/Build [component name] for Accessibility
Skill: frontend/accessibility, frontend/ui-conventions

REQUIREMENTS:
- Element: [e.g., Form, Dropdown, Navigation]
- Compliance: WCAG 2.1 AA
- Navigation: Full keyboard support

CONSTRAINTS:
- Use semantic HTML tags
- Provide visible focus indicators
- Add ARIA labels/states where needed
- Verify contrast ratios

DONE WHEN:
- Component is fully navigable via keyboard only
- Screen reader announces the correct labels and states
- Contrast ratios pass AA standards
```

