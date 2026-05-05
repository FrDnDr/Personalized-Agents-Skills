---
name: data-transformation
description: Reshaping, aggregating, joining datasets, and feature engineering basics using pandas and SQL.
---
# data-transformation.md

## Purpose

Define conventions for reshaping, aggregating, joining, and engineering features
from cleaned data — covering pandas transformation patterns, groupby operations,
pivot/melt operations, window functions, and feature engineering basics.

---

## Conventions

### Transformation Pipeline

```
Cleaned Data → Reshape → Join/Merge → Aggregate → Feature Engineer → Output
```

### Reshaping Operations

#### Pivot (Long → Wide)

```python
# Convert long-format to wide-format
revenue_by_month = df.pivot_table(
    values="revenue",
    index="customer_id",
    columns="month",
    aggfunc="sum",
    fill_value=0,
)
```

#### Melt (Wide → Long)

```python
# Convert wide-format to long-format
long_df = df.melt(
    id_vars=["customer_id", "customer_name"],
    value_vars=["jan_revenue", "feb_revenue", "mar_revenue"],
    var_name="month",
    value_name="revenue",
)
```

| Operation | Direction | Use When |
|---|---|---|
| Pivot | Long → Wide | Need columns per category for comparison |
| Melt | Wide → Long | Need tidy format for visualization/groupby |
| Stack | Columns → MultiIndex rows | Multi-level reshaping |
| Unstack | MultiIndex rows → Columns | Reverse of stack |

### Join / Merge Operations

```python
# Always specify how and validate
merged = pd.merge(
    orders,
    customers,
    on="customer_id",
    how="left",            # Always explicit — never rely on default
    validate="many_to_one", # Catch unexpected cardinality
    indicator=True,         # Add _merge column for diagnostics
)

# Check for unmatched rows
unmatched = merged[merged["_merge"] == "left_only"]
if len(unmatched) > 0:
    logger.warning(f"{len(unmatched)} orders have no matching customer")
```

| Join Type | Use When | Watch For |
|---|---|---|
| `inner` | Only need matched records | Silent data loss |
| `left` | Keep all left records, enrich from right | NULLs in right columns |
| `outer` | Need complete picture of both sides | NULLs on both sides |
| `cross` | Need all combinations (rare) | Row explosion |

**Rules:**
- Always specify `how` explicitly — default inner join silently drops rows
- Always use `validate` to catch cardinality issues
- Always use `indicator=True` during development to audit join quality
- Always log unmatched row counts

### Aggregation Patterns

```python
# Named aggregation — always use this over dict-based agg
customer_summary = orders.groupby("customer_id").agg(
    total_orders=("order_id", "nunique"),
    total_revenue=("amount", "sum"),
    avg_order_value=("amount", "mean"),
    first_order=("ordered_at", "min"),
    last_order=("ordered_at", "max"),
).reset_index()
```

**Rules:**
- Always use named aggregation for readability
- Always call `.reset_index()` after groupby to avoid MultiIndex
- Never aggregate on columns with NULLs without handling them first
- Always verify row count after aggregation matches expected granularity

### Window Functions (Pandas)

```python
# Rolling calculations
df["revenue_7d_avg"] = (
    df.sort_values("date")
    .groupby("customer_id")["revenue"]
    .transform(lambda x: x.rolling(7, min_periods=1).mean())
)

# Rank within groups
df["order_rank"] = (
    df.groupby("customer_id")["ordered_at"]
    .rank(method="dense", ascending=True)
)

# Lag / Lead
df["prev_order_date"] = (
    df.sort_values("ordered_at")
    .groupby("customer_id")["ordered_at"]
    .shift(1)
)
df["days_between_orders"] = (df["ordered_at"] - df["prev_order_date"]).dt.days
```

### Feature Engineering

| Feature Type | Method | Example |
|---|---|---|
| Date parts | `.dt.year`, `.dt.month`, `.dt.dayofweek` | `order_month`, `is_weekend` |
| Time since | Subtraction from reference date | `days_since_last_order` |
| Binning | `pd.cut()` or `pd.qcut()` | `revenue_bucket` |
| Ratios | Column division | `return_rate = returns / orders` |
| Flags | Boolean condition | `is_high_value = revenue > 1000` |
| Encoding | `pd.get_dummies()` or map | `region_encoded` |
| Rolling | `.rolling().mean()` | `avg_7d_revenue` |

```python
def engineer_customer_features(df: pd.DataFrame) -> pd.DataFrame:
    """Add derived features for customer analysis."""
    df = df.copy()
    today = pd.Timestamp.now()

    # Time-based
    df["account_age_days"] = (today - df["created_at"]).dt.days
    df["days_since_last_order"] = (today - df["last_order_at"]).dt.days

    # Behavioral
    df["avg_order_value"] = df["total_revenue"] / df["total_orders"]
    df["is_repeat_customer"] = df["total_orders"] > 1
    df["is_high_value"] = df["total_revenue"] > df["total_revenue"].quantile(0.9)

    # Segmentation
    df["recency_bucket"] = pd.cut(
        df["days_since_last_order"],
        bins=[0, 30, 90, 180, 365, float("inf")],
        labels=["Active", "Warm", "Cooling", "At-risk", "Churned"],
    )

    return df
```

---

## Anti-Patterns

- Never join without specifying `how` — default inner join silently drops data
- Never skip `validate` on merges — unexpected cardinality causes row explosion
- Never aggregate on columns with NULLs without prior null handling
- Never use unnamed aggregation (`agg({"col": "sum"})`) — use named agg
- Never forget `.reset_index()` after groupby — MultiIndex is hard to work with
- Never create features without documenting what they represent
- Never bin continuous variables without domain context for boundaries

---

## Cross-References

- **data-analytics/data-cleaning** → data must be clean before transformation
- **data-analytics/data-exploration** → EDA insights drive transformation choices
- **data-analytics/visualization** → transformed data feeds into charts
- **data-analytics/metrics-and-kpis** → aggregated metrics defined here

---

## Ready-to-Use Prompt

```
Task: Transform [dataset] for [analysis/reporting purpose]
Skill: data-analytics/data-transformation, data-analytics/data-cleaning

REQUIREMENTS:
- Input: [describe cleaned dataset]
- Output: [describe target shape and features]
- Operations: [joins / aggregation / pivoting / feature engineering]

CONSTRAINTS:
- Explicit join type with validate and indicator
- Named aggregation with reset_index
- Document all engineered features
- Log unmatched rows from joins
- Verify row counts after each operation

DONE WHEN:
- Data reshaped to target structure
- All joins validated for cardinality
- Features engineered and documented
- Row counts verified at each step
- Code review skill applied before commit
```

