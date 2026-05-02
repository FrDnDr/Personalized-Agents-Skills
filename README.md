**Personalized Agents Skills**

This repository contains a collection of skills for my personalized agents, which can be used to enhance the capabilities of AI agents.

_Note: This is on my personal preference, so if you want to reference this, be sure to use your own judgment and modify it to your needs._

### Project Structure

```text
/skills/
│
├── core/
│   ├── git.md                          # Branching strategy, commit conventions, PR rules, .gitignore standards
│   ├── api-design.md                   # REST conventions, versioning, request/response structure, status codes
│   ├── security.md                     # Auth rules, secrets management, input validation, OWASP basics
│   ├── testing.md                      # Unit, integration, e2e test conventions, coverage expectations
│   ├── observability.md                # Logging format, metrics, alerting, error tracking setup
│   ├── code-review.md                  # Pre-commit checklist, untracked file audit, code quality gates, diff review rules
│   └── project-orchestration.md        # Which skills to load and in what order per project type
│
├── frontend/
│   ├── web-ui.md                       # React/Next.js conventions, component structure, routing patterns
│   ├── mobile-ui.md                    # React Native or Flutter conventions, screen structure, navigation
│   ├── state-management.md             # When to use local vs global state, preferred libraries (Zustand, Redux)
│   └── ui-conventions.md               # Platform-specific patterns, iOS/Android design rules, accessibility
│
├── backend/
│   ├── rest-api.md                     # FastAPI conventions, router structure, dependency injection, middleware
│   ├── auth.md                         # JWT, OAuth2, session handling, role-based access control patterns
│   ├── background-jobs.md              # Queue setup, worker conventions, scheduling, retry logic
│   └── error-handling.md               # Global error handlers, custom exceptions, error response format
│
├── database/
│   ├── schema-design.md                # Table naming, indexing rules, constraints, relationship conventions
│   ├── sql.md                          # Query patterns, joins, aggregations, performance anti-patterns to avoid
│   ├── migrations.md                   # Migration tool usage, versioning, rollback strategy, naming conventions
│   └── data-modeling.md                # ER diagrams, normalization rules, choosing relational vs NoSQL
│
├── data-engineering/
│   ├── etl-pipeline.md                 # Extract, transform, load conventions, pipeline structure, error handling
│   ├── orchestration.md                # Airflow/Prefect DAG conventions, scheduling, task dependencies
│   ├── data-quality.md                 # Validation rules, null checks, schema enforcement, data testing
│   ├── warehouse.md                    # dbt project structure, Snowflake/BigQuery conventions, model layering
│   ├── streaming.md                    # Kafka topic conventions, consumer/producer patterns, event schemas
│   └── data-lake/
│       └── .keep                       # Placeholder — structure and conventions TBD when lake is implemented
│
├── data-analytics/
│   ├── data-retrieval.md               # Querying from DBs, APIs, flat files (CSV/Excel/JSON), web scraping basics
│   ├── data-cleaning.md                # Handling nulls, duplicates, outliers, type casting, pandas conventions
│   ├── data-exploration.md             # EDA patterns, descriptive stats, correlation, distribution analysis
│   ├── data-transformation.md          # Reshaping, aggregating, joining datasets, feature engineering basics
│   ├── visualization.md                # Chart selection guide, Matplotlib/Seaborn/Plotly conventions, color rules
│   ├── dashboarding.md                 # Dashboard layout principles, Streamlit/Metabase/Grafana conventions
│   ├── reporting.md                    # Report structure, narrative writing around data, executive summaries
│   ├── metrics-and-kpis.md             # Defining metrics, KPI frameworks, avoiding vanity metrics
│   ├── statistical-analysis.md         # Hypothesis testing, confidence intervals, A/B testing basics
│   ├── spreadsheet.md                  # Excel/Google Sheets conventions, formulas, pivot tables, data entry rules
│   └── notebook-conventions.md         # Jupyter notebook structure, cell ordering, markdown documentation rules
│
├── cloud/
│   ├── aws.md                          # AWS service conventions, IAM rules, preferred services per use case
│   ├── gcp.md                          # GCP service conventions, project structure, preferred services per use case
│   ├── docker.md                       # Dockerfile conventions, image layering, multi-stage builds, compose setup
│   ├── kubernetes.md                   # Cluster setup, deployment manifests, scaling rules, scalability testing
│   ├── ci-cd.md                        # Pipeline structure, build/test/deploy stages, environment promotion rules
│   └── infra-conventions.md            # Env var management, secrets injection, config per environment
│
├── hosting/
│   ├── vercel.md                       # Next.js/frontend deployment conventions, env setup, preview deployments
│   ├── railway.md                      # Backend/fullstack deployment, service linking, database provisioning
│   ├── render.md                       # Alternative to Railway, static sites, web services, cron jobs
│   ├── fly-io.md                       # Containerized app deployment, region selection, scaling configuration
│   ├── supabase.md                     # Backend-as-a-service setup, auth integration, DB and storage usage
│   └── hosting-decision.md             # Decision guide — when to pick which hosting service per project type
│
├── system-architecture/
│   ├── README.md                       # Architecture overview, how to pick an architecture per project type
│   ├── clean-architecture.md           # Layers (entity, use case, adapter, infra), dependency rule, folder structure
│   ├── monolith-vs-microservices.md    # When to use each, tradeoffs, migration path from monolith to services
│   ├── design-patterns.md              # Common patterns (repository, factory, observer, etc.) and when to apply them
│   ├── scalability.md                  # Horizontal vs vertical scaling, caching strategies, load balancing
│   └── adr.md                          # Architecture Decision Record template and when to write one
│
└── afk-workflows/
    ├── feature-sprint.md               # AFK prompt template for developing a full feature end-to-end
    ├── bug-fix.md                      # AFK prompt template for diagnosing and fixing a bug or issue
    └── project-scaffold.md             # AFK prompt template for bootstrapping a new project from scratch
```

