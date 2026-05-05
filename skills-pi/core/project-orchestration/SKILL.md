---
name: project-orchestration
description: Determines which skills to load and in what order based on project type. Load after reasoning on every task.
---
# project-orchestration.md

## Purpose

Serves as the Universal Entry Point. It identifies the project domain, loads the required skill stack, and enforces a standardized execution order to ensure consistency across all tasks.

## The Execution Pipeline

Regardless of the project type, the agent must follow this linear sequence:

1. **reasoning.md** → `Cognitive Phase`: Plan, breakdown, and edge-case analysis.
2. **Ambiguity Gate** → `Validation Phase`: Check for missing info or unclear requirements.
3. **[Domain Skills]** → `Implementation Phase`: Context-specific coding and architecture.
4. **code-review.md** → `Quality Phase`: Final audit against standards.
5. **git.md** → `Persistence Phase`: Conventional commit and push.

---

## 🚦 The Ambiguity Gate

Before moving from Reasoning to Implementation, the agent must pass the Ambiguity Gate.

**Conditions to pass:**
- The task is 100% understood (no "I think this means..." or "Assuming...").
- All necessary environment variables and API keys are confirmed.
- Target files and directories are clearly identified.
- Acceptance criteria are explicit and measurable.

**If the gate fails:**
- Stop and ask the user for clarification.
- Document the missing information in a TODO.
- Do not proceed with code generation until ambiguity is resolved.

---

## 🗺️ Project Domain Skill Maps

Each domain defines a curated stack of skills optimized for that specific architecture.

### Project Type Skill Maps

#### 1. Full Stack Web App

```
reasoning
├── frontend/web-ui
├── frontend/design-system
├── frontend/accessibility
├── frontend/interaction-patterns
├── frontend/responsive-layout
├── frontend/content-ux
├── frontend/visual-hierarchy
├── frontend/state-management
├── backend/rest-api
├── backend/auth
├── backend/error-handling
├── database/schema-design
├── database/migrations
├── core/api-design
├── core/security
├── core/testing
├── hosting/vercel          ← frontend
├── hosting/railway         ← backend
code-review
git
```

#### 2. REST API Only (Backend)

```
reasoning
├── backend/rest-api
├── backend/auth
├── backend/error-handling
├── backend/background-jobs
├── database/schema-design
├── database/sql
├── database/migrations
├── core/api-design
├── core/security
├── core/testing
code-review
git
```

#### 3. Mobile App

```
reasoning
├── frontend/mobile-ui
├── frontend/design-system
├── frontend/accessibility
├── frontend/interaction-patterns
├── frontend/responsive-layout
├── frontend/content-ux
├── frontend/visual-hierarchy
├── frontend/state-management
├── frontend/ui-conventions
├── backend/rest-api
├── backend/auth
├── database/schema-design
├── core/api-design
├── core/security
├── core/testing
code-review
git
```

#### 4. Data Engineering Pipeline

```
reasoning
├── data-engineering/etl-pipeline
├── data-engineering/orchestration
├── data-engineering/data-quality
├── data-engineering/warehouse
├── data-engineering/streaming        ← if event-driven
├── data-engineering/data-lake         ← if lake storage needed
├── database/schema-design
├── database/data-modeling
├── cloud/docker
├── cloud/ci-cd
├── core/security
├── core/testing
├── core/observability
code-review
git
```

#### 5. Data Analytics / Reporting

```
reasoning
├── data-analytics/data-retrieval
├── data-analytics/data-cleaning
├── data-analytics/data-exploration
├── data-analytics/data-transformation
├── data-analytics/visualization
├── data-analytics/dashboarding
├── data-analytics/reporting
├── data-analytics/metrics-and-kpis
├── data-analytics/statistical-analysis  ← if hypothesis testing needed
├── data-analytics/spreadsheet           ← if Excel/Sheets work needed
├── data-analytics/notebook-conventions
├── database/sql
code-review
git
```

#### 6. Cloud Infrastructure / DevOps

```
reasoning
├── cloud/docker
├── cloud/kubernetes
├── cloud/ci-cd
├── cloud/aws or cloud/gcp
├── cloud/infra-conventions
├── system-architecture/scalability
├── core/security
├── core/observability
code-review
git
```

#### 7. Full Stack Data Product

(Combines data engineering + analytics + web app)

```
reasoning
├── data-engineering/etl-pipeline
├── data-engineering/data-quality
├── data-analytics/visualization
├── data-analytics/dashboarding
├── backend/rest-api
├── frontend/web-ui
├── frontend/design-system
├── frontend/visual-hierarchy
├── database/schema-design
├── database/data-modeling
├── cloud/docker
├── hosting/vercel
├── core/api-design
├── core/security
├── core/testing
├── core/observability
code-review
git
```

