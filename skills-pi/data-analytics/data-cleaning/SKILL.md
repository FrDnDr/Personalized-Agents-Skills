---
name: data-cleaning
description: Handling nulls, duplicates, outliers, type casting, and pandas data cleaning conventions.
---
# data-cleaning.md

## Purpose

Define conventions for cleaning and preparing data — covering null handling,
duplicate detection, outlier treatment, type casting, string normalization,
and pandas best practices for reproducible data cleaning pipelines.

---

## Conventions

### Cleaning Pipeline Order

Always clean in this sequence to avoid cascading issues:

```
1. Structural fixes     → column names, types, shapes
2. Deduplication         → remove exact and fuzzy duplicates
3. Null handling         → impute, fill, or drop
4. Type casting          → enforce correct dtypes
5. String normalization  → trim, case, encoding
6. Outlier treatment     → detect and handle extreme values
7. Validation            → assert data meets expected state
```

### Column Name Standardization

```python
def standardize_columns(df: pd.DataFrame) -> pd.DataFrame:
    """Standardize all column names to snake_case."""
    df.columns = (
        df.columns
        .str.strip()
        .str.lower()
        .str.replace(r'[^a-z0-9]+', '_', regex=True)
        .str.strip('_')
    )
    return df
```

### Duplicate Handling

```python
def remove_duplicates(
    df: pd.DataFrame,
    subset: list[str] | None = None,
    keep: str = "last",
    log: bool = True,
) -> pd.DataFrame:
    """Remove duplicates with logging."""
    before = len(df)
    df = df.drop_duplicates(subset=subset, keep=keep)
    after = len(df)
    if log and before != after:
        logger.info(f"Removed {before - after} duplicates ({before} → {after})")
    return df
```

**Rules:**
- Always specify `subset` — full-row deduplication is rarely correct
- Always specify `keep` (`"first"`, `"last"`, or `False`) explicitly
- Always log the count of removed duplicates

### Null Handling Strategy

| Column Type | Strategy | Method |
|---|---|---|
| Required identifier | Drop row | `df.dropna(subset=["id"])` |
| Numeric (continuous) | Impute with median | `df["col"].fillna(df["col"].median())` |
| Numeric (count) | Fill with 0 | `df["col"].fillna(0)` |
| Categorical | Fill with mode or "Unknown" | `df["col"].fillna("Unknown")` |
| Boolean | Fill with False | `df["col"].fillna(False)` |
| Timestamp | Forward-fill or drop | `df["col"].ffill()` |

```python
def handle_nulls(df: pd.DataFrame, rules: dict[str, str]) -> pd.DataFrame:
    """Apply null handling rules per column."""
    for col, strategy in rules.items():
        if col not in df.columns:
            continue
        match strategy:
            case "drop":
                df = df.dropna(subset=[col])
            case "zero":
                df[col] = df[col].fillna(0)
            case "median":
                df[col] = df[col].fillna(df[col].median())
            case "mode":
                df[col] = df[col].fillna(df[col].mode().iloc[0] if not df[col].mode().empty else "Unknown")
            case "unknown":
                df[col] = df[col].fillna("Unknown")
            case "ffill":
                df[col] = df[col].ffill()
            case _:
                raise ValueError(f"Unknown strategy: {strategy}")
    return df
```

### Type Casting

```python
def enforce_types(df: pd.DataFrame, type_map: dict[str, str]) -> pd.DataFrame:
    """Cast columns to specified types with error handling."""
    for col, dtype in type_map.items():
        if col not in df.columns:
            continue
        try:
            if dtype == "datetime":
                df[col] = pd.to_datetime(df[col], errors="coerce")
            elif dtype == "numeric":
                df[col] = pd.to_numeric(df[col], errors="coerce")
            elif dtype == "category":
                df[col] = df[col].astype("category")
            else:
                df[col] = df[col].astype(dtype)
        except (ValueError, TypeError) as e:
            logger.warning(f"Type cast failed for {col} → {dtype}: {e}")
    return df
```

