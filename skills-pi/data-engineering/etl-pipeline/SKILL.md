---
name: etl-pipeline
description: Extract, transform, load pipeline conventions including structure, error handling, and retry patterns.
---
# etl-pipeline.md

## Purpose

Define conventions for building Extract, Transform, Load (ETL) pipelines — covering
modular pipeline architecture, stage isolation, error handling, idempotency,
schema contracts, and technology-agnostic patterns that work across Python, Node.js,
and SQL-based pipelines.

---

## Conventions

### Modular Pipeline Architecture

Every pipeline must follow a **three-stage modular structure** where each stage is an
independent, testable module with a clear contract:

```
/pipelines
  /orders_pipeline
    __init__.py
    extract.py          # Data acquisition from sources
    transform.py        # Business logic, cleaning, enrichment
    load.py             # Write to target destination
    schema.py           # Input/output contracts (Pydantic or JSON Schema)
    config.py           # Pipeline-specific configuration
    tests/
      test_extract.py
      test_transform.py
      test_load.py
  /customers_pipeline
    ...
  /shared
    connectors.py       # Reusable source/target connectors
    validators.py       # Shared validation utilities
    logger.py           # Structured logging for pipelines
    retry.py            # Retry and backoff utilities
```

### Stage Contracts

Each stage must define explicit input/output contracts:

```python
# schema.py — Pydantic models as stage contracts
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional
from uuid import UUID

class RawOrder(BaseModel):
    """Contract: output of Extract, input of Transform."""
    external_id: str
    source: str
    raw_payload: dict
    extracted_at: datetime

class CleanOrder(BaseModel):
    """Contract: output of Transform, input of Load."""
    id: UUID
    order_number: str
    customer_email: str
    total_amount: float = Field(ge=0)
    currency: str = Field(pattern=r"^[A-Z]{3}$")
    status: str
    ordered_at: datetime
    transformed_at: datetime

class LoadResult(BaseModel):
    """Contract: output of Load stage."""
    inserted: int
    updated: int
    skipped: int
    errors: int
    duration_seconds: float
```

### Extract Stage

- **Single Responsibility**: Only acquire data — no cleaning, no business logic
- **Source Abstraction**: Use connector classes to abstract source-specific details
- **Pagination**: Always implement pagination for API sources — never fetch unbounded
- **Checkpointing**: Track the last successfully extracted record for incremental pulls
- **Raw Preservation**: Store raw data before any transformation for auditability

```python
# extract.py
from .schema import RawOrder
from shared.connectors import ShopifyConnector
from shared.logger import get_pipeline_logger

logger = get_pipeline_logger("orders.extract")

async def extract_orders(
    connector: ShopifyConnector,
    since: datetime | None = None,
    batch_size: int = 250,
) -> list[RawOrder]:
    """Extract orders from source with pagination and checkpointing."""
    all_orders = []
    page_cursor = None

    while True:
        batch = await connector.fetch_orders(
            since=since,
            limit=batch_size,
            cursor=page_cursor,
        )

        if not batch.data:
            break

        raw_orders = [
            RawOrder(
                external_id=order["id"],
                source="shopify",
                raw_payload=order,
                extracted_at=datetime.utcnow(),
            )
            for order in batch.data
        ]

        all_orders.extend(raw_orders)
        page_cursor = batch.next_cursor
        logger.info(f"Extracted batch: {len(raw_orders)} orders")

    logger.info(f"Extraction complete: {len(all_orders)} total orders")
    return all_orders
```

### Transform Stage

- **Pure Functions**: Transforms should be pure — same input always produces same output
- **No Side Effects**: No API calls, no database writes, no file I/O in transforms
- **Row-Level Processing**: Process each record independently when possible
- **Error Isolation**: A single bad record must never crash the entire transform
- **Enrich, Don't Mutate**: Create new fields instead of overwriting raw data

