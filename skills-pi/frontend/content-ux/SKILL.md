---
name: content-ux
description: Microcopy style (labels, errors, CTA text), truncation, and localization conventions.
---
# content-ux.md

## Purpose

Standardize the language, tone, and formatting of text within the UI to improve clarity, reduce cognitive load, and prepare for global audiences.

---

## Conventions

### Microcopy Style

- **Concise**: Use short, punchy sentences. Remove filler words.
- **Action-Oriented**: Use verbs to start labels and buttons (e.g., "Save Changes" vs. "Ok").
- **Consistent**: Use the same term for the same action throughout the app (e.g., don't mix "Delete" and "Remove").
- **Empathetic**: Use human language for errors (e.g., "We couldn't find that page" vs. "404 Not Found").

### Truncation Rules

- Avoid truncating critical information (e.g., prices, IDs).
- Use `ellipsis` for long titles/names but provide a tooltip on hover to show the full text.
- Define a "line-clamp" strategy for multi-line descriptions (e.g., max 3 lines).

### Localization (i18n) Readiness

- Never hardcode strings in JSX; use a translation key system (e.g., `t('auth.login')`).
- Design for text expansion (German/French text is often 30% longer than English).
- Standardize date, time, and currency formatting based on locale.

---

## Anti-Patterns

- Never use "Click here" as link text — be descriptive.
- Never use all-caps for long sentences (it's hard to read).
- Never use technical jargon or internal database IDs in user-facing messages.
- Never assume a specific text length in your CSS (e.g., `width: 100px` for a label).
- Never use sarcastic or overly "cute" language in error messages.

---

## Ready-to-Use Prompt

```
Task: Review/Refine Microcopy for [feature]
Skill: frontend/content-ux, frontend/ui-conventions

REQUIREMENTS:
- Scope: Labels, placeholders, error messages, and CTAs
- Tone: [e.g., Professional/Helpful]
- i18n: Ensure all strings are extracted to keys

CONSTRAINTS:
- Keep buttons under 3 words
- Use active voice
- Ensure error messages are actionable
- Check for text overflow in long translations

DONE WHEN:
- No hardcoded strings remain in the code
- All text is clear and follows the style guide
- Tooltips are present for truncated text
```

