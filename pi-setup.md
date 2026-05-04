# Pi Environment Setup — Skill Absorption Prompt

## Overview

This prompt sets up your Pi coding agent environment to absorb and use
all custom skills. Run this once globally, or per project.

---

## Step 1 — File Placement

### Global Setup (skills available in ALL projects)

```bash
# Create the global skills directory structure
mkdir -p ~/.pi/agent/skills/core
mkdir -p ~/.pi/agent/skills/frontend
mkdir -p ~/.pi/agent/skills/backend
mkdir -p ~/.pi/agent/skills/database
mkdir -p ~/.pi/agent/skills/data-engineering
mkdir -p ~/.pi/agent/skills/data-analytics
mkdir -p ~/.pi/agent/skills/cloud
mkdir -p ~/.pi/agent/skills/hosting
mkdir -p ~/.pi/agent/skills/system-architecture
mkdir -p ~/.pi/agent/skills/afk-workflows

# Copy your skill files into global skills
cp skills/core/*.md ~/.pi/agent/skills/core/
cp skills/frontend/*.md ~/.pi/agent/skills/frontend/
cp skills/backend/*.md ~/.pi/agent/skills/backend/
cp skills/database/*.md ~/.pi/agent/skills/database/
cp skills/afk-workflows/*.md ~/.pi/agent/skills/afk-workflows/

# Copy AGENTS.md globally
cp AGENTS.md ~/.pi/agent/AGENTS.md
```

### Project Setup (skills available in ONE project only)

```bash
# Create project-level skills directory
mkdir -p .pi/skills/core
mkdir -p .pi/skills/frontend
mkdir -p .pi/skills/backend
mkdir -p .pi/skills/database

# Copy skill files into project
cp skills/core/*.md .pi/skills/core/
cp skills/frontend/*.md .pi/skills/frontend/
cp skills/backend/*.md .pi/skills/backend/
cp skills/database/*.md .pi/skills/database/
cp skills/afk-workflows/*.md .pi/skills/afk-workflows/

# Copy AGENTS.md to project root
cp AGENTS.md ./AGENTS.md
```

---

## Step 2 — Absorption Prompt

Paste this as your **first message** when starting a Pi session on any project.
This forces Pi to read and internalize all skills before doing anything else.

```
Before you do anything else, absorb my full skill library by doing the following:

STEP 1 — READ AGENTS.md:
Read AGENTS.md in full. This is your master instruction file.
Understand the universal pipeline: reasoning → project-orchestration → domain skills → code-review → git.

STEP 2 — READ ALL CORE SKILLS (required on every task):
Read each of the following files in order:
- skills/core/reasoning.md
- skills/core/project-orchestration.md
- skills/core/api-design.md
- skills/core/security.md
- skills/core/testing.md
- skills/core/code-review.md
- skills/core/git.md

STEP 3 — READ DOMAIN SKILLS:
Read each of the following files:
Frontend:
- skills/frontend/web-ui.md
- skills/frontend/mobile-ui.md
- skills/frontend/state-management.md
- skills/frontend/ui-conventions.md

Backend:
- skills/backend/rest-api.md
- skills/backend/auth.md
- skills/backend/background-jobs.md
- skills/backend/error-handling.md

Database:
- skills/database/schema-design.md
- skills/database/sql.md
- skills/database/migrations.md
- skills/database/data-modeling.md

AFK Workflows:
- skills/afk-workflows/bug-fix.md

STEP 4 — CONFIRM ABSORPTION:
After reading all files, confirm with this exact format:

✅ SKILLS ABSORBED
Core: reasoning, project-orchestration, api-design, security, testing, code-review, git
Frontend: web-ui, mobile-ui, state-management, ui-conventions
Backend: rest-api, auth, background-jobs, error-handling
Database: schema-design, sql, migrations, data-modeling
AFK Workflows: bug-fix

Active pipeline: reasoning → project-orchestration → [domain skills] → code-review → git

Ready for tasks.

STEP 5 — IDENTIFY PROJECT TYPE:
Look at the current project directory and identify:
- What type of project is this? (fullstack / API / mobile / data / etc.)
- Which skill map from project-orchestration.md applies?
- Are there any existing conventions to preserve?

Report your findings before accepting any task.
```

---

## Step 3 — Per-Task Skill Invocation

For specific tasks, invoke skills directly using Pi's skill syntax:

```bash
# Invoke a specific skill manually
/skill:core/reasoning
/skill:backend/rest-api
/skill:database/schema-design

# Or reference them in your prompt
"Using skills/backend/auth.md and skills/core/security.md,
implement JWT authentication for this project."
```

---

## Step 4 — AFK Workflow Prompt (One Hour Autonomous Run)

Use this when handing off a full task to run unattended:

```
Task: [describe feature, bug fix, or module to build]

SKILLS TO LOAD:
- skills/core/reasoning.md
- skills/core/project-orchestration.md
- [add domain skills based on task type]
- skills/core/security.md
- skills/core/testing.md
- skills/core/code-review.md
- skills/core/git.md

EXECUTE REASONING PHASE FIRST:
Follow all 6 phases in skills/core/reasoning.md before writing any code.

CONSTRAINTS:
- Follow all conventions from loaded skills exactly
- No hardcoded secrets or credentials
- Commit after each logical unit of work using Conventional Commits
- If blocked: document blocker as TODO comment, log reasoning, move on
- Do not ask for confirmation — make decisions and log your reasoning
- Apply code-review.md gate before every commit
- Do not modify files outside the task scope

DONE WHEN:
- [describe clear acceptance criteria]
- All tests pass
- Code review gate passed
- Changes committed with proper commit messages
```

---

## Step 5 — Adding New Skills as You Write Them

Every time you complete a new skill file, run:

```bash
# Global
cp skills/<track>/<skill>.md ~/.pi/agent/skills/<track>/

# Project-level
cp skills/<track>/<skill>.md .pi/skills/<track>/
```

Then update AGENTS.md to remove the _(pending)_ tag from that skill.

---

## Quick Reference — Current Skill Status

| Track                  | Status         | Files                                                                                                       |
| ---------------------- | -------------- | ----------------------------------------------------------------------------------------------------------- |
| `core/`                | ✅ 7 of 8 done | reasoning, project-orchestration, api-design, security, testing, code-review, git _(observability pending)_ |
| `frontend/`            | ✅ Complete    | web-ui, mobile-ui, state-management, ui-conventions                                                         |
| `backend/`             | ✅ Complete    | rest-api, auth, background-jobs, error-handling                                                             |
| `database/`            | ✅ Complete    | schema-design, sql, migrations, data-modeling                                                               |
| `data-engineering/`    | ⏳ Pending     | —                                                                                                           |
| `data-analytics/`      | ⏳ Pending     | —                                                                                                           |
| `cloud/`               | ⏳ Pending     | —                                                                                                           |
| `hosting/`             | ⏳ Pending     | —                                                                                                           |
| `system-architecture/` | ⏳ Pending     | —                                                                                                           |
| `afk-workflows/`       | ✅ 1 of 3 done | bug-fix _(feature-sprint, project-scaffold pending)_                                                        |
