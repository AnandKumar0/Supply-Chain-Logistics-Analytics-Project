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

### 3. What is the On-Time Delivery Percentage?
**Business Purpose:** Measure supplier delivery reliability.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN on_time_flag = true THEN 1 ELSE 0 END) AS on_time_orders,
	ROUND(SUM(CASE WHEN on_time_flag = true THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_delivery_pct
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 4 DESC;
```

### 4. Which suppliers have the highest Lead Time Variability?
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

### 5. Are Strategic suppliers actually more reliable?
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

### 6. Supplier Performance Ranking
**Business Purpose:** Rank suppliers based on reliability.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	ROUND(AVG(lead_time_days), 2) AS avg_lead_time,
	ROUND(SUM(CASE WHEN on_time_flag = true THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_pct,
	RANK() OVER(ORDER BY ROUND(SUM(CASE WHEN on_time_flag = true THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) DESC) AS supplier_rank
FROM analytics.vw_supplier_performance
GROUP BY 1, 2;
```

### 7. Which suppliers have the highest average delivery delay?
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

### 8. Which suppliers deliver earliest most often?
**Business Purpose:** Identify suppliers consistently delivering before the promised date.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS early_deliveries
FROM analytics.vw_supplier_performance
WHERE delivery_variance_days < 0
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 9. Which suppliers completed the highest number of purchase orders?
**Business Purpose:** Identify suppliers handling the largest procurement volume.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(po_id) AS completed_purchase_orders
FROM analytics.vw_supplier_performance
WHERE po_status = 'Delivered'
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 10. Which supplier tier has the highest On-Time Delivery Percentage?
**Business Purpose:** Compare delivery performance across supplier tiers.
```sql
SELECT
	supplier_tier,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) AS on_time_orders,
	ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_pct
FROM analytics.vw_supplier_performance
GROUP BY 1
ORDER BY 4;
```

### 11. Which supplier tier contributes the highest number of purchase orders?
**Business Purpose:** Analyze procurement distribution by supplier tier.
```sql
SELECT
	supplier_tier,
	COUNT(po_id) AS purchase_orders
FROM analytics.vw_supplier_performance
GROUP BY 1
ORDER BY 2 DESC;
```

### 12. Total Purchase Orders per Supplier
**Business Purpose:** Measure procurement volume handled by each supplier.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(po_id) AS total_orders
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 13. Delivered vs Pending vs Cancelled Orders
**Business Purpose:** Understand the operational status of purchase orders.
```sql
SELECT
	po_status,
	COUNT(po_id) AS total_orders
FROM analytics.vw_supplier_performance
GROUP BY 1
ORDER BY 2 DESC;
```

### 14. Cancellation Rate by Supplier
**Business Purpose:** Identify suppliers with high order cancellation rates.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN po_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_orders,
	ROUND(SUM(CASE WHEN po_status = 'Cancelled' THEN 1 ELSE 0 END) * 100.0 / COUNT(po_status), 2) AS cancellation_rate
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 5 DESC;
```

### 15. Pending Order Percentage by Supplier
**Business Purpose:** Measure suppliers with excessive pending orders.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN po_status = 'Pending' THEN 1 ELSE 0 END) AS pending_orders,
	ROUND(SUM(CASE WHEN po_status = 'Pending' THEN 1 ELSE 0 END) * 100.0 / COUNT(po_status), 2) AS pending_pct
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 5 DESC;
```

### 16. Delivery Completion Percentage
**Business Purpose:** Measure successful order completion across suppliers.
```sql
SELECT
	supplier_id,
	supplier_name,
	COUNT(*) AS total_orders,
	SUM(CASE WHEN po_status = 'Delivered' THEN 1 ELSE 0 END) AS delivered_orders,
	ROUND(SUM(CASE WHEN po_status = 'Delivered' THEN 1 ELSE 0 END) * 100.0 / COUNT(po_status), 2) AS delivery_completed_pct
FROM analytics.vw_supplier_performance
GROUP BY 1, 2
ORDER BY 5 DESC;
```

### 17. Monthly Lead Time Trend
**Business Purpose:** Analyze how lead time changes over time.
```sql
SELECT
    order_year,
    order_month,
    order_month_name,
    ROUND(AVG(lead_time_days), 2) AS average_lead_time
FROM analytics.vw_supplier_performance
GROUP BY 1, 2, 3
ORDER BY 1, 2 ASC;
```

### 18. Monthly On-Time Delivery Percentage
**Business Purpose:** Monitor delivery reliability month by month.
```sql
SELECT
    order_year,
    order_month,
    order_month_name,
    ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_delivery_percentage
FROM analytics.vw_supplier_performance
GROUP BY 1, 2, 3
ORDER BY 1, 2 ASC;
```

### 19. Monthly Delayed Orders Trend
**Business Purpose:** Track delayed deliveries over time.
```sql
SELECT
    order_year,
    order_month,
    order_month_name,
    COUNT(*) FILTER(WHERE late_delivery_flag = TRUE) AS delayed_orders
FROM analytics.vw_supplier_performance
GROUP BY 1, 2, 3
ORDER BY 1, 2 ASC;
```

### 20. Supplier Performance Trend
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

### 21. Which supplier has the highest delivery risk?
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

### 22. Overall Average Lead Time
**Business Purpose:** Measure overall procurement efficiency.
```sql
SELECT ROUND(AVG(lead_time_days), 2) AS overall_average_lead_time
FROM analytics.vw_supplier_performance;
```

### 23. Overall On-Time Delivery Percentage
**Business Purpose:** Measure overall supplier reliability.
```sql
SELECT
    ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS overall_on_time_delivery_percentage
FROM analytics.vw_supplier_performance;
```

### 24. Overall Late Delivery Percentage
**Business Purpose:** Measure the overall percentage of delayed deliveries.
```sql
SELECT
    ROUND(COUNT(*) FILTER(WHERE late_delivery_flag = TRUE) * 100.0 / COUNT(late_delivery_flag), 2) AS overall_late_delivery_percentage
FROM analytics.vw_supplier_performance;
```

### 25. Total Suppliers
**Business Purpose:** Measure the size of the supplier base.
```sql
SELECT COUNT(DISTINCT supplier_id) AS total_suppliers
FROM analytics.vw_supplier_performance;
```

### 26. Active Suppliers
**Business Purpose:** Identify suppliers currently participating in procurement.
```sql
SELECT
    COUNT(DISTINCT supplier_id) AS active_suppliers
FROM analytics.vw_supplier_performance
WHERE po_status IN ('Delivered', 'Partial', 'Pending');
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

### 2. What is the average unit price by supplier?
**Business Purpose:** Compare supplier pricing.
```sql
SELECT
    supplier_id,
    supplier_name,
    ROUND(AVG(unit_price), 2) AS avg_unit_price
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 3. How much does the same product's price vary across suppliers?
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

### 4. Which suppliers are expensive but reliable?
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

### 5. Overall Supplier Performance Score
**Business Purpose:** Create a composite supplier performance score using multiple KPIs.
```sql
SELECT
    supplier_id,
    supplier_name,
    ROUND(AVG(lead_time_days), 2) AS average_lead_time,
    ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) AS on_time_delivery_percentage,
    ROUND(AVG(unit_price), 2) AS average_unit_price,
    ROUND(SUM(procurement_amount), 2) AS total_procurement_spend,
    RANK() OVER (ORDER BY
            ROUND(SUM(CASE WHEN on_time_flag = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(on_time_flag), 2) DESC,
            AVG(lead_time_days), SUM(procurement_amount) DESC) AS supplier_performance_rank
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 7;
```

### 6. Monthly Procurement Spend
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

### 7. Spend by Category
**Business Purpose:** Identify which product categories consume the largest share of procurement budget.
```sql
SELECT
    p.category,
    ROUND(SUM(vpa.procurement_amount), 2) AS category_spend
FROM analytics.vw_procurement_analysis vpa
JOIN master.products p ON vpa.product_id = p.product_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 8. Average PO Value
**Business Purpose:** Measure the typical size of a purchase order.
```sql
SELECT
    ROUND(AVG(po_total), 2) AS average_po_value
FROM (
    SELECT po_id, SUM(procurement_amount) AS po_total
    FROM analytics.vw_procurement_analysis
    GROUP BY 1
) t;
```

### 9. Top Purchased Products
**Business Purpose:** Identify the highest-demand products by quantity ordered.
```sql
SELECT
    product_id,
    product_name,
    SUM(order_qty) AS total_qty_ordered,
    ROUND(SUM(procurement_amount), 2) AS total_spend
FROM analytics.vw_procurement_analysis
GROUP BY 1, 2
ORDER BY 3 DESC;
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

### 4. Stock Status Breakdown
**Business Purpose:** Understand the overall health distribution of inventory across all snapshots.
```sql
SELECT
    SUM(CASE WHEN stockout_flag THEN 1 ELSE 0 END) AS stockout_count,
    SUM(CASE WHEN below_reorder_point_flag THEN 1 ELSE 0 END) AS below_reorder_count,
    SUM(CASE WHEN overstock_flag THEN 1 ELSE 0 END) AS overstock_count,
    COUNT(*) AS total_snapshots
FROM analytics.vw_inventory_analysis;
```

### 5. Current Stock by Warehouse
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

### 6. Which warehouse has the highest stockout/reorder-risk frequency?
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

### 7. Which warehouse has the highest storage utilization?
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

### 8. Which warehouse holds the most product variety?
**Business Purpose:** Identify warehouses stocking the broadest range of distinct products.
```sql
SELECT warehouse_name, COUNT(DISTINCT product_id) AS distinct_products
FROM analytics.vw_inventory_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 9. Inventory Value (overall, current)
**Business Purpose:** Measure total capital tied up in current inventory.
```sql
SELECT ROUND(SUM(inventory_value),2) AS total_inventory_value
FROM (
    SELECT DISTINCT ON (warehouse_id, product_id) inventory_value
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest;
```

### 10. Inventory Value by Warehouse
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

### 11. Inventory Value by Category
**Business Purpose:** Identify which product categories tie up the most capital.
```sql
SELECT category, ROUND(SUM(inventory_value),2) AS category_inventory_value
FROM (
    SELECT DISTINCT ON (warehouse_id, product_id) category,
        warehouse_id, product_id, inventory_value
    FROM analytics.vw_inventory_analysis
    ORDER BY warehouse_id, product_id, snapshot_date DESC
) latest
GROUP BY 1
ORDER BY 2 DESC;
```

### 12. Which product has the highest reorder-risk frequency?
**Business Purpose:** Identify products that repeatedly need reordering.
```sql
SELECT product_name,
    COUNT(*) FILTER (WHERE below_reorder_point_flag=TRUE) AS below_reorder_events,
    COUNT(*) AS total_snapshots,
    ROUND(100.0 * COUNT(*) FILTER (WHERE below_reorder_point_flag=TRUE)/COUNT(*),2) AS pct
FROM analytics.vw_inventory_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 13. Which product-warehouse combination has the most volatile stock levels?
**Business Purpose:** Identify unstable inventory patterns requiring closer monitoring.
```sql
SELECT product_name, warehouse_name, ROUND(STDDEV(stock_on_hand),2) AS stock_stddev
FROM analytics.vw_inventory_analysis
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 14. Monthly Stock-on-Hand Trend
**Business Purpose:** Analyze how average inventory levels change over time.
```sql
SELECT snapshot_year, snapshot_month, ROUND(AVG(stock_on_hand),2) AS avg_stock
FROM analytics.vw_inventory_analysis
GROUP BY 1, 2
ORDER BY 1, 2;
```

### 15. Which products show a declining stock trend?
**Business Purpose:** Identify products being depleted over time without replenishment.
```sql
WITH bounds AS (
    SELECT product_id, warehouse_id,
        MIN(snapshot_date) AS first_date, MAX(snapshot_date) AS last_date
    FROM analytics.vw_inventory_analysis
    GROUP BY 1, 2
)
SELECT
	p.product_name, w.warehouse_name,
    f.stock_on_hand AS first_stock, l.stock_on_hand AS last_stock,
    (l.stock_on_hand - f.stock_on_hand) AS change
FROM bounds b
JOIN analytics.vw_inventory_analysis f ON f.product_id = b.product_id AND f.warehouse_id = b.warehouse_id AND f.snapshot_date = b.first_date
JOIN analytics.vw_inventory_analysis l ON l.product_id = b.product_id AND l.warehouse_id = b.warehouse_id AND l.snapshot_date = b.last_date
JOIN master.products p ON p.product_id = b.product_id
JOIN master.warehouses w ON w.warehouse_id = b.warehouse_id
ORDER BY 5 ASC;
```

### 16. Which products show a rising/overstock-building trend?
**Business Purpose:** Identify products accumulating excess stock over time.
```sql
WITH bounds AS (
    SELECT product_id, warehouse_id,
        MIN(snapshot_date) AS first_date, MAX(snapshot_date) AS last_date
    FROM analytics.vw_inventory_analysis
    GROUP BY 1, 2
)
SELECT
	p.product_name, w.warehouse_name,
    f.stock_on_hand AS first_stock, l.stock_on_hand AS last_stock,
    (l.stock_on_hand - f.stock_on_hand) AS change
FROM bounds b
JOIN analytics.vw_inventory_analysis f ON f.product_id = b.product_id AND f.warehouse_id = b.warehouse_id AND f.snapshot_date = b.first_date
JOIN analytics.vw_inventory_analysis l ON l.product_id = b.product_id AND l.warehouse_id = b.warehouse_id AND l.snapshot_date = b.last_date
JOIN master.products p ON p.product_id = b.product_id
JOIN master.warehouses w ON w.warehouse_id = b.warehouse_id
ORDER BY 5 DESC;
```

### 17. Average Stock Level by Category
**Business Purpose:** Compare typical stock holding across product categories.
```sql
SELECT category, ROUND(AVG(stock_on_hand),2) AS avg_stock
FROM analytics.vw_inventory_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 18. Which category has the highest reorder-risk rate?
**Business Purpose:** Identify which product categories are most prone to running low.
```sql
SELECT category,
    ROUND(100.0 * COUNT(*) FILTER (WHERE below_reorder_point_flag=TRUE)/ COUNT(*),2) AS below_reorder_pct
FROM analytics.vw_inventory_analysis
GROUP BY category
ORDER BY below_reorder_pct DESC;
```

### 19. Overall Stockout Rate
**Business Purpose:** Measure the overall frequency of complete stockouts.
```sql
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE stockout_flag=TRUE)/COUNT(*),2) AS overall_stockout_rate
FROM analytics.vw_inventory_analysis;
```

### 20. Total Distinct Product-Warehouse Combinations Tracked
**Business Purpose:** Measure the scope/coverage of inventory tracking.
```sql
SELECT COUNT(DISTINCT (product_id, warehouse_id)) AS total_combinations
FROM analytics.vw_inventory_analysis;
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

### 3. Over-Forecast Products
**Business Purpose:** Identify products where actual demand consistently falls short of the forecast.
```sql
SELECT product_name, COUNT(*) AS over_forecast_count
FROM analytics.vw_demand_analysis
WHERE forecast_status = 'Over Forecast'
GROUP BY 1
ORDER BY 2 DESC;
```

### 4. Warehouse Demand
**Business Purpose:** Compare total demand handled across warehouses.
```sql
SELECT warehouse_name, SUM(actual_demand) AS total_actual_demand, SUM(forecasted_demand) AS total_forecasted_demand
FROM analytics.vw_demand_analysis
GROUP BY 1
ORDER BY 2 DESC;
```

### 5. Monthly Demand Trend
**Business Purpose:** Track how total demand changes over time.
```sql
SELECT forecast_year, forecast_month_number, forecast_month_name,
    SUM(forecasted_demand) AS total_forecasted, SUM(actual_demand) AS total_actual
FROM analytics.vw_demand_analysis
GROUP BY 1, 2, 3
ORDER BY 1, 2 ASC;
```

### 6. Forecast Accuracy by Category
**Business Purpose:** Identify which product categories are hardest to forecast accurately.
```sql
SELECT category, ROUND(AVG(forecast_error_pct),2) AS avg_error_pct
FROM analytics.vw_demand_analysis
WHERE actual_demand IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC;
```

### 7. Best-Forecasted Products
**Business Purpose:** Identify products where the forecasting model performs best.
```sql
SELECT product_name, ROUND(AVG(forecast_error_pct),2) AS avg_error_pct
FROM analytics.vw_demand_analysis
WHERE actual_demand IS NOT NULL
GROUP BY 1
ORDER BY 2 ASC;
```

### 8. Worst-Forecasted Products
**Business Purpose:** Identify products where the forecasting model performs worst.
```sql
SELECT product_name, ROUND(AVG(forecast_error_pct),2) AS avg_error_pct
FROM analytics.vw_demand_analysis
WHERE actual_demand IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC;
```

### 9. Demand vs Supply Gap
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

### 10. Seasonal/Peak Demand Months
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