**Rules:**
- Always use `errors="coerce"` for datetime and numeric — produces NaN instead of crash
- Always cast after null handling — NaN-safe types only
- Always use `category` dtype for low-cardinality string columns (saves memory)

### String Normalization

```python
def normalize_strings(df: pd.DataFrame, columns: list[str]) -> pd.DataFrame:
    """Trim whitespace and standardize case."""
    for col in columns:
        if col in df.columns and df[col].dtype == "object":
            df[col] = df[col].str.strip().str.lower()
    return df

def normalize_emails(df: pd.DataFrame, col: str = "email") -> pd.DataFrame:
    """Standardize email format."""
    df[col] = df[col].str.strip().str.lower()
    return df
```

### Outlier Detection and Treatment

```python
def detect_outliers_iqr(
    df: pd.DataFrame, column: str, multiplier: float = 1.5
) -> pd.Series:
    """Detect outliers using IQR method. Returns boolean mask."""
    q1, q3 = df[column].quantile([0.25, 0.75])
    iqr = q3 - q1
    lower, upper = q1 - multiplier * iqr, q3 + multiplier * iqr
    return (df[column] < lower) | (df[column] > upper)

def handle_outliers(
    df: pd.DataFrame, column: str, strategy: str = "clip"
) -> pd.DataFrame:
    """Handle outliers: clip, remove, or flag."""
    outliers = detect_outliers_iqr(df, column)
    match strategy:
        case "clip":
            q1, q3 = df[column].quantile([0.25, 0.75])
            iqr = q3 - q1
            df[column] = df[column].clip(lower=q1 - 1.5 * iqr, upper=q3 + 1.5 * iqr)
        case "remove":
            df = df[~outliers]
        case "flag":
            df[f"{column}_is_outlier"] = outliers
    return df
```

| Strategy | Use Case | Impact |
|---|---|---|
| Clip | Preserve all rows, cap extremes | Modifies values |
| Remove | Outliers are errors/noise | Reduces row count |
| Flag | Keep for analysis but mark | Adds column |

### Cleaning Report

Always produce a summary after cleaning:

```python
def cleaning_report(df_before: pd.DataFrame, df_after: pd.DataFrame) -> dict:
    return {
        "rows_before": len(df_before),
        "rows_after": len(df_after),
        "rows_removed": len(df_before) - len(df_after),
        "null_counts_after": df_after.isnull().sum().to_dict(),
        "dtypes_after": df_after.dtypes.astype(str).to_dict(),
    }
```

---

## Anti-Patterns

- Never drop nulls without first understanding why they exist
- Never impute with mean for skewed distributions — use median
- Never apply global deduplication without specifying subset columns
- Never cast types before handling nulls — NaN conversion must happen first
- Never modify the original DataFrame — always return a new copy or use `.copy()`
- Never skip logging the count of changes (drops, fills, casts)
- Never assume string columns are clean — always strip and normalize

---

## Cross-References

- **data-analytics/data-retrieval** → raw data arrives from connectors needing cleaning
- **data-analytics/data-exploration** → EDA reveals what cleaning is needed
- **data-analytics/data-transformation** → cleaned data feeds into transformations
- **data-engineering/data-quality** → validation checks confirm cleaning was effective

---

## Ready-to-Use Prompt

```
Task: Clean [dataset name] for [analysis purpose]
Skill: data-analytics/data-cleaning, data-analytics/data-exploration

REQUIREMENTS:
- Dataset: [describe source and shape]
- Known issues: [nulls, duplicates, outliers, type problems]
- Target state: [describe expected clean state]

CONSTRAINTS:
- Follow cleaning pipeline order (structure → dedup → nulls → types → strings → outliers)
- Use strategy tables for null handling and outlier treatment
- Log all changes with counts
- Produce cleaning report after completion
- Never modify original data — work on copy

DONE WHEN:
- All nulls handled per strategy table
- Duplicates removed with logged counts
- Types correctly cast
- Strings normalized
- Outliers treated per use case
- Cleaning report generated
- Code review skill applied before commit
```

