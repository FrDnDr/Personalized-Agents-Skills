---
name: spreadsheet
description: Excel and Google Sheets conventions including formulas, pivot tables, and data entry rules.
---
# spreadsheet.md

## Purpose

Define conventions for working with spreadsheets (Excel/Google Sheets) — covering
formula best practices, pivot table conventions, data entry rules, formatting
standards, and patterns for reliable, maintainable spreadsheet work.

---

## Conventions

### Spreadsheet Structure

Every spreadsheet must follow a clear layout:

```
┌──────────────────────────────────────────────────┐
│ Sheet: README           → Purpose, data sources  │
├──────────────────────────────────────────────────┤
│ Sheet: Raw Data         → Untouched imported data │
│ Sheet: Cleaned Data     → Processed version       │
│ Sheet: Analysis         → Formulas, pivot tables  │
│ Sheet: Dashboard/Charts → Visualizations          │
│ Sheet: Config           → Parameters, lookups     │
└──────────────────────────────────────────────────┘
```

**Rules:**
- Never mix raw data and formulas on the same sheet
- Always include a README sheet explaining the workbook
- Always protect the Raw Data sheet from edits
- Always name sheets descriptively — never "Sheet1", "Sheet2"

### Data Entry Conventions

| Rule | Good | Bad |
|---|---|---|
| One value per cell | `42.50` | `42.50 EUR` |
| Consistent types per column | All dates as dates | Mix of "Jan 1" and "2026-01-01" |
| No merged cells in data ranges | Flat row per record | Merged header spanning columns |
| Headers in row 1 | `order_id`, `amount` | Data starting at row 1 |
| No blank rows in data | Contiguous range | Random empty rows |
| Use data validation | Dropdown for status | Free-text status values |

### Formula Best Practices

```
# Named ranges — always use instead of cell references
=SUMIFS(Revenue, Region, "DACH", Period, "Q1")

# Structured table references (Excel Tables)
=SUM(Orders[Amount])

# Avoid hard-coded values — reference Config sheet
=IF(A2 > Config!$B$2, "High", "Low")
```

| Practice | Good | Bad |
|---|---|---|
| Named ranges | `=SUM(Revenue)` | `=SUM(B2:B9999)` |
| Table references | `=VLOOKUP(A2, Orders, 3, 0)` | `=VLOOKUP(A2, $B$2:$F$999, 3, 0)` |
| Config references | `=IF(A2>Config!B2, ...)` | `=IF(A2>100, ...)` |
| Error handling | `=IFERROR(A2/B2, 0)` | `=A2/B2` |

### Recommended Formulas

| Need | Formula | Example |
|---|---|---|
| Conditional sum | `SUMIFS` | `=SUMIFS(Amount, Status, "Paid")` |
| Lookup | `XLOOKUP` (or `INDEX/MATCH`) | `=XLOOKUP(A2, IDs, Names)` |
| Conditional count | `COUNTIFS` | `=COUNTIFS(Region, "EU")` |
| Date extraction | `YEAR`, `MONTH`, `WEEKDAY` | `=YEAR(A2)` |
| Text cleaning | `TRIM`, `LOWER`, `SUBSTITUTE` | `=TRIM(LOWER(A2))` |
| Error guard | `IFERROR` | `=IFERROR(A2/B2, 0)` |
| Unique values | `UNIQUE` | `=UNIQUE(A2:A100)` |

### Pivot Table Conventions

- **Rows:** Categories to compare (Region, Product, Channel)
- **Columns:** Time periods (Month, Quarter) — sparingly
- **Values:** Always SUM, COUNT, or AVERAGE — never raw values
- **Filters:** Global filters at the top for slicing

**Rules:**
- Always label value fields clearly: "Sum of Revenue" → "Revenue (€)"
- Always format numbers in pivot tables — no raw decimals
- Always sort by value (descending) — not alphabetically
- Limit pivot tables to 3 dimensions — more becomes unreadable

### Formatting Standards

| Element | Convention |
|---|---|
| Currency | `€ #,##0.00` or `€ #,##0` |
| Percentages | `0.0%` (one decimal) |
| Dates | `YYYY-MM-DD` or regional format |
| Headers | Bold, background color, frozen row |
| Negative numbers | Red font or parentheses |
| Conditional formatting | Traffic light (green/yellow/red) for KPIs |

### Google Sheets Specific

```
# QUERY function — SQL-like syntax for filtering
=QUERY(Data!A:F, "SELECT A, SUM(D) WHERE C='Paid' GROUP BY A ORDER BY SUM(D) DESC")

# IMPORTRANGE — Pull data from other spreadsheets
=IMPORTRANGE("spreadsheet_url", "Sheet1!A:F")

# ARRAYFORMULA — Apply formula to entire column
=ARRAYFORMULA(IF(A2:A<>"", A2:A*B2:B, ""))
```

---

## Anti-Patterns

- Never use merged cells in data ranges — they break formulas, sorting, and filters
- Never reference raw cell addresses (`B2:B999`) — use named ranges or table references
- Never hard-code values in formulas — reference a Config sheet
- Never mix data types in a column — one type per column strictly
- Never leave formula errors visible (`#REF!`, `#N/A`) — wrap with `IFERROR`
- Never skip the README sheet — undocumented spreadsheets are unmaintainable
- Never build critical business processes on spreadsheets with >10K rows — migrate to a database
- Never use "Sheet1" as a sheet name — always descriptive names

---

## Cross-References

- **data-analytics/data-retrieval** → when to move from spreadsheet to database queries
- **data-analytics/visualization** → when charts need more than Excel can offer
- **data-analytics/data-cleaning** → complex cleaning is better done in pandas

---

## Ready-to-Use Prompt

```
Task: Build spreadsheet for [purpose]
Skill: data-analytics/spreadsheet

REQUIREMENTS:
- Purpose: [tracking / reporting / calculation / data entry]
- Data sources: [manual entry / import / linked sheets]
- Outputs: [dashboard / report / export for analysis]

CONSTRAINTS:
- Separate sheets for raw data, analysis, and charts
- Include README sheet
- Use named ranges or table references (no raw cell refs)
- All parameters on Config sheet (no hardcoded values)
- Data validation on entry columns
- IFERROR on all division formulas
- Formatting standards applied

DONE WHEN:
- All sheets created and named descriptively
- Formulas use named ranges or table references
- Pivot tables properly formatted and sorted
- Conditional formatting applied to KPIs
- README sheet documents purpose and sources
- Code review skill applied before commit
```

