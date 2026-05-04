---
name: interaction-patterns
description: Loading/empty/error states, form validation UX, optimistic updates, and toast patterns.
---
# interaction-patterns.md

## Purpose

Standardize how the application communicates state changes, processes user input, and handles feedback to create a predictable and responsive user experience.

---

## Conventions

### Immediate Feedback

- Always acknowledge user actions immediately.
- Use subtle animations, hover states, or progress indicators to show the system is responding.
- For high-latency actions, use a "pending" or "optimistic" state.

### Affordance

- Every interactive element must visually signal its function.
- Interactive items should look interactive (e.g., hover effects, cursor changes, or depth/shadows).
- Maintain consistency: once a pattern is established (e.g., blue text = link), never use it for non-interactive content.

### Form Validation UX

- Validate on `blur` or `submit`, not on every keystroke (unless required for password strength).
- Clear error messages immediately when the user fixes the input.
- Keep the submit button disabled until the form is potentially valid if it prevents jarring errors.

### Optimistic Updates

- Update the UI immediately upon user action (e.g., liking a post).
- Roll back gracefully with a toast notification if the server request fails.
- Show a subtle loading indicator (e.g., dimmed state) while the sync is pending.

### Confirmation and Toast Patterns

- **Toasts**: Use for non-critical feedback (e.g., "Saved successfully"). Auto-dismiss after 3-5 seconds.
- **Modals**: Use for destructive actions (e.g., "Delete account") or complex inputs.
- **Placement**: Keep toasts in a consistent corner (usually top-right or bottom-center).

---

## Anti-Patterns

- Never leave the user wondering if something is happening (always show progress).
- Never use generic "Something went wrong" messages without actionable next steps.
- Never hide critical functionality behind a hover-only state (bad for mobile/a11y).
- Never show multiple toasts that stack and cover the entire screen.
- Never use aggressive animations that slow down the actual interaction.

---

## Ready-to-Use Prompt

```
Task: Implement [feature] with robust Interaction Patterns
Skill: frontend/interaction-patterns, frontend/state-management

REQUIREMENTS:
- Action: [e.g., Creating a new record]
- States: Loading, Success, Error, Empty
- Feedback: Use optimistic updates and toasts

CONSTRAINTS:
- Use skeletons for initial load
- Implement inline validation for inputs
- Provide a clear "Undo" or "Retry" if applicable
- Ensure high-speed feedback (under 100ms)

DONE WHEN:
- The UI feels snappy and responsive
- Errors are clear and recoverable
- No "dead ends" in the user flow
```

