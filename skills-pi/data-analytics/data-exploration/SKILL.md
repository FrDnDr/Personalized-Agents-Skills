---
name: data-exploration
description: EDA patterns including descriptive stats, correlation analysis, and distribution inspection.
---
# data-exploration.md

## Purpose

Define conventions for Exploratory Data Analysis (EDA) — covering descriptive statistics,
distribution analysis, correlation analysis, pattern discovery, and systematic
approaches to understanding data before modeling or reporting.

---

## Conventions

### EDA Workflow

Follow this systematic sequence for every new dataset:

```
1. Shape & Structure    → rows, columns, dtypes, memory
2. Summary Statistics   → mean, median, std, min, max, quartiles
3. Missing Data Audit   → null counts, null patterns
4. Distribution Check   → histograms, skewness, kurtosis
5. Correlation Analysis → numeric correlations, categorical associations
6. Outlier Scan         → IQR, z-score, visual inspection
7. Temporal Patterns    → trends, seasonality (if time-series)
8. Key Findings Log     → document insights before moving on
```

### Step 1: Shape & Structure

```python
def explore_structure(df: pd.DataFrame) -> dict:
    """Quick structural overview of the dataset."""
    return {
        "shape": df.shape,
        "columns": list(df.columns),
        "dtypes": df.dtypes.value_counts().to_dict(),
        "memory_mb": round(df.memory_usage(deep=True).sum() / 1e6, 2),
        "sample": df.head(5).to_dict(),
    }
```

### Step 2: Summary Statistics

```python
def summary_statistics(df: pd.DataFrame) -> pd.DataFrame:
    """Extended summary for numeric and categorical columns."""
    numeric_summary = df.describe(percentiles=[.01, .05, .25, .5, .75, .95, .99])
    categorical_summary = df.select_dtypes(include=["object", "category"]).describe()
    return numeric_summary, categorical_summary
```

- Always include extreme percentiles (1%, 99%) to spot tail behavior
- Always check `nunique()` for categorical columns — high cardinality may need bucketing
- Always compare `mean` vs `median` — large gaps indicate skew

### Step 3: Missing Data Audit

```python
def missing_data_audit(df: pd.DataFrame) -> pd.DataFrame:
    """Profile missing data by column."""
    nulls = df.isnull().sum()
    pct = (nulls / len(df) * 100).round(2)
    return pd.DataFrame({
        "null_count": nulls,
        "null_pct": pct,
        "dtype": df.dtypes,
    }).sort_values("null_pct", ascending=False).query("null_count > 0")
```

| Null % Range | Action |
|---|---|
| 0% | No action needed |
| 1–5% | Safe to impute (median/mode) |
| 5–30% | Investigate pattern — may be MNAR |
| 30–70% | Consider dropping column or advanced imputation |
| >70% | Drop column — too sparse to be useful |

### Step 4: Distribution Analysis

```python
def check_distributions(df: pd.DataFrame, numeric_cols: list[str] = None):
    """Assess distribution shape for numeric columns."""
    cols = numeric_cols or df.select_dtypes(include="number").columns
    results = {}
    for col in cols:
        results[col] = {
            "skewness": round(df[col].skew(), 3),
            "kurtosis": round(df[col].kurtosis(), 3),
            "is_normal": abs(df[col].skew()) < 0.5,
        }
    return pd.DataFrame(results).T
```

| Skewness | Interpretation | Treatment |
|---|---|---|
| -0.5 to 0.5 | Approximately normal | Use as-is |
| 0.5 to 1.0 | Moderately skewed | Log transform may help |
| >1.0 | Highly skewed | Log/Box-Cox transform needed |

### Step 5: Correlation Analysis

```python
def correlation_analysis(df: pd.DataFrame, threshold: float = 0.7) -> pd.DataFrame:
    """Find highly correlated numeric pairs."""
    corr = df.select_dtypes(include="number").corr()
    # Extract upper triangle
    pairs = []
    for i in range(len(corr.columns)):
        for j in range(i + 1, len(corr.columns)):
            if abs(corr.iloc[i, j]) >= threshold:
                pairs.append({
                    "col_a": corr.columns[i],
                    "col_b": corr.columns[j],
                    "correlation": round(corr.iloc[i, j], 3),
                })
    return pd.DataFrame(pairs).sort_values("correlation", key=abs, ascending=False)
```

**Rules:**
- Flag pairs with |r| > 0.7 as potential multicollinearity
- Use Spearman for non-linear relationships
- Use Cramér's V for categorical-to-categorical associations

### Step 6: Outlier Scan

```python
def outlier_scan(df: pd.DataFrame, numeric_cols: list[str] = None) -> pd.DataFrame:
    """Quantify outliers per numeric column using IQR."""
    cols = numeric_cols or df.select_dtypes(include="number").columns
    results = []
    for col in cols:
        q1, q3 = df[col].quantile([0.25, 0.75])
        iqr = q3 - q1
        outlier_count = ((df[col] < q1 - 1.5 * iqr) | (df[col] > q3 + 1.5 * iqr)).sum()
        results.append({"column": col, "outlier_count": outlier_count, "outlier_pct": round(outlier_count / len(df) * 100, 2)})
    return pd.DataFrame(results).sort_values("outlier_pct", ascending=False)
```

### Key Findings Log

Always document EDA insights before moving to transformation or modeling:

```markdown
## EDA Findings — [Dataset Name]

### Shape
- Rows: X, Columns: Y

### Data Quality
- Null columns: [list with percentages]
- Duplicates found: [count]

### Distribution Insights
- Skewed columns: [list with skewness values]
- Outlier-heavy columns: [list with counts]

### Correlations
- Strong pairs: [list with r values]

### Key Observations
- [Observation 1]
- [Observation 2]

### Recommended Actions
- [Action 1: e.g., "Log-transform revenue column"]
- [Action 2: e.g., "Investigate missing shipping_date pattern"]
```

---

## Anti-Patterns

- Never skip EDA and jump straight to modeling — blind analysis produces wrong conclusions
- Never look at only summary stats — always visualize distributions
- Never ignore missing data patterns — MNAR (Missing Not At Random) biases results
- Never assume normality without checking skewness and kurtosis
- Never report correlations without checking for non-linear relationships
- Never leave EDA insights undocumented — future you will forget

---

## Cross-References

- **data-analytics/data-cleaning** → EDA reveals what cleaning is needed
- **data-analytics/visualization** → EDA charts for distribution and correlation
- **data-analytics/data-transformation** → EDA insights drive transformation decisions
- **data-analytics/statistical-analysis** → formal hypothesis testing after EDA

---

## Ready-to-Use Prompt

```
Task: Perform EDA on [dataset name]
Skill: data-analytics/data-exploration, data-analytics/visualization

REQUIREMENTS:
- Dataset: [describe source and purpose]
- Focus areas: [distributions / correlations / outliers / time patterns]

CONSTRAINTS:
- Follow 8-step EDA workflow in order
- Document all findings in Key Findings Log
- Include extreme percentiles in summary stats
- Flag high-null columns with action recommendations
- Identify strong correlations (|r| > 0.7)
- Quantify outliers per column

DONE WHEN:
- All 8 EDA steps completed
- Key Findings Log documented with observations
- Recommended actions listed
- Visualizations created for key distributions
- Code review skill applied before commit
```

