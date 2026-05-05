---
name: data-quality
description: Data validation rules, null checks, schema enforcement, and data testing patterns.
---
# data-quality.md

## Purpose

Define conventions for validating data at every stage of a pipeline — covering
schema enforcement, null checks, uniqueness validation, freshness checks,
anomaly detection, and automated data testing with Great Expectations and custom validators.

---

## Conventions

### Validation Architecture

Data quality checks operate as **gates between pipeline stages**:

```
Extract → [VALIDATE RAW] → Transform → [VALIDATE CLEAN] → Load → [VALIDATE TARGET]
```

```
/shared
  /validators
    base.py              # Abstract validator interface
    schema_validator.py  # Schema enforcement checks
    row_validator.py     # Row-level field validation
    dataset_validator.py # Dataset-level aggregate checks
    freshness.py         # Staleness and timeliness checks
  /expectations          # Great Expectations suite definitions
    orders_suite.json
    customers_suite.json
```

### Validator Interface

All validators implement a common interface for composability:

```python
# shared/validators/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from enum import Enum

class Severity(Enum):
    WARNING = "warning"    # Log and continue
    ERROR = "error"        # Isolate record, continue pipeline
    CRITICAL = "critical"  # Halt pipeline immediately

@dataclass
class ValidationResult:
    check_name: str
    passed: bool
    severity: Severity
    message: str
    failed_records: int = 0
    total_records: int = 0
    metadata: dict = field(default_factory=dict)

class BaseValidator(ABC):
    @abstractmethod
    def validate(self, data) -> list[ValidationResult]:
        """Run all checks and return results."""
        pass
```

### Check Categories

#### 1. Schema Checks (Structure)

Validate that data matches the expected schema before processing:

```python
# shared/validators/schema_validator.py
def validate_schema(df: pd.DataFrame, expected_columns: dict) -> ValidationResult:
    """Ensure DataFrame matches expected column names and types."""
    missing = set(expected_columns.keys()) - set(df.columns)
    extra = set(df.columns) - set(expected_columns.keys())
    type_mismatches = []

    for col, expected_type in expected_columns.items():
        if col in df.columns and not pd.api.types.is_dtype_equal(df[col].dtype, expected_type):
            type_mismatches.append(f"{col}: expected {expected_type}, got {df[col].dtype}")

    passed = not missing and not type_mismatches
    return ValidationResult(
        check_name="schema_validation",
        passed=passed,
        severity=Severity.CRITICAL,
        message=f"Missing: {missing}, Type mismatches: {type_mismatches}",
        metadata={"missing": list(missing), "extra": list(extra)},
    )
```

#### 2. Completeness Checks (Nulls)

```python
def validate_not_null(df: pd.DataFrame, required_columns: list[str]) -> list[ValidationResult]:
    """Check that required columns have no null values."""
    results = []
    for col in required_columns:
        null_count = df[col].isnull().sum()
        results.append(ValidationResult(
            check_name=f"not_null_{col}",
            passed=null_count == 0,
            severity=Severity.ERROR,
            message=f"{col}: {null_count} null values found",
            failed_records=int(null_count),
            total_records=len(df),
        ))
    return results
```

#### 3. Uniqueness Checks (Duplicates)

```python
def validate_unique(df: pd.DataFrame, unique_columns: list[str]) -> ValidationResult:
    """Ensure specified columns form a unique key."""
    duplicates = df.duplicated(subset=unique_columns, keep=False)
    dup_count = duplicates.sum()
    return ValidationResult(
        check_name=f"unique_{'_'.join(unique_columns)}",
        passed=dup_count == 0,
        severity=Severity.ERROR,
        message=f"{dup_count} duplicate rows on columns {unique_columns}",
        failed_records=int(dup_count),
        total_records=len(df),
    )
```

#### 4. Range and Value Checks