```python
# transform.py
from .schema import RawOrder, CleanOrder
from shared.logger import get_pipeline_logger

logger = get_pipeline_logger("orders.transform")

def transform_order(raw: RawOrder) -> CleanOrder | None:
    """Transform a single raw order to a clean order. Returns None on failure."""
    try:
        payload = raw.raw_payload
        return CleanOrder(
            id=uuid5(NAMESPACE_URL, raw.external_id),
            order_number=payload["order_number"],
            customer_email=payload["email"].strip().lower(),
            total_amount=float(payload["total_price"]),
            currency=payload["currency"].upper(),
            status=_normalize_status(payload["financial_status"]),
            ordered_at=parse_datetime(payload["created_at"]),
            transformed_at=datetime.utcnow(),
        )
    except (KeyError, ValueError, ValidationError) as e:
        logger.warning(f"Transform failed for {raw.external_id}: {e}")
        return None


def transform_orders(raws: list[RawOrder]) -> tuple[list[CleanOrder], list[RawOrder]]:
    """Batch transform with error segregation."""
    clean, failed = [], []
    for raw in raws:
        result = transform_order(raw)
        if result:
            clean.append(result)
        else:
            failed.append(raw)

    logger.info(f"Transform complete: {len(clean)} clean, {len(failed)} failed")
    return clean, failed
```

### Load Stage

- **Idempotency**: Every load must be idempotent — re-running produces the same result
- **Upsert Pattern**: Use `INSERT ... ON CONFLICT UPDATE` instead of blind inserts
- **Batch Writes**: Write in configurable batches — never one-by-one, never all-at-once
- **Transaction Safety**: Wrap batch writes in transactions with rollback on failure
- **Load Reporting**: Always return a `LoadResult` with counts

```python
# load.py
from .schema import CleanOrder, LoadResult
from shared.logger import get_pipeline_logger

logger = get_pipeline_logger("orders.load")

async def load_orders(
    db: AsyncSession,
    orders: list[CleanOrder],
    batch_size: int = 500,
) -> LoadResult:
    """Upsert orders in batches with transaction safety."""
    inserted, updated, skipped, errors = 0, 0, 0, 0
    start = time.monotonic()

    for batch in chunked(orders, batch_size):
        try:
            async with db.begin():
                for order in batch:
                    result = await upsert_order(db, order)
                    match result:
                        case "inserted": inserted += 1
                        case "updated": updated += 1
                        case "skipped": skipped += 1
        except Exception as e:
            errors += len(batch)
            logger.error(f"Batch load failed: {e}")

    duration = time.monotonic() - start
    logger.info(f"Load complete: +{inserted} ~{updated} -{skipped} !{errors}")

    return LoadResult(
        inserted=inserted,
        updated=updated,
        skipped=skipped,
        errors=errors,
        duration_seconds=round(duration, 2),
    )
```

### Pipeline Orchestration (Runner)

- **Pipeline Runner**: A single entry point that chains Extract → Transform → Load
- **Dry Run Mode**: Support `--dry-run` that runs Extract + Transform but skips Load
- **Retry Logic**: Wrap each stage in retry with exponential backoff
- **Metrics Emission**: Log start, end, duration, and record counts per stage

```python
# __init__.py — Pipeline runner
from .extract import extract_orders
from .transform import transform_orders
from .load import load_orders
from shared.logger import get_pipeline_logger

logger = get_pipeline_logger("orders")

async def run_orders_pipeline(
    config: PipelineConfig,
    dry_run: bool = False,
) -> PipelineResult:
    """Execute the full ETL pipeline."""
    logger.info(f"Pipeline started | dry_run={dry_run}")

    # 1. Extract
    raw_orders = await extract_orders(config.connector, since=config.since)

    # 2. Transform
    clean_orders, failed_orders = transform_orders(raw_orders)

    # 3. Load (skip if dry run)
    if dry_run:
        logger.info("Dry run — skipping load stage")
        load_result = LoadResult(inserted=0, updated=0, skipped=0, errors=0, duration_seconds=0)
    else:
        load_result = await load_orders(config.db, clean_orders)

    return PipelineResult(
        extracted=len(raw_orders),
        transformed=len(clean_orders),
        failed=len(failed_orders),
        load=load_result,
    )
```

### Error Handling Strategy

