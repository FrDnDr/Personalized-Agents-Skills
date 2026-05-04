# Agent Instructions

## 🎯 Role & Context

You are an expert AI developer and systems architect. You prioritize efficiency, security, and clean code.

---

## 🧠 Core Logic & Reasoning

- **Primary Entry Point:** Always initialize every task by accessing `skills/core/project-orchestration.md`.
- **Universal Pipeline:** Strictly follow this sequence every task, no exceptions:
  ```
  reasoning.md → project-orchestration.md → [domain skills] → code-review.md → git.md
  ```
- **Skill-First Approach:** Always cross-reference the `./skills/core/` directory before providing solutions.
- **Thinking:** Use the logic defined in `skills/core/reasoning.md` as your primary cognitive framework.
- **Conflict Resolution:** Refer to the conflict resolution table in `project-orchestration.md` when skills provide competing conventions. More specific skill always wins over general.

---

## 🗂 Skill Directory

### `skills/core/` — Universal (Loaded on Every Task)

| File                       | Purpose                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `reasoning.md`             | Pre-execution thinking, task breakdown, requirements analysis, edge case identification, execution planning |
| `project-orchestration.md` | Which skills to load and in what order per project type, skill conflict resolution                          |
| `project-planning.md`      | Project structure, task breakdown, sprint planning before executing any code                                |
| `api-design.md`            | REST conventions, versioning, modular templates for REST, FastAPI, Express, GraphQL                         |
| `security.md`              | Secrets management, input validation, SQL injection/XSS/CSRF prevention, HTTP headers, auth security        |
| `testing.md`               | Unit/integration/E2E conventions, pytest + vitest stack, AAA pattern, coverage thresholds                   |
| `observability.md`         | Logging format, metrics, alerting, error tracking setup _(pending)_                                         |
| `code-review.md`           | Pre-commit checklist, untracked file audit, diff review, quality gates, change justification log            |
| `git.md`                   | Branching strategy, commit conventions, PR rules, .gitignore standards, gh CLI operations                   |

### `skills/frontend/` — UI Layer

| File                  | Purpose                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------- |
| `web-ui.md`           | React/Next.js App Router conventions, component structure, Tailwind, server vs client components  |
| `mobile-ui.md`        | React Native conventions, screen structure, SafeAreaView, FlatList, platform-specific extensions  |
| `state-management.md` | State decision tree, Zustand for UI state, React Query for server data, React Hook Form for forms |
| `ui-conventions.md`   | Platform-specific UI rules, iOS HIG, Android Material 3, platform-specific patterns              |
| `design-system.md`    | Design tokens (color, spacing, radius), component variants, and dark mode rules                   |
| `accessibility.md`    | Keyboard navigation, focus states, contrast thresholds, and semantic HTML + ARIA rules            |
| `interaction-patterns.md` | Loading/empty/error states, form validation UX, optimistic updates, and toast patterns        |
| `responsive-layout.md` | Breakpoint strategy, container widths, and mobile-first spacing/layout rules                     |
| `content-ux.md`       | Microcopy style (labels, errors, CTA text), truncation, and localization conventions              |
| `visual-hierarchy.md` | Typography scale, spacing rhythm, and information prioritization rules                            |

### `skills/backend/` — Server Layer

| File                 | Purpose                                                                                                   |
| -------------------- | --------------------------------------------------------------------------------------------------------- |
| `rest-api.md`        | FastAPI modular structure, router/controller/service/schema pattern, async conventions, response envelope |
| `auth.md`            | JWT access + refresh token flow, bcrypt passwords, dependency injection for auth, RBAC, OAuth2            |
| `background-jobs.md` | Celery + Redis setup, idempotent tasks, retry logic, exponential backoff, Celery Beat scheduling          |
| `error-handling.md`  | Custom exception hierarchy, global handlers, standard error envelope, what never to expose in responses   |

### `skills/database/` — Data Layer

| File               | Purpose                                                                                               |
| ------------------ | ----------------------------------------------------------------------------------------------------- |
| `schema-design.md` | UUID PKs, required columns, constraint naming, FK ON DELETE behavior, indexing rules, soft deletes    |
| `sql.md`           | SELECT conventions, joins, aggregations, window functions, upsert, keyset pagination, transactions    |
| `migrations.md`    | Alembic conventions, file naming, upgrade/downgrade structure, migration workflow, production rules   |
| `data-modeling.md` | Entity modeling, 1NF/2NF/3NF normalization, relationship patterns, relational vs NoSQL decision table |

### `skills/data-engineering/` — Pipeline Layer _(pending)_

| File               | Purpose                                                                  |
| ------------------ | ------------------------------------------------------------------------ |
| `etl-pipeline.md`  | Extract, transform, load conventions, pipeline structure, error handling |
| `orchestration.md` | Airflow/Prefect DAG conventions, scheduling, task dependencies           |
| `data-quality.md`  | Validation rules, null checks, schema enforcement, data testing          |
| `warehouse.md`     | dbt project structure, Snowflake/BigQuery conventions, model layering    |
| `streaming.md`     | Kafka topic conventions, consumer/producer patterns, event schemas       |
| `data-lake/.keep`  | Placeholder — structure TBD                                              |

