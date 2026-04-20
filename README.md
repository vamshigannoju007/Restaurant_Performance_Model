# 🍩 Restaurant Business Intelligence System

A full-stack business intelligence project simulating a multi-location restaurant brand's data infrastructure — from raw operational data through a PostgreSQL star-schema warehouse to four Power BI dashboards.

Built as a realistic end-to-end BI system using **simulated data** from four source platforms: Toast (POS), R365 (restaurant management), HubSpot (CRM), and a unified analytics reporting layer.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Database Schema](#database-schema)
- [Dashboards](#dashboards)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [Key DAX Patterns](#key-dax-patterns)
- [Data Quality Framework](#data-quality-framework)
- [Known Limitations](#known-limitations)
- [Documentation](#documentation)

---

## Overview

This project answers three business questions across four dashboards:

| Audience | Question |
|---|---|
| Store GM | How are we doing today vs last week and last year? |
| Operations | Which locations are performing well and why? |
| Executive | What are we going to make next month — and are we on track? |
| Sales Team | What is in the pipeline and how much of it became real revenue? |

The system ingests data from three simulated operational platforms, transforms it into a clean star-schema reporting layer in PostgreSQL, and connects to Power BI for analysis.

---

## Architecture

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│    toast    │  │    r365     │  │   hubspot   │
│  (POS data) │  │ (Ops/Fin)  │  │   (CRM)     │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────────────┼────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │    analytics     │
              │  (reporting layer│
              │   star schema)   │
              └────────┬─────────┘
                       │
                       ▼
                  Power BI
           ┌──────────────────────┐
           │  GM  │  Ops  │  Exec │
           │         HubSpot      │
           └──────────────────────┘
```

### Schemas

| Schema | Source System | Contains |
|---|---|---|
| `toast` | Toast POS | Sales transactions, labor shifts |
| `r365` | R365 | Inventory, finance, operations, marketing |
| `hubspot` | HubSpot CRM | Pipeline deals, deal-to-sales bridge |
| `analytics` | Reporting layer | All dimensions, reporting facts, audit tables |

---

## Database Schema

### Source Tables

<details>
<summary><strong>toast</strong></summary>

- `fact_sales_raw` — line-level sales with order_id, product_id, channel_id, gross/net sales, discounts, refunds
- `fact_labor_employee_raw` — shift-level labor with hours, pay rates, compliance flags

</details>

<details>
<summary><strong>r365</strong></summary>

- `fact_inventory_daily_raw` — daily stock levels, waste, received qty by store and product
- `fact_finance_monthly_raw` — monthly P&L by store: revenue, COGS, labor, EBITDA proxy
- `fact_ops_daily_raw` — daily CSAT, complaint count, error rate, ticket time, scorecard
- `fact_marketing_daily_raw` — campaign-level spend, impressions, clicks, leads, conversions

</details>

<details>
<summary><strong>hubspot</strong></summary>

- `fact_pipeline_raw` — deals with stage, probability, owner, expected/actual close dates, won revenue
- `bridge_pipeline_to_sales_raw` — maps HubSpot deals to Toast orders for pipeline realization analysis

</details>

### Reporting Layer (`analytics`)

**Dimensions**

| Table | Key | Description |
|---|---|---|
| `dim_date` | `date_key` | Full calendar spine — marked as Date Table in Power BI |
| `dim_store` | `store_id` | Location attributes, channel capabilities, rent |
| `dim_channel` | `channel_id` | in_store / delivery / app / catering / wholesale |
| `dim_product` | `product_id` | Item name, category, cost, margin, flags |
| `dim_employee` | `employee_id` | Staff details, role, hire date, contract type |
| `dim_labor_role` | `role_id` | Role groupings and typical pay rates |
| `dim_customer_segment` | `segment_id` | corporate / events / wholesale |
| `dim_pipeline_stage` | `stage_id` | Stage order and default probability |

**Reporting Facts**

| Table | Grain | Primary Use |
|---|---|---|
| `fact_store_day` | Store + Day | GM dashboard KPI cards, Labor %, SPLH, Prime Cost % |
| `fact_sales_order` | Order | Channel revenue, hourly sales, AOV |
| `fact_sales_line` | Order Line | Product mix, item profitability, top items |
| `fact_labor_shift` | Shift | Employee-level labor, overtime, compliance |
| `fact_inventory_day` | Product + Store + Day | Stock levels, waste cost |
| `fact_finance_month` | Store + Month | Revenue trends, EBITDA, Prime Cost (monthly grain) |
| `fact_ops_day` | Store + Day | CSAT, complaints, error rates, scorecard |
| `fact_marketing_day` | Campaign + Day | Spend, funnel metrics, attributed pipeline |
| `fact_pipeline_deal` | Deal | CRM pipeline health and deal outcomes |
| `fact_pipeline_realization` | Deal–Order Bridge | Pipeline-to-revenue matching |

---

## Dashboards

### GM Dashboard — Store Level
Daily operational monitoring for store General Managers.

- Daily net sales vs prior week and prior year (window-logic line charts)
- Hourly sales: selected day vs same day last week
- Labor % and SPLH
- Prime Cost % trend
- Top 10 selling items by quantity
- Item profitability table
- Catering orders, sales, and AOV
- Customer satisfaction gauge

### Operations Dashboard — Multi-Unit
Cross-location performance benchmarking for operations leadership.

- Store ranking matrix (sales, labor %, prime cost %, CSAT, error rate, scorecard)
- COGS by product category
- Catering performance by location
- Sales mix: retail vs catering, product category breakdown
- Customer feedback KPIs and trend
- Operational error rates by store
- Marketing funnel (split by totals / ratios / unit costs)

### Executive Dashboard
Business-wide summary for senior leadership.

- Total revenue (retail + catering + wholesale)
- Pipeline value: next 30 / 60 / 90 days
- Win rate and close rate
- Revenue by channel (bar chart)
- EBITDA by location
- Revenue growth vs prior month and prior year
- Revenue rolling 3-month trend
- Prime Cost % trend
- Marketing ROI

### HubSpot Dashboard — 2 Pages

**Page 1: Pipeline Health & Forecast**
- Pipeline value by stage and segment
- Open pipeline KPI cards
- Weighted forecast windows (30 / 60 / 90 days)
- Expected close by month
- Slipped deals and average days to close

**Page 2: Conversion & Revenue Realization**
- Won / lost / open outcomes
- Sales rep performance matrix
- Pipeline realization rate
- Realized revenue vs pipeline by segment and rep

---

## Tech Stack

- **Database** — PostgreSQL
- **Reporting** — Power BI Desktop
- **Data generation** — Python (simulated synthetic data)
- **DAX** — All KPI measures built in Power BI measure tables

---

## Project Structure

```
/
├── sql/
│   ├── schema/
│   │   ├── toast_tables.sql
│   │   ├── r365_tables.sql
│   │   ├── hubspot_tables.sql
│   │   └── analytics_tables.sql
│   ├── audit/
│   │   ├── run_audit.sql
│   │   ├── run_all_audits.sql
│   │   └── data_quality_checks.sql
│   └── views/
├── data/
│   ├── toast/
│   ├── r365/
│   ├── hubspot/
│   └── analytics/
├── powerbi/
│   └── Restaurant_BI.pbix
├── docs/
│   ├── Restaurant_BI_Documentation.docx
│   └── Restaurant_BI_Event_Log.docx
└── README.md
```

---

## Data Model

### Relationship Rules

- All relationships are **many-to-one** from fact to dimension
- `analytics.dim_date` is the single Date Table — marked in Power BI
- `dim_date[full_date]` must be typed as `DATE` for time-intelligence to work
- `fact_store_day` has **no channel_id** — channel breakdowns must use `fact_sales_order`
- `fact_pipeline_deal` has three date foreign keys; only `created_date_key` is active — use `USERELATIONSHIP()` for the others
- `fact_finance_month` connects to `dim_date` via `month_start_date = full_date` — axis must be monthly not daily

### Important Visual-to-Table Mapping

```
Channel revenue visuals    → fact_sales_order   (has channel_id)
Product / item visuals     → fact_sales_line    (has product_id)
Daily summary KPIs         → fact_store_day     (pre-aggregated)
Finance trend charts       → fact_finance_month (monthly grain, month axis only)
Pipeline analysis          → fact_pipeline_deal + fact_pipeline_realization
```

---

## Key DAX Patterns

### Slicer-Aware Window Logic (Line Charts)

Line chart measures cannot use single-date shifted logic. They require per-axis-date evaluation anchored to the slicer.

```dax
Net Sales Last 7 Days =
VAR SelectedDate = SELECTEDVALUE ( 'analytics dim_date'[full_date] )
VAR AxisDate     = MAX ( 'analytics dim_date'[full_date] )
RETURN
IF (
    AxisDate >= SelectedDate - 6 && AxisDate <= SelectedDate,
    CALCULATE (
        [Net Sales Order],
        REMOVEFILTERS ( 'analytics dim_date' ),
        'analytics fact_sales_order'[full_date] = AxisDate
    )
)
```

### ALLSELECTED vs ALL

`ALL()` removes slicer filters — avoid in window measures.  
`ALLSELECTED()` respects the slicer — always use this for user-driven date anchors.

```dax
-- ❌ Wrong — ignores slicer
VAR MaxDate = CALCULATE ( MAX ( dim_date[full_date] ), ALL ( dim_date ) )

-- ✅ Correct — respects slicer
VAR MaxDate = MAXX ( ALLSELECTED ( dim_date[full_date] ), dim_date[full_date] )
```

### Never Use TODAY() in Historical Datasets

```dax
-- ❌ Wrong in simulated/historical data
Pipeline Value Next 30 Days = FILTER ( ..., close_date >= TODAY() ... )

-- ✅ Correct — anchor to visible data
VAR SelectedDate = MAX ( 'analytics dim_date'[full_date] )
```

### Calculated Column vs Measure — Critical Distinction

> **A filter measure must always be a MEASURE, never a calculated column.**  
> A calculated column evaluates once at refresh with no slicer awareness.  
> A measure evaluates in context on every render, respecting all active filters.

```dax
-- ✅ Always create axis filter logic as a MEASURE
Show Last 7 Days Axis =
VAR MaxVisibleDate = MAXX ( ALLSELECTED ( dim_date[full_date] ), dim_date[full_date] )
VAR CurrentAxisDate = MAX ( dim_date[full_date] )
RETURN IF ( CurrentAxisDate >= MaxVisibleDate - 6 && CurrentAxisDate <= MaxVisibleDate, 1, 0 )
```

### Never Average Stored Ratio Columns

```dax
-- ❌ Wrong — averaging percentages across rows
Labor % = AVERAGE ( fact_store_day[labor_pct] )

-- ✅ Correct — calculate from component totals
Labor % = DIVIDE ( SUM ( fact_store_day[labor_cost] ), SUM ( fact_store_day[sales_net] ), 0 )
```

---

## Data Quality Framework

The project includes a SQL-based audit and data quality system that runs before connecting Power BI.

### Audit Functions

| Object | Purpose |
|---|---|
| `analytics.run_audit()` | Reusable function — checks row count, null keys, duplicates, date range, distinct count for any table |
| `analytics.run_all_audits()` | Runs all checks across every raw, dimension, and reporting fact table in one command |
| `analytics.run_data_quality_check()` | Logs a single named business-rule or relationship check result |
| `analytics.run_all_data_quality_checks()` | Relationship integrity checks across all fact-dimension pairs |
| `analytics.run_all_business_rule_checks()` | Formula validation, valid hour ranges, non-negative labor values |
| `analytics.run_all_grain_checks()` | Confirms correct row-level uniqueness per fact table |

### Running the Audit

```sql
-- Run full load audit across all tables
CALL analytics.run_all_audits();

-- Run business rule and grain checks
CALL analytics.run_all_data_quality_checks();
CALL analytics.run_all_business_rule_checks();
CALL analytics.run_all_grain_checks();

-- Query results
SELECT * FROM analytics.audit_run ORDER BY run_ts DESC;
SELECT * FROM analytics.data_quality_check_run WHERE status = 'FAIL';
```

---

## Known Limitations

| Area | Limitation |
|---|---|
| **Cash Flow** | Removed from all dashboards. Model lacks AR/AP, capex, debt schedule data. EBITDA is the only profitability metric. |
| **Forecasting** | `Forecast Revenue Proxy Next Month` is a 3-month rolling average — not a statistical model. Always label as *Estimate / Proxy*. |
| **Stage Conversion** | True funnel conversion rates are not possible. No stage history table exists — only current stage and final status are recorded. |
| **Inventory Variance** | Source data contains zero variance values. Variance KPIs are placeholder measures pending data regeneration. |
| **Marketing Attribution** | Marketing ROI uses HubSpot `won_revenue` as a proxy. No multi-touch attribution model. |
| **Subscription Revenue** | No recurring-billing logic. Wholesale channel (CH05) approximates subscription revenue. |

---

## Documentation

Full project documentation is in `/docs`:

| Document | Description |
|---|---|
| `Restaurant_BI_Documentation.docx` | Technical & functional reference — data model, KPI definitions, dashboard specs, full DAX reference |
| `Restaurant_BI_Event_Log.docx` | Chronological log of every major bug, fix, and design decision made during the build |
