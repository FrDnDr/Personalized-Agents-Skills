reasoning.md
markdown# Reasoning Skill

## Purpose

Force the agent to think deeply before producing any output — mirroring
Claude AI's extended thinking process. No code, no files, no commands
are executed until the reasoning phase is fully complete.

---

## Conventions

### Phase 1 — Understand

Before anything else, the agent must:

- Restate the task in its own words
- Identify what is explicitly asked vs what is implied
- List all inputs, outputs, and constraints
- Flag any ambiguities — do not assume, document them
- Identify which skills are relevant to this task

### Phase 2 — Analyze

- Break the task into the smallest logical units of work
- Identify dependencies between units — what must happen before what
- Identify edge cases:
  - What happens if input is null, empty, or malformed?
  - What happens if an external service is unavailable?
  - What happens if the task is partially completed and interrupted?
- Identify risks:
  - Could this change break existing functionality?
  - Are there security implications?
  - Are there performance implications at scale?

### Phase 3 — Plan

- Produce a numbered step-by-step execution plan before writing a single line
- Each step must be:
  - Atomic — one clear action per step
  - Verifiable — has a clear done condition
  - Ordered — dependencies respected
- Estimate which steps are high risk and flag them explicitly
- Define what "done" looks like for the entire task

### Phase 4 — Validate the Plan

Before executing, the agent must ask internally:

- Does this plan fully satisfy the requirements?
- Does it follow the conventions of all loaded skills?
- Are there simpler approaches that achieve the same result?
- Is any step doing more than one thing? If yes, split it.
- Is the plan reversible if something goes wrong?

### Phase 5 — Execute

- Only after Phases 1–4 are complete does the agent begin execution
- Execute strictly according to the plan — no improvising mid-task
- If an unexpected blocker appears during execution:
  - Stop
  - Re-enter Phase 2 for that specific blocker
  - Update the plan
  - Continue
- Log reasoning inline as comments when a non-obvious decision is made

### Phase 6 — Review

- After execution, before any commit:
  - Re-read all changes against the original requirements
  - Check that every planned step was completed
  - Verify no unintended files were modified
  - Apply `code-review.md` as the final gate

---

## Anti-Patterns

- Never skip straight to writing code without completing Phases 1–3
- Never assume ambiguous requirements — document them explicitly
- Never make decisions mid-execution that weren't in the plan without re-entering Phase 2
- Never proceed if Phase 4 validation fails — replan first
- Never treat reasoning as optional for "simple" tasks — every task goes through all phases
- Never leave the reasoning log empty — even one-liners must have a documented rationale

---

## Ready-to-Use Prompt

```
REASONING PHASE — execute before anything else:

PHASE 1 — UNDERSTAND:
- Restate the task in your own words
- List all explicit requirements
- List all implied requirements
- List all constraints from loaded skills
- Flag any ambiguities — do not assume

PHASE 2 — ANALYZE:
- Break task into atomic units of work
- Map dependencies between units
- Identify edge cases (null input, service failure, partial completion)
- Identify risks (breaking changes, security, performance)

PHASE 3 — PLAN:
- Write a numbered step-by-step execution plan
- Each step: one action, one done condition
- Flag high-risk steps explicitly
- Define what "done" looks like for the full task

PHASE 4 — VALIDATE:
- Does the plan satisfy all requirements?
- Does it follow all loaded skill conventions?
- Is there a simpler approach?
- Is every step atomic and reversible?
- Only proceed if all answers are satisfactory

PHASE 5 — EXECUTE:
- Follow the plan strictly
- Log reasoning inline for non-obvious decisions
- If blocked: stop, re-analyze, update plan, continue

PHASE 6 — REVIEW:
- Re-read all changes against requirements
- Verify all planned steps completed
- Check for unintended file modifications
- Apply code-review.md before any commit
```
