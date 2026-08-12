# Supply Chain & Logistics Analysis using Power BI

## Project Overview

**Project Title:** Supply Chain & Logistics Analysis

**Tools Used:** PostgreSQL (data warehouse), Power BI Desktop, DAX

**Data Source:** Multi-table supply chain dataset — purchase orders, deliveries, suppliers, warehouses, inventory, and demand forecasts

This project demonstrates end-to-end supply chain analytics — from a PostgreSQL data warehouse feeding curated analytics views, through to a 6-page interactive Power BI report covering delivery performance, supplier risk, procurement spend, inventory health, and demand forecast accuracy.

The project replicates a real-world logistics/procurement scenario: monitoring on-time delivery, identifying high-risk suppliers, tracking spend, flagging overstock/reorder risk across warehouses, and comparing forecasted vs actual demand.

## Business Problem

A retail company orders products from 10 different global suppliers. Some suppliers deliver early; others take up to many days. Because delivery times are unpredictable, warehouses either run completely out of stock or become overcrowded with excess inventory.

The goal: build a dashboard that identifies **which suppliers are actually unreliable** — not just slow on average, but inconsistent — so the purchasing team can make evidence-based decisions instead of guessing.

## Objectives

**Data Warehouse & Modeling**
Build curated analytics views on top of raw operational tables (`vw_supplier_performance`, `vw_inventory_analysis`, `vw_procurement_analysis`, `vw_demand_analysis`) linked to a shared `Dim_Date` date dimension, feeding a central `All_Measures` DAX measure table.

**Delivery Performance Monitoring**
Track on-time vs late vs undelivered orders at the company, supplier, and monthly level.

**Supplier Risk Analysis**
Quantify supplier delivery consistency using lead time variability (Coefficient of Variation) and classify suppliers into risk tiers.

**Procurement Spend Analysis**
Break down total spend by month, supplier, product category, and warehouse, and flag products with high price variance.

**Inventory Health Monitoring**
Track stock levels, overstock, reorder risk, and stockouts across five warehouses and eight product categories.

**Demand Forecast Accuracy**
Compare forecasted vs actual demand by year, warehouse, category, and product to measure forecasting accuracy and error.

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
`dim_suppliers` · `dim_products` · `dim_warehouses` — small, deduplicated lookup tables used to build a proper star-schema in Power BI, so slicers filter every connected report page consistently.

### Reporting Views (`analytics`)
`vw_supplier_performance` · `vw_supplier_performance_summary` · `vw_procurement_analysis` · `vw_inventory_analysis` · `vw_demand_analysis`

## Data Model (Power BI)

| Table / View | Purpose |
|---|---|
| `analytics vw_supplier_performance` | Supplier-level delivery, lead time, and risk metrics (supplier_name, supplier_tier, country) |
| `analytics vw_inventory_analysis` | Warehouse/category-level stock, overstock, and reorder metrics |
| `analytics vw_procurement_analysis` | Purchase order spend, quantity, and pricing by supplier/category/warehouse/product |
| `analytics vw_demand_analysis` | Forecasted vs actual demand by warehouse/category/product |
| `Dim_Date` | Shared date dimension (Year, Month, Date Hierarchy) for time intelligence across all pages |
| `All_Measures` | Central DAX measures table — all KPIs and calculations referenced by every visual |

Power BI connects via DirectQuery/Import to the `analytics` views rather than raw `staging`/`master` tables.

## Tech Stack

- **PostgreSQL** — schema creation, data cleaning, transformation, view design
- **Power BI + DAX** — DirectQuery mode, measure logic, interactive dashboarding

## Project Dashboard 