```python
def validate_range(df: pd.DataFrame, column: str, min_val=None, max_val=None) -> ValidationResult:
    """Ensure values fall within expected range."""
    violations = pd.Series([False] * len(df))
    if min_val is not None:
        violations |= df[column] < min_val
    if max_val is not None:
        violations |= df[column] > max_val

    fail_count = violations.sum()
    return ValidationResult(
        check_name=f"range_{column}",
        passed=fail_count == 0,
        severity=Severity.ERROR,
        message=f"{column}: {fail_count} values outside [{min_val}, {max_val}]",
        failed_records=int(fail_count),
        total_records=len(df),
    )
```

#### 5. Freshness Checks

```python
def validate_freshness(
    latest_timestamp: datetime,
    max_staleness: timedelta,
) -> ValidationResult:
    """Ensure data is not stale beyond acceptable threshold."""
    age = datetime.utcnow() - latest_timestamp
    return ValidationResult(
        check_name="data_freshness",
        passed=age <= max_staleness,
        severity=Severity.WARNING,
        message=f"Data age: {age}, max allowed: {max_staleness}",
        metadata={"age_seconds": age.total_seconds()},
    )
```

#### 6. Volume Checks (Row Count Anomalies)

```python
def validate_volume(
    current_count: int,
    expected_count: int,
    tolerance_pct: float = 0.2,
) -> ValidationResult:
    """Detect abnormal data volume — too many or too few records."""
    lower = expected_count * (1 - tolerance_pct)
    upper = expected_count * (1 + tolerance_pct)
    passed = lower <= current_count <= upper

    return ValidationResult(
        check_name="volume_check",
        passed=passed,
        severity=Severity.WARNING,
        message=f"Count: {current_count}, expected: {expected_count} ±{tolerance_pct*100}%",
        total_records=current_count,
    )
```

### Severity-Based Response

| Severity | Pipeline Action | Notification |
|---|---|---|
| WARNING | Continue pipeline, log result | Log + optional Slack |
| ERROR | Isolate failed records to DLQ, continue | Log + Slack alert |
| CRITICAL | Halt pipeline immediately | Log + Slack + PagerDuty |

### Validation Runner

Compose multiple validators into a single run:

```python
def run_validation_suite(
    df: pd.DataFrame,
    suite: list[callable],
) -> tuple[bool, list[ValidationResult]]:
    """Run all validations and return aggregate pass/fail."""
    all_results = []
    for check in suite:
        results = check(df) if isinstance(check(df), list) else [check(df)]
        all_results.extend(results)

    has_critical = any(r.severity == Severity.CRITICAL and not r.passed for r in all_results)
    has_error = any(r.severity == Severity.ERROR and not r.passed for r in all_results)

    passed = not has_critical
    return passed, all_results
```

---

## Anti-Patterns

- Never skip validation between pipeline stages — schema drift causes silent corruption
- Never treat all validation failures equally — use severity levels
- Never halt pipelines on WARNING-level issues — log and continue
- Never validate only at the end — catch issues as early as possible
- Never hardcode validation thresholds — make them configurable
- Never silently drop records that fail validation — route to DLQ
- Never skip volume checks — sudden drops or spikes indicate upstream issues

---

## Cross-References

- **data-engineering/etl-pipeline** → validators run between ETL stages
- **data-engineering/warehouse** → target-side validation after load
- **core/testing** → unit tests for validator logic itself

---

## Ready-to-Use Prompt

```
Task: Add data quality checks to [pipeline name]
Skill: data-engineering/data-quality, data-engineering/etl-pipeline

REQUIREMENTS:
- Checks needed: [schema / nulls / uniqueness / range / freshness / volume]
- Severity levels: [define which checks are WARNING vs ERROR vs CRITICAL]
- DLQ routing: [yes / no]

CONSTRAINTS:
- Validators implement BaseValidator interface
- Compose checks via validation runner
- Severity determines pipeline response (continue / isolate / halt)
- All thresholds configurable, not hardcoded
- Log all results with structured logging

DONE WHEN:
- All required checks implemented and passing
- Severity levels correctly assigned
- Failed records routed to DLQ (if applicable)
- Validation runner composes all checks
- Unit tests cover edge cases
- Code review skill applied before commit
```

