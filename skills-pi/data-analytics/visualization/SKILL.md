---
name: visualization
description: Chart selection guide and conventions for Matplotlib, Seaborn, and Plotly including color rules.
---
# visualization.md

## Purpose

Define conventions for data visualization — covering chart selection, Matplotlib/Seaborn/Plotly
conventions, color palette rules, annotation standards, and modular chart-building patterns
for clear, accurate, and publication-quality data communication.

---

## Conventions

### Chart Selection Guide

| Question | Chart Type | Library |
|---|---|---|
| How is one variable distributed? | Histogram, KDE, Box plot | Seaborn |
| How do two variables relate? | Scatter plot, Heatmap | Seaborn / Plotly |
| How do categories compare? | Bar chart (horizontal preferred) | Matplotlib / Plotly |
| How does a value change over time? | Line chart | Matplotlib / Plotly |
| What is the part-to-whole? | Stacked bar, Treemap (avoid pie) | Plotly |
| How are values distributed per group? | Violin, Box, Strip | Seaborn |
| What are top N items? | Horizontal bar chart | Matplotlib |
| How do multiple series trend? | Multi-line chart | Plotly |
| Geographic patterns? | Choropleth, Scatter map | Plotly |

### Default Style Configuration

```python
# Always set at the top of any visualization script/notebook
import matplotlib.pyplot as plt
import seaborn as sns

# Global defaults
sns.set_theme(style="whitegrid", font_scale=1.1)
plt.rcParams.update({
    "figure.figsize": (10, 6),
    "figure.dpi": 150,
    "axes.titlesize": 14,
    "axes.titleweight": "bold",
    "axes.labelsize": 12,
    "savefig.bbox": "tight",
    "savefig.dpi": 300,
})
```

### Color Palette Rules

| Palette Type | Use Case | Example |
|---|---|---|
| Sequential | Ordered values (low → high) | `Blues`, `Viridis`, `YlOrRd` |
| Diverging | Deviation from center | `RdBu`, `coolwarm` |
| Categorical | Distinct groups | `Set2`, `tab10`, custom |
| Highlight | Emphasize one category | Gray + one accent color |

**Rules:**
- Never use rainbow/jet colormap — poor for colorblind readers
- Always use colorblind-safe palettes: `Set2`, `ColorBrewer`, `Viridis`
- Limit categorical colors to 8 max — group remaining as "Other"
- Use consistent colors for the same entity across all charts
- Use gray for context/background data, accent for the story

```python
# Standard color palettes
PALETTE_CATEGORICAL = sns.color_palette("Set2", 8)
PALETTE_SEQUENTIAL = "YlOrRd"
PALETTE_DIVERGING = "RdBu_r"
ACCENT_COLOR = "#E63946"
MUTED_COLOR = "#A8DADC"
```

### Chart Anatomy (Every Chart Must Have)

1. **Title** — What the chart shows (descriptive, not generic)
2. **Axis labels** — With units where applicable
3. **Legend** — Only if multiple series (positioned to not overlap data)
4. **Source annotation** — Data source and date range
5. **Clean whitespace** — Remove chartjunk (unnecessary gridlines, borders)

```python
def format_chart(ax, title: str, xlabel: str, ylabel: str, source: str = None):
    """Apply standard formatting to any Matplotlib axis."""
    ax.set_title(title, pad=15)
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    ax.spines[["top", "right"]].set_visible(False)
    if source:
        ax.text(0.99, -0.12, f"Source: {source}", transform=ax.transAxes,
                fontsize=8, ha="right", style="italic", color="gray")
```

### Matplotlib Patterns

```python
# Bar chart — always horizontal for readability with long labels
fig, ax = plt.subplots(figsize=(10, 6))
df_sorted = df.sort_values("revenue")
ax.barh(df_sorted["category"], df_sorted["revenue"], color=PALETTE_CATEGORICAL[0])
format_chart(ax, "Revenue by Category", "Revenue (€)", "")
plt.tight_layout()
```

### Seaborn Patterns

```python
# Distribution — histogram + KDE overlay
fig, ax = plt.subplots(figsize=(10, 6))
sns.histplot(df["order_value"], kde=True, bins=30, color=PALETTE_CATEGORICAL[0], ax=ax)
format_chart(ax, "Distribution of Order Values", "Order Value (€)", "Frequency")
```

### Plotly Patterns (Interactive)

```python
import plotly.express as px

# Time series — interactive with hover
fig = px.line(
    df, x="date", y="revenue", color="channel",
    title="Revenue Trend by Channel",
    labels={"revenue": "Revenue (€)", "date": "Date"},
    template="plotly_white",
)
fig.update_layout(
    hovermode="x unified",
    legend=dict(orientation="h", yanchor="bottom", y=1.02),
)
fig.show()
```

### Annotation Best Practices

```python
# Annotate key data points — not every point
ax.annotate(
    f"Peak: €{peak_value:,.0f}",
    xy=(peak_date, peak_value),
    xytext=(15, 15), textcoords="offset points",
    arrowprops=dict(arrowstyle="->", color="gray"),
    fontsize=10, fontweight="bold",
)
```

**Rules:**
- Annotate only the most important points — not every data point
- Use offset text with arrows for clarity
- Format numbers with proper separators and units
- Keep annotation text concise — 5 words or fewer

### Number Formatting

```python
# Format large numbers for readability
from matplotlib.ticker import FuncFormatter

def currency_formatter(x, pos):
    if x >= 1e6: return f"€{x/1e6:.1f}M"
    if x >= 1e3: return f"€{x/1e3:.0f}K"
    return f"€{x:.0f}"

ax.xaxis.set_major_formatter(FuncFormatter(currency_formatter))
```

### Export Conventions

| Format | Use Case | Settings |
|---|---|---|
| PNG | Reports, presentations | `dpi=300, bbox_inches="tight"` |
| SVG | Web, scalable | `format="svg"` |
| HTML | Interactive dashboards | Plotly `.write_html()` |
| PDF | Print-ready documents | `format="pdf"` |

---

## Anti-Patterns

- Never use pie charts for more than 3 categories — use horizontal bar instead
- Never use 3D charts — they distort perception and add no information
- Never use rainbow/jet colormaps — not colorblind-safe
- Never skip axis labels or titles — unlabeled charts are uninterpretable
- Never use default Matplotlib colors without intentional styling
- Never plot time on the y-axis — time always goes on x-axis
- Never truncate y-axis to exaggerate differences without clear labeling
- Never create charts without specifying figure size and DPI

---

## Cross-References

- **data-analytics/data-exploration** → EDA visualizations for distributions and correlations
- **data-analytics/dashboarding** → charts embedded in dashboards
- **data-analytics/reporting** → publication-quality charts for reports
- **frontend/design-system** → color token alignment for web-embedded charts

---

## Ready-to-Use Prompt

```
Task: Create visualization for [data/metric]
Skill: data-analytics/visualization, data-analytics/data-exploration

REQUIREMENTS:
- Data: [describe dataset and key columns]
- Chart type: [bar / line / scatter / distribution / heatmap]
- Audience: [technical / business / executive]
- Format: [static / interactive]

CONSTRAINTS:
- Follow chart selection guide
- Colorblind-safe palettes only
- Include title, axis labels, source annotation
- Format numbers with proper separators and units
- Remove chartjunk (unnecessary gridlines, borders, 3D)
- Annotate only key data points

DONE WHEN:
- Chart accurately represents the data
- All formatting standards applied
- Exported in correct format and DPI
- Code review skill applied before commit
```

