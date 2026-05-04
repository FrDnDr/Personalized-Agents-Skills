---
name: visual-hierarchy
description: Typography scale, spacing rhythm, and information prioritization rules.
---
# visual-hierarchy.md

## Purpose

Direct the user's attention to the most important information and actions through the strategic use of size, color, weight, and spacing.

---

## Conventions

### Typography Scale

- **H1**: Page title (highest importance). One per page.
- **H2**: Section headers.
- **H3**: Subsection/Card headers.
- **Body**: Standard reading text.
- **Caption**: Low importance/meta-data.
- Use font-weight (`bold` vs `normal`) to differentiate titles from body text.

### Spacing Rhythm

- Use larger gaps between major sections and smaller gaps between related elements (Gestalt principle of proximity).
- Standardize vertical margins between paragraphs and headers.
- Use whitespace (padding) to prevent a cluttered UI.

### Priority and Scanning (The "Invisible UI")

- Direct the user's eye naturally through the page using scale, weight, and positioning.
- The best UI is "invisible"—structured so intuitively that users navigate without conscious thought.
- **Primary Actions**: Use high-contrast colors (e.g., solid background).
- **Secondary Actions**: Use lower contrast (e.g., outline or ghost buttons).
- **Color Balance (60-30-10)**: Ensure the UI isn't overwhelming by using 60% neutral/dominant, 30% secondary, and only 10% accent color.
- **Scanability**: Use bullet points, bold text, and clear headings.

---

## Anti-Patterns

- Never have more than one primary button in the same view.
- Never use the same font size and weight for headers and body text.
- Never clutter the UI with too many competing colors.
- Never use small, low-contrast text for important information.
- Never ignore the "fold" — keep the most critical action visible without scrolling.

---

## Ready-to-Use Prompt

```
Task: Apply Visual Hierarchy to [page/component]
Skill: frontend/visual-hierarchy, frontend/design-system

REQUIREMENTS:
- Component: [e.g., Landing Page Hero, Data Table]
- Goal: Highlight the [Primary Action]

CONSTRAINTS:
- Use established typography scale
- Apply consistent spacing rhythm
- Ensure the primary CTA is the most visible element
- De-emphasize secondary meta-data

DONE WHEN:
- The user's eye is naturally drawn to the correct starting point
- Content is easy to scan
- Spacing feels balanced and intentional
```