### `skills/data-analytics/` — Analytics Layer _(pending)_

| File                      | Purpose                                                                   |
| ------------------------- | ------------------------------------------------------------------------- |
| `data-retrieval.md`       | Querying from DBs, APIs, flat files, web scraping basics                  |
| `data-cleaning.md`        | Handling nulls, duplicates, outliers, type casting, pandas conventions    |
| `data-exploration.md`     | EDA patterns, descriptive stats, correlation, distribution analysis       |
| `data-transformation.md`  | Reshaping, aggregating, joining datasets, feature engineering basics      |
| `visualization.md`        | Chart selection guide, Matplotlib/Seaborn/Plotly conventions, color rules |
| `dashboarding.md`         | Dashboard layout, Streamlit/Metabase/Grafana conventions                  |
| `reporting.md`            | Report structure, narrative writing around data, executive summaries      |
| `metrics-and-kpis.md`     | Defining metrics, KPI frameworks, avoiding vanity metrics                 |
| `statistical-analysis.md` | Hypothesis testing, confidence intervals, A/B testing basics              |
| `spreadsheet.md`          | Excel/Google Sheets conventions, formulas, pivot tables, data entry rules |
| `notebook-conventions.md` | Jupyter notebook structure, cell ordering, markdown documentation rules   |

### `skills/cloud/` — Infrastructure Layer _(pending)_

| File                   | Purpose                                                                     |
| ---------------------- | --------------------------------------------------------------------------- |
| `aws.md`               | AWS service conventions, IAM rules, preferred services per use case         |
| `gcp.md`               | GCP service conventions, project structure, preferred services per use case |
| `docker.md`            | Dockerfile conventions, image layering, multi-stage builds, compose setup   |
| `kubernetes.md`        | Cluster setup, deployment manifests, scaling rules, scalability testing     |
| `ci-cd.md`             | Pipeline structure, build/test/deploy stages, environment promotion rules   |
| `infra-conventions.md` | Env var management, secrets injection, config per environment               |

### `skills/hosting/` — Deployment Layer _(pending)_

| File                  | Purpose                                                                 |
| --------------------- | ----------------------------------------------------------------------- |
| `vercel.md`           | Next.js/frontend deployment conventions, env setup, preview deployments |
| `railway.md`          | Backend/fullstack deployment, service linking, database provisioning    |
| `render.md`           | Alternative to Railway, static sites, web services, cron jobs           |
| `fly-io.md`           | Containerized app deployment, region selection, scaling configuration   |
| `supabase.md`         | Backend-as-a-service setup, auth integration, DB and storage usage      |
| `hosting-decision.md` | Decision guide — when to pick which hosting service per project type    |

### `skills/system-architecture/` — Architecture Layer _(pending)_

| File                           | Purpose                                                                      |
| ------------------------------ | ---------------------------------------------------------------------------- |
| `README.md`                    | Architecture overview, how to pick an architecture per project type          |
| `clean-architecture.md`        | Layers (entity, use case, adapter, infra), dependency rule, folder structure |
| `monolith-vs-microservices.md` | When to use each, tradeoffs, migration path                                  |
| `design-patterns.md`           | Common patterns (repository, factory, observer) and when to apply them       |
| `scalability.md`               | Horizontal vs vertical scaling, caching strategies, load balancing           |
| `adr.md`                       | Architecture Decision Record template and when to write one                  |

### `skills/afk-workflows/` — Autonomous Workflows

| File                  | Purpose                                                          |
| --------------------- | ---------------------------------------------------------------- |
| `feature-sprint.md`   | AFK prompt template for developing a full feature end-to-end     |
| `bug-fix.md`          | AFK prompt template for diagnosing and fixing a bug or issue     |
| `project-scaffold.md` | AFK prompt template for bootstrapping a new project from scratch |

---

## 🛠 Technical Standards

- **Project Planning:** Use `skills/core/project-planning.md` to determine project structure and task breakdown before executing any code.
- **Project Orchestration:** Use `skills/core/project-orchestration.md` to determine which skills to load per task.
- **Architecture:** Standardize on the Repository Pattern for data-heavy applications.
- **Security:** Adhere to `skills/core/security.md`. Never hardcode credentials and always validate inputs.
- **Testing:** All core logic must include unit tests as defined in `skills/core/testing.md`.
- **Git:** Use Conventional Commits (`feat:`, `fix:`, `docs:`, etc.) for all version control interactions.

---

## 🚫 Operational Constraints

- Explain terminal commands before running them unless in `--yolo` mode.
- Do not use deprecated libraries or magic numbers.
- Preserve existing file formatting and comments during edits.
- Never skip `reasoning.md` phase — even for simple tasks.
- Never commit without passing `code-review.md` gate first.
- **Bug Fixes:** Always use the `skills/afk-workflows/bug-fix.md` diagnostic sequence when fixing bugs.
- Skills marked _(pending)_ are not yet written — do not reference them until they exist.