| Error Type | Handling | Example |
|---|---|---|
| Source unavailable | Retry with backoff → alert after max retries | API timeout, DB connection refused |
| Schema validation | Isolate bad record → continue pipeline → log to dead letter | Missing required field |
| Transform logic | Isolate bad record → log with context → continue | Unexpected date format |
| Load conflict | Upsert resolution → log decision | Duplicate primary key |
| Load batch failure | Rollback batch → retry once → dead letter on second failure | Transaction deadlock |
| Pipeline crash | Checkpoint → restart from last checkpoint | Out of memory |

### Dead Letter Queue (DLQ)

Records that fail processing must never be silently dropped:

```python
# shared/dead_letter.py
async def send_to_dlq(
    pipeline: str,
    stage: str,
    record: dict,
    error: str,
    timestamp: datetime,
) -> None:
    """Route failed records to dead letter storage for manual review."""
    dlq_entry = {
        "pipeline": pipeline,
        "stage": stage,
        "record": record,
        "error": str(error),
        "failed_at": timestamp.isoformat(),
    }
    # Write to DLQ table, file, or message queue
    await dlq_store.insert(dlq_entry)
```

### Incremental vs Full Load Decision

| Scenario | Strategy | Checkpoint |
|---|---|---|
| First run ever | Full load | None |
| Source supports `updated_since` | Incremental by timestamp | `last_extracted_at` |
| Source has event stream | CDC (Change Data Capture) | Stream offset |
| Data corrections needed | Full reload of affected partition | Partition key |
| Schema change in source | Full reload | None |

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Pipeline directory | `snake_case` + `_pipeline` | `orders_pipeline` |
| Stage files | Stage name only | `extract.py`, `transform.py`, `load.py` |
| Schema models | `Raw` + entity (extract output), `Clean` + entity (transform output) | `RawOrder`, `CleanOrder` |
| Config | `PipelineConfig` class | `OrdersPipelineConfig` |
| Log names | Dot-separated: `pipeline.stage` | `orders.extract` |

---

## Anti-Patterns

- Never mix extraction logic with transformation logic in the same function
- Never perform API calls or database writes inside the transform stage
- Never load records one-by-one — always batch writes
- Never silently drop failed records — route to DLQ with full context
- Never skip schema validation between stages — contracts prevent cascading failures
- Never hardcode source credentials — use environment variables via config
- Never run without idempotency — re-running a pipeline must be safe
- Never assume source data is clean — validate everything at extract boundaries
- Never build a pipeline without a dry-run mode
- Never skip logging stage boundaries (start, count, duration, errors)

---

## Cross-References

- **data-engineering/data-quality** → validation rules applied between stages
- **data-engineering/orchestration** → scheduling and dependency management for pipelines
- **data-engineering/warehouse** → target schema and model layering for load stage
- **database/schema-design** → target table conventions for the load destination
- **core/observability** → structured logging and metrics for pipeline monitoring
- **core/testing** → unit tests for each stage independently

---

## Ready-to-Use Prompt

```
Task: Build ETL pipeline for [data source] → [target destination]
Skill: data-engineering/etl-pipeline, data-engineering/data-quality, data-engineering/orchestration

REQUIREMENTS:
- Source: [API / database / file / event stream]
- Target: [warehouse / database / data lake]
- Load strategy: [full / incremental / CDC]
- Schema: [describe key entities and fields]

CONSTRAINTS:
- Modular three-stage architecture: extract.py / transform.py / load.py
- Pydantic schema contracts between every stage
- Pure transform functions — no side effects
- Idempotent loads via upsert pattern
- Dead letter queue for failed records
- Batch writes with configurable batch size
- Structured logging at stage boundaries
- Dry-run mode supported
- Retry with exponential backoff on source failures

IF BLOCKED:
- Document blocker as TODO comment
- Log reasoning for non-obvious design decisions

DONE WHEN:
- All three stages implemented and independently testable
- Schema contracts validated between stages
- Error handling covers all failure types in the strategy table
- DLQ captures all failed records with context
- Pipeline runner chains stages with metrics logging
- Unit tests cover happy path + edge cases per stage
- Code review skill applied before commit
```

