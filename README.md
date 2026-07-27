# Supply Chain & Logistics Analytics Project

An end-to-end data analytics project that identifies unreliable suppliers, quantifies their downstream impact on inventory health, and tracks demand-forecast accuracy — built with **PostgreSQL** (data cleaning, modeling, automation) and **Power BI / DAX** (dashboarding).

---

## Business Problem

A retail company orders products from 10 different global suppliers. Some suppliers deliver in as little as 5 days; others take up to 45 days. Because delivery times are unpredictable, warehouses either run completely out of stock or become overcrowded with excess inventory.

The goal: build a data pipeline and dashboard that identifies **which suppliers are actually unreliable** — not just slow on average, but inconsistent — so the purchasing team can make evidence-based decisions instead of guessing.

---

## Data Architecture

This project uses a 3-layer schema design, mirroring standard data-warehouse practice:

| Schema | Purpose |
|---|---|
| `staging` | Raw, as-loaded data — intentionally messy (mixed date formats, duplicates, NULLs, orphan foreign keys) to simulate real-world data-quality issues |
| `master` | Cleaned, typed, canonical source-of-truth tables — populated via SQL transformation from `staging` |
| `analytics` | Read-only reporting views (joins + derived flags) built on top of `master`, consumed directly by Power BI |

**Why this separation:**
- `staging` can be wiped and reloaded without touching cleaned data
- `master` stays the single source of truth, independent of how reporting logic evolves
- `analytics` views encode business logic once (e.g. reorder-point flags, late-delivery flags), so every dashboard measure sees a consistent definition

### Core Tables (`master`)
`suppliers` · `products` · `warehouses` · `purchase_orders` · `po_line_items` · `inventory_snapshot` · `supplier_performance_log` · `demand_forecast`

### Dimension Tables (`analytics`)
`dim_suppliers` · `dim_products` · `dim_warehouses` — small, deduplicated lookup tables used to build proper star-schema relationships in Power BI, so slicers filter every connected report page consistently.

### Reporting Views (`analytics`)
`vw_supplier_performance` · `vw_procurement_analysis` · `vw_inventory_analysis` · `vw_demand_analysis`

---

## Tech Stack

- **PostgreSQL** — data cleaning, transformation, view design, indexing, triggers, and stored procedures
- **Power BI + DAX** — interactive dashboard and measure logic

---

## Key Features

- **Data cleaning pipeline** — NULL handling, mixed date-format normalization, duplicate detection and removal, orphan foreign-key detection and cascading cleanup
- **Star-schema dimension tables** — proper Power BI relationships instead of wide, denormalized, unrelated fact tables
- **Indexed for performance** — targeted indexes on all foreign-key and filter columns used across the reporting views
- **5-page Power BI dashboard**:
  - **Supply Chain Logistic (Overview)** — top-line KPIs across the whole business
  - **Supplier Performance** — lead time, on-time %, late deliveries, by supplier and by tier
  - **Supplier Risk Analysis** — Coefficient of Variation ranking and a Low/Medium/High risk flag per supplier
  - **Procurement Analysis** — spend by supplier/category/warehouse, price variance across suppliers for the same product
  - **Inventory Analysis** — current stock, overstock, reorder-risk, and warehouse utilization
  - **Demand Forecast** — forecast accuracy and error % by warehouse, category, and product

---

## Key Insights

- **Coefficient of Variation reveals a different risk ranking than raw standard deviation.** Milano Fashion House has a lower raw lead-time standard deviation than Hanoi Textile Group, but a *higher* Coefficient of Variation (77.65% vs 47.27%) — meaning Milano is proportionally the more unpredictable supplier once its shorter average lead time is accounted for.
- **Supplier tier does not guarantee reliability.** Two "Strategic"-tier suppliers rank in the middle of the pack on both lead time and on-time delivery — tier alone is not a safe proxy for performance.
- **Only ~50% of delivered orders have a logged performance record.** This coverage gap means on-time/late percentages must be computed against *logged* orders, not all orders, to avoid silently understating reliability.
- **A handful of products show extreme cross-supplier price variance** (up to ~$498 difference for the same item), highlighting concrete renegotiation or re-sourcing opportunities.

---



---

## Dashboard Preview

*(Add exported screenshots or a link to the published Power BI report here.)*

---

## What This Project Demonstrates

- Designing a layered SQL data warehouse (staging → master → analytics) instead of working out of a single flat schema
- Diagnosing and fixing real data-quality defects: NULL-handling bugs in percentage calculations, date-format ambiguity, duplicate rows at scale, and cascading referential-integrity issues
- Building a proper Power BI star schema and understanding *why* missing relationships silently break slicers and charts
- Automating a metric with PostgreSQL triggers and stored procedures rather than relying on manual refresh alone
- Translating a business problem (unpredictable suppliers → stockouts/overcrowding) into a specific, defensible statistical metric (Coefficient of Variation) rather than a naive one (raw standard deviation)
