---
name: design-system
description: Design tokens (color, spacing, radius), component variants, and dark mode rules.
---
# design-system.md

## Purpose

Define the visual and structural foundations of the application to ensure consistency, brand alignment, and development speed through a tokenized design system.

---

## Conventions

### Design Tokens (CSS Variables / Tailwind)

- **Color Purpose (60-30-10 Rule)**: Use a balanced color distribution to create harmony and focus.
  - **60% (Primary)**: Dominant neutral color for backgrounds and large surfaces.
  - **30% (Secondary)**: Supporting color for headers, sidebars, or secondary components.
  - **10% (Accent)**: High-contrast color reserved exclusively for primary actions (CTAs) and vital system feedback.
- **Spacing & Whitespace**: Use a 4px/8px grid system. Whitespace is a functional tool used to group related elements (proximity) and separate sections, giving content room to "breathe."
- **Typography**: Prioritize readability with clear typefaces and comfortable line heights. Limit font weights to avoid visual chaos.
- **Affordance & Shadows**: Use shadows and depth to visually signal function (e.g., elevated surfaces for buttons). Every element must look like what it does.
- **Grid System**: Use a consistent grid as the foundation for alignment and order, ensuring the UI is professional and scalable.

### Component Variants

- **Buttons**: Define `primary`, `secondary`, `outline`, `ghost`, and `danger` states.
- **Inputs**: Standardize `default`, `focused`, `error`, and `disabled` appearances.
- **Cards**: Use consistent padding, border-color, and shadow tokens.
- **Icons**: Standardize size sets (e.g., `16px`, `20px`, `24px`).

### Dark Mode Rules

- Use CSS variables for all colors to allow easy theme switching.
- Ensure contrast ratios remain valid in both light and dark modes.
- Avoid pure black (`#000`) for backgrounds; use deep grays or navy tints for better readability.
- Dim image/illustration brightness slightly in dark mode if they are too vibrant.

---

## Anti-Patterns

- Never hardcode color hex codes or pixel values in component styles.
- Never use random spacing values that fall outside the defined grid.
- Never create "one-off" component variations that aren't in the design system.
- Never use inconsistent border-radius values across the UI.
- Never calculate colors manually (e.g., `darken(color, 10%)`) in components; use tokens.

---

## Ready-to-Use Prompt

```
Task: Build [component name] using the Design System
Skill: frontend/design-system, frontend/web-ui

REQUIREMENTS:
- Component: [e.g., Button, Modal, Card]
- Theme: Support Light and Dark mode
- Tokens: Use defined color, spacing, and radius tokens

CONSTRAINTS:
- No hardcoded hex codes or pixel values
- Follow the 4px/8px spacing grid
- Adhere to the established typography scale
- Implement standard component variants

DONE WHEN:
- Component uses 100% tokens for styling
- Looks correct in both light and dark themes
- Matches existing UI consistency
```

