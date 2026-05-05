---
name: warehouse
description: dbt project structure and Snowflake or BigQuery conventions including model layering standards.
---
# warehouse.md

## Purpose

Define conventions for data warehouse architecture — covering dbt project structure,
model layering (staging → intermediate → mart), Snowflake/BigQuery conventions,
materializations, testing, and documentation standards.

---

## Conventions

### Warehouse Model Layering

Use a **three-layer architecture** to separate raw ingestion from business logic:

```
/models
  /staging         # 1:1 with source tables — rename, cast, deduplicate
    /shopify
      stg_shopify__orders.sql
      stg_shopify__customers.sql
      _shopify__sources.yml
    /stripe
      stg_stripe__payments.sql
      _stripe__sources.yml
  /intermediate    # Business logic joins, calculations, enrichment
    int_orders__enriched.sql
    int_customer__lifetime_value.sql
  /marts           # Final, consumer-facing models by department
    /finance
      fct_revenue.sql
      dim_customers.sql
    /marketing
      fct_campaign_performance.sql
    /operations
      fct_fulfillment.sql
  /utilities       # Reusable macros and date spines
    date_spine.sql
```

### Layer Responsibilities

| Layer | Purpose | Materialization | Naming |
|---|---|---|---|
| Staging | 1:1 source mirror — rename, cast, deduplicate | View | `stg_{source}__{entity}` |
| Intermediate | Business logic — joins, calculations | View or Ephemeral | `int_{entity}__{verb}` |
| Marts | Consumer-facing — fact & dimension tables | Table or Incremental | `fct_{noun}` / `dim_{noun}` |

### Staging Model Conventions

- One staging model per source table — never combine sources in staging
- Only rename columns, cast types, and deduplicate — no business logic
- Always add `_loaded_at` timestamp for lineage tracking

```sql
-- models/staging/shopify/stg_shopify__orders.sql
with source as (
    select * from {{ source('shopify', 'orders') }}
),

renamed as (
    select
        id                          as order_id,
        cast(order_number as varchar) as order_number,
        email                       as customer_email,
        cast(total_price as decimal(10,2)) as total_amount,
        upper(currency)             as currency_code,
        financial_status            as payment_status,
        cast(created_at as timestamp) as ordered_at,
        cast(_etl_loaded_at as timestamp) as _loaded_at
    from source
)

select * from renamed
```

### Intermediate Model Conventions

- Joins between staging models happen here — never in marts
- Complex calculations and business rule application
- Can reference staging and other intermediate models

```sql
-- models/intermediate/int_orders__enriched.sql
with orders as (
    select * from {{ ref('stg_shopify__orders') }}
),

payments as (
    select * from {{ ref('stg_stripe__payments') }}
),

enriched as (
    select
        o.order_id,
        o.order_number,
        o.customer_email,
        o.total_amount,
        o.currency_code,
        p.payment_method,
        p.paid_at,
        case
            when p.paid_at is not null then 'paid'
            when o.ordered_at < current_timestamp - interval '7 days' then 'overdue'
            else 'pending'
        end as payment_status
    from orders o
    left join payments p on o.order_id = p.order_id
)

select * from enriched
```

### Mart Model Conventions

- Mart models are the only layer consumed by BI tools and analysts
- Follow **star schema**: fact tables (events/transactions) + dimension tables (entities)
- Always include surrogate keys for dimension tables

```sql
-- models/marts/finance/fct_revenue.sql
{{ config(materialized='incremental', unique_key='order_id') }}

with orders as (
    select * from {{ ref('int_orders__enriched') }}
    {% if is_incremental() %}
    where ordered_at > (select max(ordered_at) from {{ this }})
    {% endif %}
)

select
    order_id,
    order_number,
    customer_email,
    total_amount,
    currency_code,
    payment_status,
    payment_method,
    ordered_at,
    current_timestamp as _dbt_loaded_at
from orders
where payment_status = 'paid'
```

### Materialization Decision

| Model Type | Default | Switch To | When |
|---|---|---|---|
| Staging | View | Table | Source >1M rows and slow queries |
| Intermediate | Ephemeral | View | Needs to be debuggable / reused |
| Fact tables | Table | Incremental | >10M rows, append-mostly |
| Dimension tables | Table | Table | Always full refresh |
| Aggregations | Table | Incremental | Time-series data |

### dbt Testing Conventions

```yaml
# models/staging/shopify/_shopify__sources.yml
version: 2
sources:
  - name: shopify
    schema: raw_shopify
    tables:
      - name: orders
        columns:
          - name: id
            tests: [unique, not_null]
          - name: email
            tests: [not_null]
          - name: total_price
            tests:
              - not_null
              - dbt_expectations.expect_column_values_to_be_between:
                  min_value: 0
```

### dbt Project Structure

```yaml
# dbt_project.yml
name: 'analytics_warehouse'
version: '1.0.0'

models:
  analytics_warehouse:
    staging:
      +materialized: view
      +schema: staging
    intermediate:
      +materialized: ephemeral
    marts:
      +materialized: table
      finance:
        +schema: finance
      marketing:
        +schema: marketing
```

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Staging models | `stg_{source}__{entity}` | `stg_shopify__orders` |
| Intermediate | `int_{entity}__{verb}` | `int_orders__enriched` |
| Fact tables | `fct_{noun}` | `fct_revenue` |
| Dimension tables | `dim_{noun}` | `dim_customers` |
| Sources YAML | `_{source}__sources.yml` | `_shopify__sources.yml` |
| Macros | `{verb}_{noun}` | `calculate_lifetime_value` |

---

## Anti-Patterns

- Never put business logic in staging models — only rename, cast, deduplicate
- Never join across sources in the staging layer — do it in intermediate
- Never let BI tools query staging or intermediate models — marts only
- Never skip dbt tests on primary keys — always test `unique` + `not_null`
- Never use `SELECT *` in mart models — always explicit column lists
- Never materialize everything as tables — follow the decision matrix
- Never skip documentation (YAML) for mart models — they are consumer-facing
- Never create one giant mart model — separate facts from dimensions

---

## Cross-References

- **data-engineering/etl-pipeline** → pipeline loads raw data that staging models consume
- **data-engineering/data-quality** → dbt tests enforce quality on warehouse models
- **database/schema-design** → target table conventions for warehouse schemas
- **data-analytics/dashboarding** → BI tools consume mart models

---

## Ready-to-Use Prompt

```
Task: Build warehouse models for [domain/feature]
Skill: data-engineering/warehouse, data-engineering/data-quality

REQUIREMENTS:
- Sources: [list raw source tables]
- Marts needed: [list fact and dimension tables]
- Materialization: [view / table / incremental]

CONSTRAINTS:
- Three-layer architecture: staging → intermediate → marts
- Staging: rename, cast, deduplicate only
- Intermediate: joins and business logic
- Marts: star schema — facts + dimensions
- dbt tests on all primary keys (unique + not_null)
- YAML documentation for all mart models
- Follow naming conventions for all layers

DONE WHEN:
- All layers implemented with correct materialization
- dbt tests passing on all models
- YAML documentation complete for consumer-facing models
- DAG lineage is clean (no circular references)
- Code review skill applied before commit
```

