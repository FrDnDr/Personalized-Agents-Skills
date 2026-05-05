---
name: notebook-conventions
description: Jupyter notebook structure, cell ordering, and markdown documentation rules for portfolio-ready notebooks.
---
# notebook-conventions.md

## Purpose

Define conventions for Jupyter notebook structure — covering cell ordering,
markdown documentation, code organization, reproducibility patterns,
and conventions that make notebooks collaborative and maintainable.

---

## Conventions

### Notebook Structure

Every notebook must follow this cell ordering:

```
Cell 1:  [Markdown]  Title, author, date, purpose
Cell 2:  [Code]      Imports and configuration
Cell 3:  [Code]      Data loading
Cell 4:  [Markdown]  Data Overview heading
Cell 5:  [Code]      Shape, dtypes, head() exploration
Cell 6:  [Markdown]  Section headings between logical blocks
Cell 7+: [Code]      Analysis, organized by section
Cell N:  [Markdown]  Conclusions and next steps
```

### Cell 1: Header Template

```markdown
# [Analysis Title]

**Author:** [Name]
**Date:** [YYYY-MM-DD]
**Status:** [Draft / Final / Archived]

## Purpose
[1-2 sentences describing what this notebook answers]

## Data Sources
- Source 1: [description, location, date range]
- Source 2: [description]

## Key Findings (updated after analysis)
- Finding 1
- Finding 2
```

### Cell 2: Imports and Config

```python
# --- Imports ---
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
import warnings

# --- Configuration ---
warnings.filterwarnings("ignore")
pd.set_option("display.max_columns", 50)
pd.set_option("display.max_rows", 100)
pd.set_option("display.float_format", "{:.2f}".format)

sns.set_theme(style="whitegrid", font_scale=1.1)
plt.rcParams["figure.figsize"] = (10, 6)
plt.rcParams["figure.dpi"] = 150

# --- Constants ---
DATA_DIR = Path("../data")
OUTPUT_DIR = Path("../output")
RANDOM_SEED = 42
```

**Rules:**
- All imports in one cell at the top — never scattered throughout
- All configuration in the same cell as imports
- All constants defined here — never magic numbers in analysis cells
- Always set a `RANDOM_SEED` for reproducibility

### Section Organization

Use markdown headers to create a navigable table of contents:

```markdown
## 1. Data Loading
## 2. Data Cleaning
## 3. Exploratory Data Analysis
### 3.1 Distribution Analysis
### 3.2 Correlation Analysis
## 4. Feature Engineering
## 5. Modeling / Analysis
## 6. Results
## 7. Conclusions & Next Steps
```

**Rules:**
- Every code block must be preceded by a markdown cell explaining what it does
- Section numbers for navigation
- Maximum 10 lines per code cell — split longer logic into functions
- Never have 5+ code cells in a row without markdown explanation

### Code Cell Best Practices

```python
# Good: Self-contained, well-documented, output is clear
def calculate_revenue_by_region(df: pd.DataFrame) -> pd.DataFrame:
    """Calculate total revenue by region with period comparison."""
    return (
        df.groupby("region")
        .agg(revenue=("amount", "sum"), orders=("order_id", "nunique"))
        .sort_values("revenue", ascending=False)
        .reset_index()
    )

revenue_by_region = calculate_revenue_by_region(orders_df)
revenue_by_region.head(10)
```

**Rules:**
- Extract reusable logic into functions — don't repeat code across cells
- End cells with a display statement (`head()`, `shape`, variable name)
- Never leave cells that produce no output — every cell should show something
- Never use `print()` for DataFrames — use `display()` or just the variable name

### Output Management

- Clear all outputs before committing to version control
- Use `%store` magic for sharing variables between notebooks
- Save intermediate results to files for long-running computations:

```python
# Save expensive computation result
if not Path("../data/processed/enriched_orders.parquet").exists():
    enriched = expensive_computation(raw_df)
    enriched.to_parquet("../data/processed/enriched_orders.parquet")
else:
    enriched = pd.read_parquet("../data/processed/enriched_orders.parquet")
```

### Conclusions Cell Template

```markdown
## 7. Conclusions & Next Steps

### Key Findings
1. [Quantified finding with context]
2. [Quantified finding with context]

### Limitations
- [Caveat about data quality, scope, or methodology]

### Next Steps
- [ ] [Specific action item]
- [ ] [Specific action item]

### Questions for Stakeholders
- [Open question needing input]
```

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Notebook file | `{number}_{verb}_{topic}.ipynb` | `01_explore_orders.ipynb` |
| DataFrames | Descriptive, suffixed with `_df` | `orders_df`, `clean_orders_df` |
| Figures | `fig`, `ax` or descriptive | `revenue_fig`, `dist_ax` |
| Constants | `UPPER_SNAKE_CASE` | `DATA_DIR`, `RANDOM_SEED` |

### Notebook File Organization

```
/notebooks
  01_data_loading.ipynb
  02_data_cleaning.ipynb
  03_exploratory_analysis.ipynb
  04_feature_engineering.ipynb
  05_modeling.ipynb
  06_results_and_reporting.ipynb
/data
  /raw
  /processed
/output
  /charts
  /reports
```

---

## Anti-Patterns

- Never scatter imports throughout the notebook — all imports in cell 2
- Never have code cells without preceding markdown explanation
- Never use magic numbers — define constants in the config cell
- Never leave cells that produce no visible output
- Never commit notebooks with outputs to version control (use `nbstripout`)
- Never write notebooks longer than ~50 cells — split into multiple notebooks
- Never copy-paste code between cells — extract into functions
- Never skip the conclusions cell — notebooks without conclusions are incomplete

---

## Cross-References

- **data-analytics/data-exploration** → EDA patterns used inside notebooks
- **data-analytics/visualization** → chart conventions applied in notebook charts
- **data-analytics/data-cleaning** → cleaning workflows executed in notebooks
- **core/testing** → extract notebook logic into testable modules

---

## Ready-to-Use Prompt

```
Task: Create analysis notebook for [topic]
Skill: data-analytics/notebook-conventions, data-analytics/data-exploration

REQUIREMENTS:
- Purpose: [what question does this notebook answer]
- Data: [describe sources]
- Sections: [list analysis sections needed]

CONSTRAINTS:
- Follow standard cell ordering (header → imports → load → analyze → conclude)
- All imports in one cell at top
- Markdown before every code block
- Max 10 lines per code cell
- Extract reusable logic into functions
- Every cell produces visible output
- Conclusions with key findings and next steps
- Numbered notebook filename

DONE WHEN:
- Notebook follows standard structure
- All sections have markdown headers
- Key findings documented in conclusions
- Outputs clear and readable
- Code review skill applied before commit
```

