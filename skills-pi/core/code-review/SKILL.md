---
name: code-review
description: Pre-commit gate covering untracked file audit, diff review, quality checklist, and change justification log. Always load before git.
---
# code-review.md

## Purpose

Gate every commit with a structured pre-commit review — comparing what changed,
why the change was necessary, and whether it meets all loaded skill conventions
before anything is staged or committed.

---

## Conventions

### When This Skill Activates

- **Always** — before every `git add` and `git commit`, no exceptions
- Even for "small" or "obvious" fixes — every change goes through this gate
- After any AFK workflow completes, before the final commit

---

### Phase 1 — Untracked File Audit

Before reviewing any code, check the full working tree:

```bash
git status
```

For every file listed:

- **Modified** — is this change intentional and in scope for this task?
- **Untracked** — should this file be committed or added to `.gitignore`?
- **Deleted** — was this deletion deliberate or accidental?

Flag any file that was not part of the original task plan.
Do not stage flagged files until the reason is confirmed.

---

### Phase 2 — Diff Review

Read the full diff of all staged changes before committing:

```bash
git diff --staged
```

For every change in the diff, the agent must answer:

1. **What changed?** — describe the change in one line
2. **Why was this change necessary?** — justify it against the task requirements
3. **Does it follow the conventions of all loaded skills?** — check each loaded skill
4. **Does it introduce any risk?** — breaking change, security issue, performance impact

If any change cannot be justified — it must be removed or documented as a TODO.

---

### Phase 3 — Quality Gates

Every staged change must pass all of the following before committing:

#### Security

- [ ] No hardcoded credentials, API keys, tokens, or passwords
- [ ] No `.env` files or secret files staged
- [ ] No sensitive data exposed in logs or responses

#### Code Quality

- [ ] No debug `print()`, `console.log()`, or `debugger` statements left in
- [ ] No commented-out dead code left in without explanation
- [ ] No TODO left unresolved unless explicitly flagged with a reason
- [ ] No unused imports or variables
- [ ] No magic numbers or hardcoded values without constants

#### Conventions

- [ ] Follows naming conventions of the loaded skills
- [ ] Follows folder and module structure of the loaded skills
- [ ] Error handling is present where required
- [ ] Input validation is present where required

#### Tests

- [ ] Existing tests still pass
- [ ] New functionality has corresponding tests where applicable

---

### Phase 4 — Change Justification Log

Before committing, the agent must produce a one-line justification per changed file:

```
CHANGE LOG:
- src/users/router.py     → Added POST /users endpoint per task requirement
- src/shared/response.py  → Extended response formatter to include error code field
- .gitignore              → Added *.log exclusion missed during scaffolding
```

This log becomes the basis for the commit message.

---

### Phase 5 — Commit Message

After passing all gates, write the commit message using Conventional Commits format
as defined in `git.md`:

```
<type>(<scope>): <description>

[optional body — paste the change justification log here if multiple files changed]
```

---

## Anti-Patterns

- Never skip this skill for "quick" or "small" fixes — all commits go through the gate
- Never stage files that were not part of the task scope without explicit justification
- Never commit if any quality gate item is unchecked
- Never write a vague commit message after producing a change log — use the log
- Never treat a passing diff as automatic approval — always read and justify each change
- Never commit untracked files without first deciding: commit it or ignore it

---

## Ready-to-Use Prompt

```
Task: Pre-commit code review for [task or feature name]
Skill: code-review, git

PHASE 1 — UNTRACKED FILE AUDIT:
- Run `git status`
- For every file: is it intentional, in scope, or should it be ignored?
- Flag anything not in the original task plan — do not stage until confirmed

PHASE 2 — DIFF REVIEW:
- Run `git diff --staged`
- For every change answer:
  1. What changed?
  2. Why was this change necessary?
  3. Does it follow all loaded skill conventions?
  4. Does it introduce any risk?
- Remove or flag as TODO any change that cannot be justified

PHASE 3 — QUALITY GATES:
Security:
  - [ ] No hardcoded credentials, keys, or tokens
  - [ ] No .env or secret files staged
  - [ ] No sensitive data in logs or responses
Code Quality:
  - [ ] No debug statements left in
  - [ ] No dead commented-out code without explanation
  - [ ] No unresolved TODOs without a reason
  - [ ] No unused imports or variables
Conventions:
  - [ ] Follows naming and structure of all loaded skills
  - [ ] Error handling present where required
  - [ ] Input validation present where required
Tests:
  - [ ] Existing tests pass
  - [ ] New functionality has tests where applicable

PHASE 4 — CHANGE JUSTIFICATION LOG:
- Produce one-line justification per changed file
- Format: filename → reason for change

PHASE 5 — COMMIT:
- Write commit message using Conventional Commits format
- Use the change justification log as the commit body

DO NOT COMMIT if any gate is unchecked.
DO NOT COMMIT if any change cannot be justified.
```