![Dashboard pdf](https://github.com/AnandKumar0/Supply-Chain-Logistics-Analytics-Project/blob/main/Supplly_Chain%20%26%20logistics_Project.pdf)


## Dashboard Structure (6 Pages)

### Page 1 — Executive Summary (Supply Chain Logistics Overview)

**KPI Cards**
- Total Purchase Orders: **146.19K**
- On Time Delivery: **31.31%**
- Late Delivery: **36.98%**
- No Data: **31.71%**
- Avg Lead Time: **14.88** days
- Total Spend: **9.31bn**
- Inventory Value: **8.65M**

**Order Status (Donut Chart)**
- Delivered: 84K (57.35%)
- Partial: 21K (14.41%)
- Pending: 21K (14.03%)
- Cancelled: 21K (14.21%)

**Month Wise On-Time Delivery Orders (Line Chart)**
Jan 3,400 → Feb 2,256 → Mar 3,420 → Apr 2,109 → May 2,182 → Jun 3,124 → Jul 3,408 → Aug 3,227 → Sep 2,298 → Oct 2,282 → Nov 2,933 → Dec 2,207

**Stock On Hand by Warehouse** — Delhi Central 9.2K, Mumbai Port 8.3K, Nagpur Distribution 8.0K, Kolkata Eastern Hub 7.9K, Bangalore Tech 7.1K

**On-Time Deliveries by Supplier** — Shenzhen Electronics Co 3.9K, Osaka Components Ltd 3.7K, Berlin Precision Gmbh 3.6K, Seoul Semiconductor Inc 3.4K, Chicago Steelworks 3.3K, Mumbai Textiles Pvt Ltd 3.3K, Monterrey Auto Parts 3.2K, Dhaka Garments Ltd 3.0K, Milano Fashion House 2.8K, Hanoi Textile Group 2.8K

**Overstock by Warehouse** — Delhi Central 34, Mumbai Port 31, Bangalore Tech 29, Kolkata Eastern Hub 29, Nagpur Distribution 28

**Storage Capacity by Warehouse** — Mumbai Port 75K, Nagpur Distribution 60K, Delhi Central 50K, Bangalore Tech 40K, Kolkata Eastern Hub 30K

---

### Page 2 — Supplier Performance

**Filters:** Country, Supplier Tier, Year, Supplier

**KPI Cards**
- Suppliers: **10**
- Avg Lead Time: **14.88**
- Avg Delay: **1.36**
- Lead Time Std Dev: **9.02**
- Performance Coverage %: **49.77**

**On-Time Delivery % by Supplier** — Shenzhen 37.69%, Osaka 35.19%, Berlin 34.14%, Seoul 32.58%, Mumbai Textiles 31.36%, Chicago 30.97%, Monterrey 29.91%, Dhaka 28.58%, Milano 26.61%, Hanoi 26.12%

**Late Deliveries by Supplier** — Hanoi 5.0K, Milano 4.9K, Dhaka 4.3K, Monterrey 4.2K, Chicago 4.0K, Mumbai Textiles 3.8K, Seoul 3.6K, Berlin 3.3K, Osaka 3.0K, Shenzhen 2.6K

**On-Time Delivery % by Month** — Jan 31.02%, Feb 31.11%, Mar 31.61%, Apr 30.83%, May 31.47%, Jun 31.56%, Jul 31.59%, Aug 31.21%, Sep 31.46%, Oct 31.51%, Nov 30.92%, Dec 31.21%

**Avg Lead Time by Supplier Tier** — Backup 17.45, Strategic 14.90, Preferred 12.29

**Average Lead Time by Supplier** — Hanoi 24.79, Dhaka 21.41, Milano 20.92, Monterrey 17.51, Mumbai Textiles 14.53, Chicago 13.62, Seoul 11.51, Berlin 9.51, Osaka 8.54, Shenzhen 6.49

---

### Page 3 — Supplier Risk Analysis

**Filters:** Supplier, Supplier Tier

**Supplier Risk Table**

| Supplier | Tier | Country | Total PO | Deliveries | Deliv % | On-Time % | Late % | No Data % | Avg Lead Time | Avg Delay | Std Dev | CoV % | Risk |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Milano Fashion House | Backup | Italy | 14,586 | 10,447 | 71.62% | 26.61% | 47.36% | 26.03% | 20.92 | 5.11 | 16.25 | 77.65 | 🔴 High |
| Hanoi Textile Group | Backup | Vietnam | 14,741 | 10,630 | 72.11% | 26.12% | 46.99% | 26.89% | 24.79 | 3.28 | 11.72 | 47.27 | 🔴 High |
| Dhaka Garments Ltd | Strategic | Bangladesh | 14,545 | 10,350 | 71.16% | 28.58% | 41.86% | 29.56% | 21.41 | 1.25 | 5.05 | 23.61 | 🟡 Medium |
| Shenzhen Electronics Co | Backup | China | 14,505 | 10,417 | 71.82% | 37.69% | 24.71% | 37.60% | 6.49 | 0.26 | 1.53 | 23.55 | 🟡 Medium |
| Monterrey Auto Parts | Backup | Mexico | 14,784 | 10,555 | 71.39% | 29.91% | 40.01% | 30.08% | 17.51 | 0.96 | 3.95 | 22.57 | 🟡 Medium |
| Chicago Steelworks | Preferred | USA | 14,674 | 10,633 | 72.46% | 30.97% | 37.55% | 31.48% | 13.62 | 0.70 | 2.99 | 21.92 | 🟡 Medium |
| Seoul Semiconductor Inc | Preferred | South Korea | 14,568 | 10,441 | 71.67% | 32.58% | 34.29% | 33.13% | 11.51 | 0.54 | 2.50 | 21.76 | 🟡 Medium |
| Berlin Precision Gmbh | Preferred | Germany | 14,636 | 10,570 | 72.22% | 34.14% | 31.11% | 34.75% | 9.51 | 0.41 | 2.02 | 21.21 | 🟡 Medium |
| Osaka Components Ltd | Strategic | Japan | 14,593 | 10,466 | 71.72% | 35.19% | 28.83% | 35.98% | 8.54 | 0.35 | 1.80 | 21.08 | 🟡 Medium |
| Mumbai Textiles Pvt Ltd | Preferred | India | 14,558 | 10,398 | 71.42% | 31.36% | 37.00% | 31.64% | 14.53 | 0.68 | 2.96 | 20.39 | 🟡 Medium |

**Coefficient of Variation % by Supplier** (mirrors table above) — Milano 77.65, Hanoi 47.27, Dhaka 23.61, Shenzhen 23.55, Monterrey 22.57, Chicago 21.92, Seoul 21.76, Berlin 21.21, Osaka 21.08, Mumbai Textiles 20.39

**Avg Lead Time vs Avg Delay by Supplier** — same ranking as the table's Avg Lead Time / Avg Delay columns, visualized as a clustered comparison.

---

### Page 4 — Procurement Analysis

**Filters:** Category, Year

**KPI Cards**
- Total Spend: **9.31bn**
- Avg PO Value: **63.84K**
- Avg Unit Cost: **207.86**
- Total PO: **146.19K**
- Total Purchase Qty: **37.12M**

**Monthly Spend** — Jan 0.97bn, Feb 0.64bn, Mar 0.60bn, Apr 0.62bn, May 0.96bn, Jun 0.88bn, Jul 0.96bn, Aug 0.64bn, Sep 0.93bn, Oct 0.64bn, Nov 0.63bn, Dec 0.83bn

**Spend by Supplier** — Chicago Steelworks 943.58M, Monterrey Auto Parts 943.34M, Berlin Precision Gmbh 942.90M, Hanoi Textile Group 937.04M, Mumbai Textiles Pvt Ltd 931.09M, Osaka Components Ltd 931.08M, Dhaka Garments Ltd 922.98M, Shenzhen Electronics Co 920.63M, Milano Fashion House 919.83M, Seoul Semiconductor Inc 917.53M

**Spend by Category** — Automotive Parts 1.31bn, Textiles 1.30bn, Electronics 1.30bn, Kitchenware 1.12bn, Furniture 1.12bn, Fashion Accessories 1.11bn, Footwear 1.11bn, Toys 0.94bn

**Spend by Warehouse** — Nagpur Distribution 1.88bn, Mumbai Port 1.87bn, Delhi Central 1.86bn, Bangalore Tech 1.85bn, Kolkata Eastern Hub 1.85bn

**Top Purchase Products (by Qty)** — Canvas Tote Fabric 764.98K, USB-C Fast Charger 761.19K, Electric Kettle 1.5L 760.54K, Building Blocks Set 757.67K, Noise Cancelling [Headphones] 754.41K

**Price Variance Table** (top rows by variation)

| Product | Max Price | Min Price | Price Variation |
|---|---|---|---|
| Foldable Bookshelf | 499.99 | 2.00 | 497.99 |
| Bedside Nightstand | 500.00 | 2.02 | 497.98 |
| Remote Control Car | 499.97 | 2.00 | 497.97 |
| Linen Table Runner | 499.99 | 2.04 | 497.95 |
| Kids Art Supply Kit | 499.97 | 2.03 | 497.94 |
| Denim Fabric Roll | 499.95 | 2.01 | 497.94 |
| Sunglasses UV Protection | 499.98 | 2.04 | 497.94 |
| Bluetooth Speaker Mini | 499.98 | 2.05 | 497.93 |
| Building Blocks Set 200pc | 499.98 | 2.05 | 497.93 |
| Canvas Backpack | 499.96 | 2.03 | 497.93 |
| Leather Wallet Men | 499.99 | 2.06 | 497.93 |

---

### Page 5 — Inventory Analysis

**Filters:** Warehouse, Category

**KPI Cards**
- Inventory Value: **8.65M**
- Total Stock: **40K**
- Overstock: **151**
- Below Reorder Point: **4**
- Stockout Count: **0**

**Inventory by Warehouse** — Delhi Central 1.88M, Nagpur Distribution 1.83M, Mumbai Port 1.78M, Bangalore Tech 1.73M, Kolkata Eastern Hub 1.43M

**Inventory by Category** — Electronics 1.72M, Kitchenware 1.65M, Fashion Accessories 1.49M, Textiles 1.00M, Footwear 0.77M, Toys 0.69M, Automotive Parts 0.69M, Furniture 0.64M

**Reorder Risk by Category** — Electronics 23, Textiles 21, Automotive Parts 19, Furniture 19, Footwear 18, Kitchenware 18, Fashion Accessories 17, Toys 16

**Overstock by Category** — Kitchenware 2, Fashion Accessories 1, Furniture 1

**Overstock by Warehouse** — Delhi Central 34, Mumbai Port 31, Bangalore Tech 29, Kolkata Eastern Hub 29, Nagpur Distribution 28

**Warehouse Info Table**

| Warehouse | Capacity | Total Stock | Overstock | Utilization % |
|---|---|---|---|---|
| Kolkata Eastern Hub | 30,000 | 7,887 | 29 | 26.29% |
| Delhi Central Warehouse | 50,000 | 9,179 | 34 | 18.36% |
| Bangalore Tech Warehouse | 40,000 | 7,123 | 29 | 17.81% |
| Nagpur Distribution Center | 60,000 | 7,963 | 28 | 13.27% |
| Mumbai Port Warehouse | 75,000 | 8,280 | 31 | 11.04% |
| **Total** | **255,000** | **40,432** | **151** | **15.86%** |

---

### Page 6 — Demand Forecast

**Filters:** Warehouse, Category

**KPI Cards**
- Forecast Accuracy %: **74.69**
- Forecast Error %: **25.31**
- Forecasted Demand: **45.96M**
- Actual Demand: **41.42M**

**Forecast vs Actual Demand by Year** — 2024: Forecasted 18.7M / Actual 16.9M · 2025: Forecasted 8.7M / Actual 7.8M · 2026: Forecasted 18.6M / Actual 16.8M

**Forecast Accuracy by Warehouse** — Kolkata Eastern Hub 74.93, Mumbai Port 74.70, Nagpur Distribution 74.69, Bangalore Tech 74.57, Delhi Central 74.55

**Demand by Product (Top 5, Actual)** — Women's Leather Sandals 1.13M, Denim Fabric Roll 1.13M, Ceramic Dinner Plate Set 1.12M, Wooden Cutting Board 1.12M, Engine Oil Filter 1.12M

**Demand by Warehouse (Actual)** — Kolkata Eastern Hub 9.0M, Mumbai Port 8.5M, Bangalore Tech 8.3M, Delhi Central 7.9M, Nagpur Distribution 7.8M

**Forecast vs Actual Demand by Category** — Textiles 7.2M/6.5M, Electronics 6.7M/6.0M, Automotive Parts 6.5M/5.8M, Kitchenware 5.7M/5.2M, Footwear 5.7M/5.1M, Fashion Accessories 5.2M/4.7M, Furniture 5.0M/4.5M, Toys 4.0M/3.6M *(Forecasted/Actual)*

**Forecast vs Actual Demand by Warehouse Table**

| Warehouse | Forecasted Demand | Actual Demand | Accuracy % | Error % |
|---|---|---|---|---|
| Nagpur Distribution Center | 8,683,749 | 7,801,535 | 74.69 | 25.31 |
| Mumbai Port Warehouse | 9,412,347 | 8,479,482 | 74.70 | 25.30 |
| Kolkata Eastern Hub | 9,976,637 | 9,005,600 | 74.93 | 25.07 |
| Delhi Central Warehouse | 8,709,930 | 7,859,355 | 74.55 | 25.45 |
| Bangalore Tech Warehouse | 9,181,678 | 8,273,353 | 74.57 | 25.43 |

**Forecast Error % by Warehouse** — Delhi Central 25.5, Bangalore Tech 25.4, Nagpur Distribution 25.3, Mumbai Port 25.3, Kolkata Eastern Hub 25.1

---

# All Business Questions & SQL Queries



## Supplier Reliability & Performance

### 1. Which suppliers deliver late most often?
**Business Purpose:** Identify suppliers causing the highest number of delayed deliveries.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN on_time_flag = false THEN 1 ELSE 0 END) AS late_orders,
	ROUND(SUM(CASE WHEN on_time_flag = false THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS late_delivery_percentage
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY late_delivery_percentage DESC;
```

### 2. What is the average lead time for each supplier?
**Business Purpose:** Identify suppliers taking the longest time to fulfill orders.
```sql
SELECT
	supplier_id,
	supplier_name,
	ROUND(AVG(lead_time_days), 2) AS avg_lead_time_days
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 3 DESC;
```


### 3. Which suppliers have the highest Lead Time Variability?
**Business Purpose:** Find suppliers whose delivery times are inconsistent.
```sql
SELECT
	supplier_id,
	supplier_name,
	ROUND(AVG(lead_time_days), 2) AS avg_lead_time,
	ROUND(STDDEV(lead_time_days), 2) AS lead_time_standard_deviation
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 4 DESC;
```

### 4. Are Strategic suppliers actually more reliable?
**Business Purpose:** Compare supplier tiers.
```sql
SELECT
	supplier_tier,
	COUNT(*) AS total_orders,
	ROUND(AVG(lead_time_days), 2) AS avg_lead_time,
	ROUND(SUM(CASE WHEN on_time_flag = true THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_pct
FROM analytics.vw_supplier_performance
GROUP BY 1
ORDER BY 4 DESC;
```

### 5. Which suppliers have the highest average delivery delay?
**Business Purpose:** Identify suppliers that exceed promised delivery dates by the largest margin.
```sql
SELECT
	supplier_id,
	supplier_name,
	ROUND(AVG(delay_days), 2) AS avg_delay_days
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY avg_delay_days DESC;
```
### 6. Supplier Performance Trend
**Business Purpose:** Measure whether supplier performance is improving or declining.
```sql
SELECT
    supplier_id,
    supplier_name,
    order_year,
    order_month,
    order_month_name,
    ROUND(AVG(lead_time_days), 2) AS average_lead_time,
    ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_delivery_percentage
FROM analytics.vw_supplier_performance
GROUP BY 1, 2, 3, 4, 5
ORDER BY 2, 3, 4;
```

### 7. Which supplier has the highest delivery risk?
**Business Purpose:** Identify suppliers most likely to impact operations.
```sql
SELECT
    supplier_id,
    supplier_name,
    ROUND(AVG(lead_time_days), 2) AS average_lead_time,
    ROUND(AVG(delay_days), 2) AS average_delay,
    ROUND(COUNT(*) FILTER(WHERE late_delivery_flag = TRUE) * 100.0 / COUNT(late_delivery_flag), 2) AS late_delivery_percentage
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 4, 5;
```
---

## Procurement & Cost Analysis

### 1. Which supplier has the highest procurement spend?
**Business Purpose:** Identify suppliers contributing the highest procurement cost.
```sql
SELECT
    supplier_id,
    supplier_name,
    ROUND(SUM(procurement_amount), 2) AS total_procurement_spend
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 2. How much does the same product's price vary across suppliers?
**Business Purpose:** Identify pricing differences for identical products.
```sql
SELECT
    product_id,
    product_name,
    ROUND(MIN(unit_price), 2) AS minimum_price,
    ROUND(MAX(unit_price), 2) AS maximum_price,
    ROUND(MAX(unit_price) - MIN(unit_price), 2) AS price_variation
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 5 DESC;
```

### 3. Which suppliers are expensive but reliable?
**Business Purpose:** Evaluate the cost versus reliability trade-off.
```sql
SELECT
    supplier_id,
    supplier_name,
    ROUND(AVG(unit_price), 2) AS average_unit_price,
    ROUND(AVG(lead_time_days), 2) AS average_lead_time,
    ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_delivery_percentage
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 4 ASC;
```

### 4. Monthly Procurement Spend
**Business Purpose:** Track how procurement spend changes over time.
```sql
SELECT
    EXTRACT(YEAR FROM po.order_date) AS order_year,
    EXTRACT(MONTH FROM po.order_date) AS order_month,
    ROUND(SUM(vpa.procurement_amount), 2) AS monthly_spend
FROM analytics.vw_procurement_analysis vpa
JOIN master.purchase_orders po ON vpa.po_id = po.po_id
GROUP BY 1, 2
ORDER BY 1, 2;
```
---

## Inventory Analysis

### 1. Stockout Products (current status)
**Business Purpose:** Identify products that are completely out of stock right now.
```sql
SELECT product_name, warehouse_name, stock_on_hand, snapshot_date
FROM (
	SELECT DISTINCT ON (warehouse_id, product_id)
		product_name, warehouse_name, stock_on_hand, stockout_flag, snapshot_date
	FROM analytics.vw_inventory_analysis
	ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
WHERE stockout_flag = true;
```

### 2. Overstock Products (current status)
**Business Purpose:** Identify products significantly exceeding their reorder threshold.
```sql
SELECT product_name, warehouse_name, stock_on_hand, reorder_point, snapshot_date
FROM (
	SELECT DISTINCT ON (warehouse_id, product_id)
		product_name, warehouse_name, stock_on_hand, reorder_point, overstock_flag, snapshot_date
	FROM analytics.vw_inventory_analysis
	ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
WHERE overstock_flag = true;
```

### 3. Products Below Reorder Point (current status)
**Business Purpose:** Flag products that need to be reordered soon.
```sql
SELECT product_name, warehouse_name, stock_on_hand, reorder_point
FROM (
	SELECT DISTINCT ON (warehouse_id, product_id)
		product_name, warehouse_name, stock_on_hand, reorder_point, below_reorder_point_flag, snapshot_date
	FROM analytics.vw_inventory_analysis
	ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
WHERE below_reorder_point_flag = true;
```


### 4. Current Stock by Warehouse
**Business Purpose:** Measure total inventory volume held at each warehouse right now.
```sql
SELECT warehouse_name, SUM(stock_on_hand) AS current_total_stock
FROM (
    SELECT DISTINCT ON (warehouse_id, product_id)
        warehouse_name, warehouse_id, product_id, stock_on_hand
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
GROUP BY 1
ORDER BY 2 DESC;
```

### 5. Which warehouse has the highest stockout/reorder-risk frequency?
**Business Purpose:** Identify warehouses that most often run low on stock.
```sql
SELECT warehouse_name,
    COUNT(*) FILTER (WHERE below_reorder_point_flag=TRUE) AS below_reorder_events,
    COUNT(*) AS total_snapshots,
    ROUND(100.0*COUNT(*) FILTER (WHERE below_reorder_point_flag=TRUE)/COUNT(*),2) AS pct
FROM analytics.vw_inventory_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 6. Which warehouse has the highest storage utilization?
**Business Purpose:** Measure how much of each warehouse's capacity is currently in use.
```sql
SELECT warehouse_name, current_stock, storage_capacity, utilization_pct
FROM (
    SELECT warehouse_name, storage_capacity, SUM(stock_on_hand) AS current_stock,
        ROUND(100.0 * SUM(stock_on_hand)/storage_capacity,2) AS utilization_pct
    FROM (
        SELECT DISTINCT ON (warehouse_id, product_id)
            warehouse_name, warehouse_id, product_id, stock_on_hand, storage_capacity
        FROM analytics.vw_inventory_analysis
        ORDER BY warehouse_id, product_id, snapshot_date DESC
    ) latest
    GROUP BY 1, 2
) t
ORDER BY 4 DESC;
```

### 7. Inventory Value (overall, current)
**Business Purpose:** Measure total capital tied up in current inventory.
```sql
SELECT ROUND(SUM(inventory_value),2) AS total_inventory_value
FROM (
    SELECT DISTINCT ON (warehouse_id, product_id) inventory_value
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest;
```

### 8. Inventory Value by Warehouse
**Business Purpose:** Identify which warehouses hold the most capital in stock.
```sql
SELECT warehouse_name, ROUND(SUM(inventory_value),2) AS warehouse_inventory_value
FROM (
    SELECT DISTINCT ON (warehouse_id, product_id)
        warehouse_name, warehouse_id, product_id, inventory_value
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
GROUP BY 1
ORDER BY 2 DESC;
```

### 9. Monthly Stock-on-Hand Trend
**Business Purpose:** Analyze how average inventory levels change over time.
```sql
SELECT snapshot_year, snapshot_month, ROUND(AVG(stock_on_hand),2) AS avg_stock
FROM analytics.vw_inventory_analysis
GROUP BY 1, 2
ORDER BY 1, 2;
```
---

## Demand Forecast Analysis

### 1. Forecast Error %
**Business Purpose:** Measure how far off the forecast typically is from actual demand.
```sql
SELECT product_name, ROUND(AVG(forecast_error_pct),2) AS avg_error_pct
FROM analytics.vw_demand_analysis
WHERE actual_demand IS NOT NULL
GROUP BY product_name
ORDER BY avg_error_pct DESC;
```

### 2. Under-Forecast Products
**Business Purpose:** Identify products where actual demand consistently exceeds the forecast.
```sql
SELECT product_name, COUNT(*) AS under_forecast_count
FROM analytics.vw_demand_analysis
WHERE forecast_status = 'Under Forecast'
GROUP BY 1
ORDER BY 2 DESC;
```
### 3. Warehouse Demand
**Business Purpose:** Compare total demand handled across warehouses.
```sql
SELECT warehouse_name, SUM(actual_demand) AS total_actual_demand, SUM(forecasted_demand) AS total_forecasted_demand
FROM analytics.vw_demand_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 4. Demand vs Supply Gap
**Business Purpose:** Compare forecasted demand against current stock to assess if enough is being ordered.
```sql
SELECT
    d.product_name, d.warehouse_name, d.forecasted_demand,
    inv.stock_on_hand AS current_stock,
    (inv.stock_on_hand - d.forecasted_demand) AS supply_gap
FROM analytics.vw_demand_analysis d
JOIN (
    SELECT DISTINCT ON (warehouse_id, product_id)
        warehouse_id, product_id, stock_on_hand
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) inv ON inv.warehouse_id = d.warehouse_id AND inv.product_id = d.product_id
WHERE d.forecast_month_date = (SELECT MAX(forecast_month_date) FROM analytics.vw_demand_analysis)
ORDER BY 5 ASC;
```

### 5. Seasonal/Peak Demand Months
**Business Purpose:** Identify months with consistently higher demand across years.
```sql
SELECT
	forecast_month_number, TO_CHAR(forecast_month,'Month') AS month_name,
    COUNT(DISTINCT forecast_year) AS years_covered,
    ROUND(SUM(actual_demand)::NUMERIC / COUNT(DISTINCT forecast_year),2) AS avg_demand_per_year
FROM analytics.vw_demand_analysis
GROUP BY 1, 2
ORDER BY 1 ASC;
```


## Findings

- **Delivery reliability is a company-wide problem, not a single supplier issue** — even the best supplier (Shenzhen Electronics, 37.69% on-time) is well below an acceptable on-time target, and "No Data" status alone accounts for ~31.71% of all orders — suggesting a delivery-status tracking/data capture gap as much as an operational one.
- **Two suppliers are High Risk** — Milano Fashion House (CoV 77.65) and Hanoi Textile Group (CoV 47.27) show by far the most inconsistent lead times of the 10 suppliers, driven by both longer average lead time and higher delay variance — these are the suppliers most likely to cause downstream stockouts.
- **Backup-tier suppliers carry the most risk** — both High Risk suppliers are tagged "Backup" tier, and Backup tier also has the highest average lead time (17.45 vs 14.90 Strategic and 12.29 Preferred) — worth reviewing whether "Backup" suppliers are being over-relied upon.
- **Spend is evenly distributed, price variance is not** — total spend per supplier clusters tightly (917M–944M each), but individual products show extreme price variance (up to ~498 between max and min unit price for the same item), which points to a pricing/data-quality issue worth investigating before it's treated as normal.
- **Inventory is well-controlled overall** — zero stockouts and only 4 items below reorder point against 40K total stock is healthy, though 151 overstocked items (concentrated in Delhi Central and Mumbai Port warehouses) ties up working capital unnecessarily.
- **Forecast accuracy is consistent but has room to improve** — ~74.7% accuracy is stable across all 5 warehouses (a tight 74.55–74.93% range), meaning the forecasting model is consistently biased rather than erratic — a systematic adjustment could lift accuracy across the board rather than needing warehouse-specific fixes.
- **Coefficient of Variation reveals a different risk ranking than raw standard deviation alone.** Milano Fashion House has both the highest CoV (77.65%) and a higher lead-time standard deviation (16.25 days) than Hanoi Textile Group (47.27%, 11.72 days) — confirming Milano as the single most unpredictable supplier, not just a slow one.
- **Supplier tier does not guarantee reliability.** Some "Strategic"-tier suppliers rank only in the middle of the pack on both lead time and on-time delivery — tier alone is not a safe proxy for actual performance.
- **Only ~50% of delivered orders have a logged performance record** (Performance Coverage % = 49.77 on the Supplier Performance page). This means on-time/late percentages must be computed against *logged* orders, not all orders — otherwise reliability looks worse than it may actually be, or the gap itself becomes a data-tracking issue to fix.

## Skills Demonstrated

✔ Layered SQL Data Warehouse Design (staging → master → analytics)
✔ Data Cleaning (NULL handling, mixed date-format normalization, duplicate removal, orphan foreign-key cleanup)
✔ Advanced SQL (window-style `DISTINCT ON` for latest-snapshot logic, `FILTER`, `STDDEV`, correlated subqueries, joins across schemas)
✔ Star-Schema Dimension Modeling (`dim_suppliers`, `dim_products`, `dim_warehouses`)
✔ DAX Measures (KPI aggregation, % calculations, risk classification logic)
✔ Multi-Page Dashboard Design (6 linked report pages)
✔ Supplier Risk Scoring (Coefficient of Variation-based)
✔ Time Intelligence (monthly/yearly trend analysis via shared date dimension)
✔ Cross-Filtering with Slicers (Country, Tier, Year, Warehouse, Category, Supplier)
✔ Business Storytelling Across Procurement, Inventory, and Forecasting Domains

## Conclusion

This project strengthened practical skills in building a layered, warehouse-backed Power BI solution that spans the full supply chain lifecycle — supplier reliability, procurement spend, inventory health, and demand forecasting — on a single connected data model.

The insights generated can support decision-making around supplier renegotiation/replacement, safety-stock and reorder-point tuning, warehouse capacity planning, and forecast model calibration.

## How To Use

1. Download or clone this repository.
2. Open `Supply_Chain___logistics_Project.pbix` in Power BI Desktop.
3. If prompted, update the data source connection to point to your PostgreSQL analytics views (or local copy of the dataset).
4. Refresh the data model.
5. Navigate through the 6 report pages (Executive Summary, Supplier Performance, Supplier Risk, Procurement Analysis, Inventory Analysis, Demand Forecast) and use the filters on each page to drill into specific suppliers, warehouses, categories, or time periods.

## Author

**Anand Kumar**
MIS Executive | DAta Analst |

PostgreSQL | Power BI | DAX | 

This project is part of my Data Analytics portfolio showcasing end-to-end supply chain analytics skills required for Data Analyst roles.

Feel free to connect, provide feedback, or collaborate on future projects.

