# ui-conventions.md

## Purpose

Define platform-specific UI/UX patterns, accessibility rules, and design
conventions that apply across web and mobile to ensure consistent,
native-feeling interfaces on every platform.

---

## Conventions

### Universal Rules (Web + Mobile)

- Always handle all three interaction states: default, hover/press, disabled
- Always provide loading feedback for any action that takes >300ms
- Always show empty states — never render a blank screen
- Always show error states with a recovery action (retry button, back link)
- Never use color alone to convey meaning — always pair with text or icon
- Minimum tap/click target size: 44x44px (Apple HIG standard)

### Web — UI Conventions

#### Layout

- Use CSS Grid for page-level layout, Flexbox for component-level layout
- Max content width: 1280px centered — never full-bleed text on large screens
- Consistent spacing scale: 4px base unit (4, 8, 12, 16, 24, 32, 48, 64)
- Always use semantic HTML: `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`

#### Typography

- Never use more than 3 font sizes per page section
- Body text minimum: 16px
- Line height for body: 1.5
- Never use pure black (`#000`) for text — use near-black (`#111`, `#1a1a1a`)

#### Interactivity

- All interactive elements must have visible focus states (outline or ring)
- Hover states required for all clickable elements on desktop
- Transitions max 200ms — never animate layout shifts
- Never disable browser scroll behavior without an explicit reason

#### Accessibility (Web)

- All images must have `alt` text — decorative images use `alt=""`
- All form inputs must have associated `<label>` elements
- Color contrast minimum: 4.5:1 for normal text, 3:1 for large text
- All interactive elements must be keyboard navigable
- Use ARIA roles only when semantic HTML is insufficient

### iOS — UI Conventions

- Follow Apple Human Interface Guidelines (HIG)
- Navigation: use stack-based navigation for hierarchy, tabs for top-level sections
- Destructive actions (delete, remove): always use red color + confirmation dialog
- Use system fonts (SF Pro) unless brand requires custom font
- Respect Dynamic Type — never hardcode font sizes, use system text styles
- Safe area insets must be respected on all screens
- Pull-to-refresh for list screens where data can update

### Android — UI Conventions

- Follow Material Design 3 guidelines
- Navigation: use bottom navigation for top-level sections
- Use Material You dynamic color theming when possible
- Back navigation must always work — never trap users in a screen
- FAB (Floating Action Button) for primary actions on list screens
- Ripple effect required on all touchable elements
- Respect system font scaling — never hardcode text sizes

### Platform Differences to Handle Explicitly

| Behavior          | iOS                   | Android                     |
| ----------------- | --------------------- | --------------------------- |
| Back navigation   | Swipe from left edge  | System back button/gesture  |
| Keyboard dismiss  | Tap outside or scroll | Back button                 |
| Date picker       | Wheel spinner         | Calendar dialog             |
| Alerts            | Centered modal        | Bottom-anchored dialog      |
| Loading indicator | `ActivityIndicator`   | `CircularProgressIndicator` |

---

## Anti-Patterns

- Never use platform-specific UI patterns on the wrong platform
- Never ignore safe area insets on mobile
- Never skip empty and error states — blank screens break trust
- Never animate layout shifts — only animate opacity and transforms
- Never use color alone to indicate state (error, success, warning)
- Never skip keyboard accessibility on web
- Never hardcode font sizes on mobile — breaks system accessibility settings

---

## Ready-to-Use Prompt

```
Task: Implement UI conventions for [screen, component, or page name]
Skill: frontend/ui-conventions, frontend/web-ui or frontend/mobile-ui

REQUIREMENTS:
- Platform: [web / iOS / Android / all]
- Component type: [page / screen / form / list / modal]
- Interactive states needed: [default, hover, pressed, disabled, loading, error, empty]

CONSTRAINTS:
- Follow platform-specific conventions from ui-conventions.md
- Minimum tap target: 44x44px on mobile
- All states must be handled: loading, error, empty, success
- Accessibility: alt text, labels, focus states, contrast on web
- No hardcoded font sizes on mobile
- No color-only state indicators

DONE WHEN:
- All interactive states implemented
- Platform conventions followed correctly
- Accessibility requirements met
- Empty and error states have recovery actions
- Code review skill applied before commit
```
