---
name: bug-fix
description: AFK prompt template for autonomously diagnosing and fixing a bug or issue.
---
---
name: bug-fix
description: AFK prompt template for autonomously diagnosing and fixing a bug or issue. Use when a specific bug, error, or unexpected behavior needs to be investigated and resolved.
---

# bug-fix.md

## Purpose

Define the autonomous bug-fix workflow — a diagnosis-first, ambiguity-aware
process that prevents the agent from producing large incorrect diffs by
forcing it to understand the problem before writing a single line of fix.

---

## Core Principle

> Bug fixing has high ambiguity by default.
> Never produce a fix before completing the diagnosis phase.
> Output size must be proportional to ambiguity — the higher the ambiguity, the smaller each step.

---

## Ambiguity Rating Scale

Before doing anything, the agent must rate the bug's ambiguity:

| Rating    | Criteria                                                            | Approach                                                 |
| --------- | ------------------------------------------------------------------- | -------------------------------------------------------- |
| 🟢 Low    | Error is explicit, stack trace points to exact line, cause is clear | Single pass — diagnose + fix in one session              |
| 🟡 Medium | Error is clear but root cause has 2–3 possible sources              | Two pass — diagnose first, fix second                    |
| 🔴 High   | Behavior is unexpected, no clear error, multiple systems involved   | Three pass — diagnose, plan fix, implement incrementally |

Never self-rate as 🟢 unless a stack trace explicitly points to the cause.
When in doubt, rate higher.

---

## Conventions

### Phase 1 — Reproduce

Before reading any code, reproduce the bug:

- What is the exact error message or unexpected behavior?
- What are the exact steps to trigger it?
- Is it consistent or intermittent?
- What environment does it occur in? (OS, Python version, Node version, etc.)
- Does it occur in all environments or only specific ones?

If the bug cannot be reproduced — stop and document why. Do not attempt a fix.

### Phase 2 — Isolate

Read the relevant code, do not read everything:

- Start from the error location (stack trace, log line, failing test)
- Trace upward through call chains only as far as needed
- Identify the smallest unit of code that contains the fault
- List all files touched — nothing outside this list should be modified

```bash
# Useful isolation commands
grep -rn "error_text" .          # Find where error originates
git log --oneline -10            # Recent changes that may have caused it
git diff HEAD~1 -- <file>        # What changed in the suspect file
pytest tests/test_x.py -v        # Run only the relevant test
```

### Phase 3 — Ambiguity Assessment

After isolation, re-rate ambiguity and produce a written diagnosis:

```
BUG DIAGNOSIS REPORT
====================
Bug: [one-line description]
Ambiguity: 🟢 / 🟡 / 🔴

Root Cause (confirmed / suspected):
[describe exactly what is wrong and why]

Files involved:
- [file path] — [what role it plays in the bug]

Hypothesis:
[if suspected, list 2-3 possible causes ranked by likelihood]

Fix approach:
[describe the fix before writing any code]

Risk of fix:
[what could break if the fix is wrong]

Scope boundary:
[list exactly which files will be modified — nothing else]
```

Do not proceed to Phase 4 until this report is complete.

### Phase 4 — Fix

Apply the fix strictly within the scope boundary defined in Phase 3:

**For 🟢 Low ambiguity:**

- Apply fix directly
- Run tests immediately after
- If tests pass → proceed to Phase 5

**For 🟡 Medium ambiguity:**

- Apply fix for most likely hypothesis first
- Run tests
- If tests fail → revert and try next hypothesis
- Never apply multiple hypotheses simultaneously

**For 🔴 High ambiguity:**

- Fix one file at a time
- Run tests after each file change
- Commit after each verified fix — do not batch
- If a fix introduces a new failure → revert immediately, do not continue

### Phase 5 — Verify

After applying the fix:

```bash
# Confirm original bug is resolved
[reproduce steps from Phase 1 — confirm error no longer occurs]

# Confirm no regressions
pytest --tb=short                    # Full test suite
# or
npm test                             # Frontend tests

# Confirm no unintended file changes
git status
git diff --staged
```

If any test fails that was passing before — the fix introduced a regression.
Revert and re-diagnose before proceeding.

### Phase 6 — Code Review + Commit

Apply `code-review.md` gate in full before committing.

Commit message format for bug fixes:

```
fix(<scope>): <what was broken and how it was fixed>

Root cause: <one line>
Files changed: <list>
Tests added: <yes/no — describe>
```

Example:

```
fix(validation): resolve null pointer when target file has no headers

Root cause: compare() called before column inference completed
Files changed: backend/services/validator.py, tests/test_validator.py
Tests added: yes — added test_compare_with_empty_target_headers
```

---

## Anti-Patterns

- Never write a fix before completing the diagnosis report
- Never modify files outside the scope boundary defined in Phase 3
- Never apply multiple hypothesis fixes simultaneously
- Never skip test verification after applying a fix
- Never batch multiple bug fixes in one commit — one bug, one commit
- Never self-rate ambiguity as 🟢 without an explicit stack trace pointing to the line
- Never continue fixing if a regression is introduced — revert first

---

## Ready-to-Use AFK Prompt

```
Task: Fix the following bug — [paste error message or describe unexpected behavior]
Skill: afk-workflows/bug-fix, core/reasoning, core/security, core/testing, core/code-review, core/git
Project type: [describe project — e.g. FastAPI + Next.js fullstack]

PHASE 1 — REPRODUCE:
- Confirm the exact error message or behavior
- Confirm the exact steps to trigger it
- Confirm which environment it occurs in
- If cannot be reproduced: stop and document why

PHASE 2 — ISOLATE:
- Start from the stack trace or error location
- Trace only as far as needed
- List all files involved — nothing outside this list will be touched

PHASE 3 — AMBIGUITY ASSESSMENT:
Rate ambiguity: 🟢 Low / 🟡 Medium / 🔴 High
Produce the full BUG DIAGNOSIS REPORT before writing any fix.
Do not proceed until report is complete.

PHASE 4 — FIX:
Apply fix strictly within scope boundary.
🟢 → single pass fix
🟡 → fix most likely hypothesis first, test, then next if needed
🔴 → one file at a time, commit after each verified change

PHASE 5 — VERIFY:
- Reproduce original steps — confirm bug is gone
- Run full test suite — confirm no regressions
- Run git status — confirm no unintended file changes

PHASE 6 — CODE REVIEW + COMMIT:
- Apply code-review.md gate in full
- Commit with fix(<scope>): format
- Include root cause and files changed in commit body

CONSTRAINTS:
- Never modify files outside the scope boundary
- Never apply multiple hypotheses simultaneously
- Never skip test verification
- Never batch multiple bugs in one commit
- Revert immediately if a regression appears

DONE WHEN:
- Original bug no longer reproduces
- Full test suite passes with no new failures
- Commit message includes root cause
- code-review.md gate passed
```

