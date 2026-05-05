---
name: orchestration
description: Airflow and Prefect DAG conventions including scheduling, task dependencies, and failure handling.
---
# orchestration.md

## Purpose

Define conventions for orchestrating data pipelines — covering DAG design,
task dependency management, scheduling strategies, failure handling, and
modular patterns for Airflow, Prefect, and lightweight cron-based orchestration.

---

## Conventions

### Orchestrator Selection

| Scenario | Recommended Tool | Rationale |
|---|---|---|
| Simple scheduled scripts | Cron + wrapper script | Minimal overhead, ≤5 pipelines |
| Medium complexity (10–30) | Prefect | Pythonic API, good observability |
| Enterprise / complex deps | Apache Airflow | Industry standard, rich ecosystem |
| Event-driven pipelines | Prefect + event triggers | Native event support |
| Cloud-native | Cloud Composer / MWAA | Managed Airflow, reduced ops |

### Modular DAG Architecture

Logic lives in pipeline modules, not in DAG files:

```
/dags
  orders_dag.py            # DAG definition only — no business logic
  customers_dag.py
/pipelines                  # Business logic (from etl-pipeline.md)
  /orders_pipeline
    extract.py / transform.py / load.py
/config
  dag_config.yaml           # Centralized DAG configuration
/plugins
  /operators                # Custom reusable operators
  /sensors                  # Custom sensors for external triggers
```

### DAG Definition (Airflow)

- DAG files define flow only — import pipeline logic from `/pipelines`
- One DAG per pipeline domain
- Naming: `{domain}_{frequency}_{action}` → `orders_daily_sync`
- Always set `catchup=False` unless historical backfill is needed
- Always set `max_active_runs=1` to prevent overlap
- Always define `default_args` with owner, retries, retry delay

```python
# dags/orders_dag.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.utils.dates import days_ago
from datetime import timedelta
from pipelines.orders_pipeline import extract_orders, transform_orders, load_orders

default_args = {
    "owner": "data-engineering",
    "depends_on_past": False,
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,
    "email_on_failure": True,
}

with DAG(
    dag_id="orders_daily_sync",
    default_args=default_args,
    schedule_interval="0 6 * * *",
    start_date=days_ago(1),
    catchup=False,
    max_active_runs=1,
    tags=["orders", "daily", "shopify"],
) as dag:
    extract = PythonOperator(task_id="extract_orders", python_callable=extract_orders)
    transform = PythonOperator(task_id="transform_orders", python_callable=transform_orders)
    load = PythonOperator(task_id="load_orders", python_callable=load_orders)
    extract >> transform >> load
```

### DAG Definition (Prefect)

```python
from prefect import flow, task
from datetime import timedelta
from pipelines.orders_pipeline import extract_orders, transform_orders, load_orders

@task(name="extract-orders", retries=3, retry_delay_seconds=[60, 300, 900])
def extract_task(source: str):
    return extract_orders(source=source)

@task(name="transform-orders")
def transform_task(raw_orders: list):
    return transform_orders(raw_orders)

@task(name="load-orders", retries=2)
def load_task(clean_orders: list):
    return load_orders(clean_orders)

@flow(name="orders-daily-sync", retries=1, log_prints=True)
def orders_pipeline(source: str = "shopify"):
    raw = extract_task(source)
    clean, failed = transform_task(raw)
    return load_task(clean)
```

### Task Dependency Patterns

| Pattern | Use Case | Airflow | Prefect |
|---|---|---|---|
| Linear chain | Sequential stages | `A >> B >> C` | Return values |
| Fan-out | Parallel extraction | `A >> [B1, B2] >> C` | `map()` |
| Fan-in | Merge streams | `[A1, A2] >> B` | `wait_for` |
| Conditional | Skip stages | `BranchPythonOperator` | `if/else` in flow |
| Cross-DAG | Pipeline dependency | `ExternalTaskSensor` | Flow of flows |

### Scheduling Conventions

| Pipeline Type | Schedule | Rationale |
|---|---|---|
| Transactional sync | `0 */4 * * *` | Balance freshness vs rate limits |
| Daily aggregation | `0 6 * * *` | After source batch close |
| Weekly reports | `0 8 * * 1` | Monday start of business |
| Real-time events | Event-triggered | Streaming or webhooks |
| Monthly snapshots | `0 2 1 * *` | After month-end close |

### Failure Handling

- Always configure `on_failure_callback` for alerting
- Always define `execution_timeout` — never let tasks run forever
- Define DAG-level SLA for end-to-end completion
- Alert on SLA breach — don't just log it

### Configuration Management

Use YAML config — never hardcode parameters in DAG files:

```yaml
# config/dag_config.yaml
orders_daily_sync:
  schedule: "0 6 * * *"
  source: shopify
  batch_size: 250
  max_retries: 3
  timeout_minutes: 60
```

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| DAG ID | `{domain}_{frequency}_{action}` | `orders_daily_sync` |
| Task ID | `{verb}_{entity}` | `extract_orders` |
| DAG file | `{domain}_dag.py` | `orders_dag.py` |
| Flow name | kebab-case | `orders-daily-sync` |
| Tags | Lowercase, descriptive | `["orders", "daily"]` |

---

## Anti-Patterns

- Never put business logic in DAG files — import from pipeline modules
- Never use `catchup=True` without explicit backfill requirements
- Never run DAGs without `max_active_runs` limit
- Never skip `execution_timeout` — runaway tasks consume resources indefinitely
- Never hardcode schedules or config in DAG files — use YAML or Airflow Variables
- Never create monolithic DAGs with 50+ tasks — decompose by domain
- Never ignore task failures silently — always configure failure callbacks
- Never schedule heavy pipelines during business hours without resource isolation

---

## Cross-References

- **data-engineering/etl-pipeline** → pipeline modules that DAGs orchestrate
- **data-engineering/data-quality** → validation tasks added as DAG stages
- **core/observability** → logging and alerting for pipeline monitoring

---

## Ready-to-Use Prompt

```
Task: Create orchestration for [pipeline name]
Skill: data-engineering/orchestration, data-engineering/etl-pipeline

REQUIREMENTS:
- Orchestrator: [Airflow / Prefect / cron]
- Schedule: [cron expression or event-triggered]
- Stages: [list pipeline stages and dependencies]

CONSTRAINTS:
- DAG files contain flow definition only — no business logic
- Import logic from /pipelines modules
- catchup=False, max_active_runs=1
- Define retries, retry_delay, execution_timeout
- Configure on_failure_callback for alerting
- Externalize config to YAML

DONE WHEN:
- DAG defined with explicit task dependencies
- All tasks have timeout and retry configuration
- Failure callbacks configured
- Config externalized
- Code review skill applied before commit
```

