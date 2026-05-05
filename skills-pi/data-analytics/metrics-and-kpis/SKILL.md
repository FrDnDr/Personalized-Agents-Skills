---
name: metrics-and-kpis
description: Defining metrics, KPI frameworks, and rules for avoiding vanity metrics.
---
# metrics-and-kpis.md

## Purpose

Define conventions for designing, defining, and managing business metrics and KPIs —
covering metric taxonomy, KPI frameworks, metric ownership, avoiding vanity metrics,
and standardized metric definition templates.

---

## Conventions

### Metric Taxonomy

| Tier | Type | Purpose | Example |
|---|---|---|---|
| Tier 1 | North Star | Company-level success indicator | Monthly Active Revenue |
| Tier 2 | KPIs | Department-level performance | CAC, LTV, Churn Rate |
| Tier 3 | Operational | Process-level health | Order processing time |
| Tier 4 | Diagnostic | Root cause investigation | Cart abandonment rate by step |

### Metric Definition Template

Every metric must be documented using this template:

```yaml
metric:
  name: "Customer Acquisition Cost (CAC)"
  tier: 2                           # Tier 1–4
  owner: "Marketing"                # Department responsible
  definition: "Total marketing spend divided by number of new customers acquired"
  formula: "SUM(marketing_spend) / COUNT(DISTINCT new_customers)"
  unit: "EUR"
  direction: "lower_is_better"      # lower_is_better | higher_is_better | target_range
  frequency: "monthly"              # daily | weekly | monthly | quarterly
  data_source: "marketing_spend + orders"
  dimensions:                        # Breakdowns available
    - channel
    - region
    - campaign
  targets:
    good: "<€40"
    acceptable: "€40–€60"
    bad: ">€60"
  caveats:
    - "Excludes organic/referral customers"
    - "Spend attribution uses last-touch model"
```

### KPI Framework: Input → Output → Outcome

Structure KPIs as a causal chain:

```
INPUT METRICS          OUTPUT METRICS         OUTCOME METRICS
(actions we control)   (results of actions)   (business impact)
─────────────────      ──────────────────     ─────────────────
Ad spend               Impressions            Revenue
Email campaigns sent   Click-through rate     Customer LTV
Support tickets handled Resolution time        NPS / CSAT
Features shipped       Adoption rate          Retention
```

**Rules:**
- Always pair outcome metrics with the input metrics that drive them
- Track input metrics weekly, outcome metrics monthly
- Never set targets on outcome metrics alone — manage the inputs

### Common KPI Definitions

#### Revenue Metrics

| Metric | Formula | Direction |
|---|---|---|
| Monthly Recurring Revenue (MRR) | SUM(active_subscription_value) | ↑ Higher |
| Average Order Value (AOV) | SUM(revenue) / COUNT(orders) | ↑ Higher |
| Revenue Per Customer | SUM(revenue) / COUNT(DISTINCT customers) | ↑ Higher |
| Gross Margin | (Revenue - COGS) / Revenue × 100 | ↑ Higher |

#### Customer Metrics

| Metric | Formula | Direction |
|---|---|---|
| Customer Acquisition Cost (CAC) | Marketing Spend / New Customers | ↓ Lower |
| Customer Lifetime Value (LTV) | AOV × Purchase Frequency × Avg Lifespan | ↑ Higher |
| LTV:CAC Ratio | LTV / CAC | ↑ Higher (target >3:1) |
| Churn Rate | Lost Customers / Total Customers × 100 | ↓ Lower |
| Retention Rate | Retained Customers / Starting Customers × 100 | ↑ Higher |

#### Operational Metrics

| Metric | Formula | Direction |
|---|---|---|
| Order Fulfillment Time | AVG(shipped_at - ordered_at) | ↓ Lower |
| Return Rate | Returned Orders / Total Orders × 100 | ↓ Lower |
| Inventory Turnover | COGS / Average Inventory | ↑ Higher |

### Vanity Metric Detection

A metric is "vanity" if:

| Warning Sign | Example | Better Alternative |
|---|---|---|
| No actionable insight | Total page views | Conversion rate |
| Always goes up | Cumulative users | Monthly active users |
| No target or threshold | Social media followers | Engagement rate |
| Not tied to revenue/retention | App downloads | Day-7 retention |
| Can be gamed easily | Email list size | Email open rate |

**Rule:** If a metric doesn't change a decision, it's vanity — remove it from dashboards.

### Metric Governance

- **Single owner per metric** — one team is accountable for definition and accuracy
- **Version metric definitions** — when formulas change, document why and when
- **Review quarterly** — remove metrics no longer driving decisions
- **Centralize definitions** — one source of truth, not spreadsheet copies

### Metric SQL Patterns

```sql
-- Standard metric calculation with period comparison
WITH current_period AS (
    SELECT
        DATE_TRUNC('month', ordered_at) AS period,
        COUNT(DISTINCT customer_id) AS customers,
        COUNT(*) AS orders,
        SUM(amount) AS revenue,
        SUM(amount) / COUNT(*) AS aov
    FROM orders
    WHERE ordered_at >= DATE_TRUNC('month', CURRENT_DATE)
    GROUP BY 1
),
prior_period AS (
    SELECT
        COUNT(DISTINCT customer_id) AS customers,
        COUNT(*) AS orders,
        SUM(amount) AS revenue,
        SUM(amount) / COUNT(*) AS aov
    FROM orders
    WHERE ordered_at >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
      AND ordered_at < DATE_TRUNC('month', CURRENT_DATE)
)
SELECT
    c.*,
    ROUND((c.revenue - p.revenue) / p.revenue * 100, 1) AS revenue_delta_pct
FROM current_period c
CROSS JOIN prior_period p;
```

---

## Anti-Patterns

- Never define metrics without specifying the formula — ambiguity causes misalignment
- Never track vanity metrics on dashboards — they consume attention without value
- Never set targets on outcome metrics without tracking the input metrics that drive them
- Never let multiple teams define the same metric differently — centralize definitions
- Never display metrics without comparison period — absolute numbers lack context
- Never show more than 5 KPIs on a single dashboard — focus on what drives decisions
- Never change metric definitions without version documentation

---

## Cross-References

- **data-analytics/dashboarding** → KPIs displayed as headline metrics
- **data-analytics/reporting** → metrics contextualized in narrative reports
- **data-analytics/visualization** → metric trends visualized in charts
- **database/sql** → SQL patterns for metric calculations

---

## Ready-to-Use Prompt

```
Task: Define metrics framework for [domain/department]
Skill: data-analytics/metrics-and-kpis, data-analytics/dashboarding

REQUIREMENTS:
- Domain: [revenue / marketing / operations / product]
- Metrics needed: [list key questions to answer]
- Audience: [executive / manager / analyst]

CONSTRAINTS:
- Use metric definition template for every metric
- Classify by tier (North Star / KPI / Operational / Diagnostic)
- Map input → output → outcome chains
- Screen for vanity metrics using detection checklist
- Define targets with good/acceptable/bad thresholds
- Single owner per metric

DONE WHEN:
- All metrics defined with template
- Tier classification complete
- Input-output-outcome chains mapped
- Vanity metrics identified and removed
- SQL patterns provided for key calculations
- Code review skill applied before commit
```

