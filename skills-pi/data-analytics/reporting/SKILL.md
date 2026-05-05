---
name: reporting
description: Report structure conventions and narrative writing around data including executive summary format.
---
# reporting.md

## Purpose

Define conventions for data-driven reports — covering report structure, narrative
writing around data, executive summary formatting, insight communication,
and reproducible report generation patterns.

---

## Conventions

### Report Structure

Every data report follows a five-section framework:

```
1. Executive Summary     → Key findings in 3–5 bullet points
2. Context & Methodology → What was analyzed, how, and why
3. Key Findings          → Charts + narrative, organized by theme
4. Recommendations       → Actionable next steps tied to findings
5. Appendix              → Detailed tables, methodology notes, caveats
```

### Executive Summary Rules

- **Maximum 5 bullet points** — each one actionable or insight-bearing
- **Lead with the most important finding** — not the methodology
- **Quantify everything** — "Revenue grew 12%" not "Revenue grew"
- **Include time frame** — "in Q1 2026" not "recently"
- **End with a recommendation** — what should the reader do next

```markdown
## Executive Summary

- **Revenue reached €1.2M in Q1 2026**, a 12% increase YoY driven by the
  DACH region (+23%).
- **Customer acquisition cost (CAC) rose 18%** to €47, primarily from
  increased ad spend on Meta channels.
- **Repeat purchase rate declined** from 34% to 28%, indicating retention
  issues in the 60–90 day window.
- **Top recommendation:** Shift 15% of Meta budget to email retention
  campaigns targeting the 60-day cohort.
```

### Narrative Writing Conventions

| Principle | Good | Bad |
|---|---|---|
| Lead with insight | "Revenue grew 12%, driven by DACH" | "This chart shows revenue" |
| Quantify always | "CAC rose 18% to €47" | "CAC increased significantly" |
| Be specific | "Q1 2026 vs Q1 2025" | "compared to before" |
| State implications | "This suggests retention is declining" | "The number went down" |
| Actionable | "Recommend shifting budget to email" | "Something should be done" |

### Finding Template

Each finding follows: **Observation → Context → Implication → Action**

```markdown
### Finding 1: DACH Region Driving Growth

**Observation:** DACH region revenue grew 23% YoY to €420K, now representing
35% of total revenue (up from 29%).

**Context:** Growth correlates with the March product launch and localized
marketing campaign that ran Feb–Apr.

**Implication:** DACH is becoming the primary growth engine. Over-reliance
on one region creates concentration risk.

**Action:** Continue DACH investment while diversifying into Benelux (currently
underperforming at -5% YoY).

![DACH Revenue Trend](charts/dach_revenue_trend.png)
```

### Chart-Narrative Pairing

Every chart in a report must be accompanied by a narrative paragraph:

1. **State what the chart shows** — one sentence
2. **Highlight the key takeaway** — what should the reader notice
3. **Provide context** — why this matters or what caused it
4. **Suggest action** — if applicable

### Report Formatting

| Element | Convention |
|---|---|
| Date format | "Q1 2026" or "January 2026" — never "01/2026" |
| Currency | "€125K" for summaries, "€125,432" for detail |
| Percentages | One decimal: "12.3%" — never "12.3456%" |
| Comparisons | "12% YoY" or "+€15K vs prior month" |
| Tables | Max 7 columns — if more, move to appendix |
| Charts per page | Max 2 — each with narrative |
| Report length | Exec summary: 1 page, Full report: 5–10 pages |

### Reproducible Report Generation

```python
# report_generator.py
from datetime import date
from pathlib import Path

class ReportGenerator:
    def __init__(self, title: str, period: str, author: str):
        self.title = title
        self.period = period
        self.author = author
        self.sections = []

    def add_section(self, heading: str, content: str, charts: list[Path] = None):
        self.sections.append({
            "heading": heading,
            "content": content,
            "charts": charts or [],
        })

    def render_markdown(self) -> str:
        lines = [
            f"# {self.title}",
            f"**Period:** {self.period}  ",
            f"**Author:** {self.author}  ",
            f"**Generated:** {date.today().isoformat()}",
            "",
        ]
        for section in self.sections:
            lines.append(f"## {section['heading']}")
            lines.append(section["content"])
            for chart in section["charts"]:
                lines.append(f"![{chart.stem}]({chart})")
            lines.append("")
        return "\n".join(lines)
```

---

## Anti-Patterns

- Never start a report with methodology — lead with the executive summary
- Never present charts without narrative — charts don't speak for themselves
- Never use vague language ("significant", "notable") — quantify everything
- Never include more than 2 charts per page — it overwhelms the reader
- Never skip the recommendations section — reports without actions are wasted effort
- Never present raw data without context — explain what it means and why it matters
- Never assume the reader has domain knowledge — provide brief context

---

## Cross-References

- **data-analytics/visualization** → chart conventions for report figures
- **data-analytics/metrics-and-kpis** → metric definitions for reporting
- **data-analytics/dashboarding** → live dashboards complement periodic reports

---

## Ready-to-Use Prompt

```
Task: Create [report type] report for [audience]
Skill: data-analytics/reporting, data-analytics/visualization, data-analytics/metrics-and-kpis

REQUIREMENTS:
- Period: [date range]
- Audience: [executive / manager / analyst]
- Focus: [revenue / operations / customer / marketing]
- Findings: [expected number of key findings]

CONSTRAINTS:
- Five-section structure (summary → context → findings → recommendations → appendix)
- Executive summary: max 5 bullets, lead with biggest insight
- Each finding: observation → context → implication → action
- Every chart paired with narrative paragraph
- Quantify all claims — no vague language
- Format numbers per convention table

DONE WHEN:
- Executive summary captures key insights
- All findings follow the four-part template
- Charts have narrative accompaniment
- Recommendations are specific and actionable
- Report length within 5–10 pages
- Code review skill applied before commit
```