#### 8. Microservices Architecture

```
reasoning
├── system-architecture/monolith-vs-microservices
├── system-architecture/clean-architecture
├── system-architecture/design-patterns
├── system-architecture/scalability
├── backend/rest-api
├── backend/auth
├── backend/background-jobs
├── database/schema-design
├── cloud/docker
├── cloud/kubernetes
├── cloud/ci-cd
├── core/api-design
├── core/security
├── core/observability
code-review
git
```

---

### How to Pick a Project Type

Answer these questions before loading any skills:

```
1. Is there a user interface?
   - Web → load frontend/web-ui
   - Mobile → load frontend/mobile-ui
   - Is UI/UX polish required? → load frontend/design-system, accessibility, interaction-patterns, responsive-layout, content-ux, visual-hierarchy
   - None → skip frontend skills

2. Is there a backend or API?
   - Yes → load backend/rest-api, backend/auth, backend/error-handling
   - No → skip backend skills

3. Is there a database?
   - Yes → load database/schema-design + sql or data-modeling
   - No → skip database skills

4. Is this data-focused?
   - Pipeline/ETL → load data-engineering track
   - Analysis/Reporting → load data-analytics track
   - Both → load Full Stack Data Product map

5. Where does it deploy?
   - Frontend → vercel or render
   - Backend → railway, render, or fly-io
   - Containerized → docker + kubernetes
   - Cloud-native → aws or gcp

6. What scale is expected?
   - Prototype → skip kubernetes, minimal cloud
   - Production → add kubernetes, ci-cd, observability
```

---

### Skill Conflict Resolution

When two skills have conflicting conventions on the same topic:

| Conflict              | Resolution                                                                |
| --------------------- | ------------------------------------------------------------------------- |
| Env var format        | `cloud/infra-conventions` wins                                            |
| Folder structure      | Most specific skill wins (e.g. `backend/rest-api` over `core/api-design`) |
| Error response format | `backend/error-handling` wins for backend, `core/api-design` for contract |
| Auth implementation   | `backend/auth` always wins                                                |
| Commit format         | `git.md` always wins                                                      |

When in doubt — the more specific skill overrides the more general one.

---

### AFK Workflow Integration

When running an AFK workflow, project-orchestration always runs first to:

1. Identify the project type from the task description
2. Load the correct skill map
3. Pass the loaded skills to `reasoning.md` for planning
4. **Bug Fix Workflow:** If the task is a bug fix, automatically load `skills/afk-workflows/bug-fix.md` and follow its diagnostic sequence.
5. Gate the end of execution with `code-review.md` and `git.md`

---

## Anti-Patterns

- Never start a task without loading `reasoning.md` first
- Never end a task without passing through `code-review.md` and `git.md`
- Never load all skills at once — only load what the project type requires
- Never skip the project type decision step — wrong skills produce wrong output
- Never resolve skill conflicts by ignoring one skill entirely — follow the resolution table

---

## Ready-to-Use Prompt

```
Task: [describe the project or feature]
Skill: project-orchestration

STEP 1 — IDENTIFY PROJECT TYPE:
Answer the following:
- Is there a UI? (web / mobile / none)
- Is there a backend or API? (yes / no)
- Is there a database? (yes / no)
- Is this data-focused? (pipeline / analytics / both / no)
- Where does it deploy? (vercel / railway / docker / cloud-native)
- What scale? (prototype / production)

STEP 2 — LOAD SKILL MAP:
Based on answers above, select the matching project type:
[ ] Full Stack Web App
[ ] REST API Only
[ ] Mobile App
[ ] Data Engineering Pipeline
[ ] Data Analytics / Reporting
[ ] Cloud Infrastructure / DevOps
[ ] Full Stack Data Product
[ ] Microservices Architecture

STEP 3 — CONFIRM LOADING ORDER:
1. reasoning.md — always first
2. [selected skill map in order]
3. code-review.md — always before commit
4. git.md — always last

STEP 4 — RESOLVE ANY CONFLICTS:
- Apply conflict resolution table if two skills disagree
- More specific skill always wins over general

STEP 5 — PASS AMBIGUITY GATE:
- Is the task 100% clear? (yes / no)
- Are all inputs/outputs identified? (yes / no)
- If "no" to either, stop and ask for clarification.

STEP 6 — PROCEED:
Hand off to reasoning.md with the full loaded skill list and the "Clear" signal from the Ambiguity Gate.
```

