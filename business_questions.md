# Business Questions & SQL Queries

This document contains all business analytical questions answered in this project, organized by module. Every query runs against the `analytics` schema views (`vw_supplier_performance`, `vw_supplier_performance`, `vw_procurement_analysis`, `vw_inventory_analysis`, `vw_demand_analysis`), which sit on top of the cleaned `master` schema tables.

---

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
