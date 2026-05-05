---
name: dashboarding
description: Dashboard layout principles and conventions for Streamlit, Metabase, and Grafana.
---
# dashboarding.md

## Purpose

Define conventions for building data dashboards — covering layout principles,
KPI card design, filter patterns, Streamlit/Metabase/Grafana conventions,
refresh strategies, and modular dashboard architecture.

---

## Conventions

### Dashboard Architecture

```
/dashboard
  app.py                   # Main entry point
  /pages
    overview.py            # Executive summary page
    revenue.py             # Revenue deep-dive
    operations.py          # Operational metrics
  /components
    kpi_card.py            # Reusable KPI card component
    chart_builder.py       # Chart factory with standard formatting
    filters.py             # Shared filter components
  /data
    queries.py             # All data queries (separation of concerns)
    cache.py               # Caching layer
  /config
    dashboard_config.yaml  # Layout and metric definitions
```

### Layout Principles

1. **KPI Cards First** — Top row: 3–5 headline metrics with trend indicators
2. **Charts Second** — Middle: 2–3 key charts answering primary questions
3. **Detail Tables Last** — Bottom: drill-down data tables for exploration
4. **Filters Top-Left** — Sidebar or top bar: global date range and category filters

```
┌─────────────────────────────────────────────────┐
│  [Filter: Date Range]  [Filter: Category]        │
├──────────┬──────────┬──────────┬────────────────┤
│ KPI: Rev │ KPI: Ord │ KPI: AOV │ KPI: Customers │
│ €125K ↑  │ 1,234 ↑  │ €101 ↓  │ 892 →          │
├──────────┴──────────┼──────────┴────────────────┤
│ Revenue Trend (Line) │ Revenue by Channel (Bar)  │
│                      │                           │
├──────────────────────┴───────────────────────────┤
│ Orders Table (sortable, filterable)               │
└──────────────────────────────────────────────────┘
```

### KPI Card Component

```python
# components/kpi_card.py
import streamlit as st

def kpi_card(label: str, value: str, delta: str = None, delta_color: str = "normal"):
    """Render a KPI metric card."""
    st.metric(
        label=label,
        value=value,
        delta=delta,
        delta_color=delta_color,  # "normal", "inverse", "off"
    )
```

**Rules:**
- Always show comparison period (vs last week/month/year)
- Always format numbers for readability (€125K not €125000)
- Use ↑↓→ or delta values to indicate trend direction
- Limit to 5 KPI cards maximum — more dilutes focus

### Streamlit Conventions

```python
# app.py — Main dashboard
import streamlit as st

st.set_page_config(
    page_title="Business Dashboard",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded",
)

# Cache data queries — never re-fetch on every interaction
@st.cache_data(ttl=300)  # 5-minute cache
def load_data(date_range):
    return queries.get_dashboard_data(date_range)

# Sidebar filters
with st.sidebar:
    date_range = st.date_input("Date Range", value=(start, end))
    category = st.selectbox("Category", options=categories)

# KPI row
cols = st.columns(4)
with cols[0]: kpi_card("Revenue", "€125K", "+12%")
with cols[1]: kpi_card("Orders", "1,234", "+5%")
with cols[2]: kpi_card("AOV", "€101", "-3%", delta_color="inverse")
with cols[3]: kpi_card("Customers", "892", "0%")

# Charts
col1, col2 = st.columns(2)
with col1: st.plotly_chart(revenue_trend_chart, use_container_width=True)
with col2: st.plotly_chart(channel_bar_chart, use_container_width=True)

# Detail table
st.dataframe(orders_table, use_container_width=True)
```

### Data Layer Separation

Never put queries inside dashboard components:

```python
# data/queries.py — All SQL/API queries live here
import pandas as pd

def get_revenue_summary(start_date, end_date) -> pd.DataFrame:
    query = """
        SELECT date, SUM(amount) as revenue, COUNT(*) as orders
        FROM orders
        WHERE ordered_at BETWEEN :start AND :end
        GROUP BY date ORDER BY date
    """
    return db.execute(query, {"start": start_date, "end": end_date})

def get_kpi_metrics(start_date, end_date) -> dict:
    current = get_revenue_summary(start_date, end_date)
    previous = get_revenue_summary(start_date - delta, end_date - delta)
    return {
        "revenue": current["revenue"].sum(),
        "revenue_delta": calculate_delta(current, previous, "revenue"),
    }
```

### Refresh and Caching Strategy

| Data Type | Cache TTL | Refresh Trigger |
|---|---|---|
| KPI metrics | 5 minutes | Auto-refresh on interval |
| Charts | 5 minutes | Filter change or interval |
| Detail tables | 1 minute | On-demand (user interaction) |
| Static reference data | 1 hour | Application restart |

### Dashboard Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Page files | `{domain}.py` | `revenue.py`, `operations.py` |
| Component files | `{component_type}.py` | `kpi_card.py`, `filters.py` |
| Query functions | `get_{metric}_{detail}` | `get_revenue_summary` |
| Config keys | `snake_case` | `default_date_range` |

---

## Anti-Patterns

- Never put data queries inside dashboard components — use separate data layer
- Never skip caching — every interaction re-querying the database is wasteful
- Never show more than 5 KPI cards — focus on what matters
- Never build dashboards without filters — static dashboards are useless
- Never use auto-refresh intervals under 1 minute — it strains the database
- Never display raw timestamps — always format for the audience
- Never show decimal precision beyond what's meaningful (€125,432.17 → €125K)
- Never build without mobile responsiveness in mind

---

## Cross-References

- **data-analytics/visualization** → chart conventions applied inside dashboards
- **data-analytics/metrics-and-kpis** → what metrics to display
- **data-analytics/reporting** → dashboards complement static reports
- **data-engineering/warehouse** → mart models feed dashboard queries

---

## Ready-to-Use Prompt

```
Task: Build dashboard for [domain/audience]
Skill: data-analytics/dashboarding, data-analytics/visualization, data-analytics/metrics-and-kpis

REQUIREMENTS:
- Audience: [executive / analyst / operations]
- KPIs: [list 3–5 headline metrics]
- Charts: [list key visualizations]
- Filters: [date range, category, etc.]
- Tool: [Streamlit / Metabase / Grafana]

CONSTRAINTS:
- KPI cards top row with trend indicators
- Charts in middle section
- Detail tables at bottom
- Separate data layer from UI components
- Cache all queries with appropriate TTL
- Format numbers for readability
- Responsive layout

DONE WHEN:
- All KPI cards showing with deltas
- Charts rendered with standard formatting
- Filters functional and connected to data
- Caching layer implemented
- Data queries separated from UI
- Code review skill applied before commit
```

