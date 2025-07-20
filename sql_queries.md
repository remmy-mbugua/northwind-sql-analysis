# Northwind SQL Analysis Queries

This project demonstrates practical SQL querying using the classic Northwind database. Each query addresses a relevant business scenario — from employee sales performance and customer segmentation to product category analysis. The queries make use of Common Table Expressions (CTEs), window functions, aggregations, and conditional logic to deliver insights for hypothetical stakeholders.

---
## 1. Total Sales per Employee and Rank

Calculate total sales per employee and rank employees by sales descending
```sql
WITH sales_by_employee AS (
  SELECT 
    o.employee_id, 
    e.first_name || ' ' || e.last_name AS employee_name,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS sales_total
  FROM orders o
  JOIN employees e ON o.employee_id = e.employee_id
  JOIN order_details od ON o.order_id = od.order_id
  GROUP BY o.employee_id, employee_name
)

SELECT 
  employee_name, 
  sales_total, 
  RANK() OVER(ORDER BY sales_total DESC) AS employee_rank
FROM sales_by_employee;
...
```
## 2. Running Total of Monthly Sales

Calculate total monthly sales and running total
```sql
WITH sales_by_month AS (
  SELECT 
    DATE_TRUNC('month', o.order_date)::DATE AS month,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS sales_total
  FROM orders o
  JOIN order_details od ON o.order_id = od.order_id
  GROUP BY month
)

SELECT 
  month, 
  SUM(sales_total) OVER(ORDER BY month) AS running_total
FROM sales_by_month
ORDER BY month;
...
```
## 3. Month-over-Month Sales Growth

Calculate sales growth as % difference from previous month
```sql
WITH sales_per_month AS (
  SELECT 
    EXTRACT(year FROM order_date) AS year,
    EXTRACT(month FROM order_date) AS month,
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS sales_total
  FROM orders o
  JOIN order_details od ON o.order_id = od.order_id
  GROUP BY year, month
)

SELECT 
  year, 
  month,
  COALESCE(
    ROUND(
      (sales_total - LAG(sales_total) OVER(ORDER BY year, month)) / 
      LAG(sales_total) OVER(ORDER BY year, month) * 100, 2
    ), 
    0
  ) AS pc_change
FROM sales_per_month;
...
```
## 4. Customers by High Value Orders

Identify high-value orders and count how many orders each customer has
```sql
WITH order_values AS (
    SELECT 
        o.customer_id, 
        o.order_id, 
        SUM(od.unit_price * od.quantity * (1 - od.discount)) AS order_value
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.customer_id, o.order_id
),
avg_order_value AS (
    SELECT AVG(order_value) AS avg_val FROM order_values
),

ranked AS (
    SELECT 
        customer_id, 
        COUNT(*) FILTER (WHERE ov.order_value > a.avg_val) AS above_avg_order_cnt,
        DENSE_RANK() OVER(ORDER BY COUNT(*) FILTER (WHERE ov.order_value > a.avg_val) DESC) AS rnk
    FROM order_values ov
CROSS JOIN avg_order_value a
    GROUP BY customer_id
)
SELECT 
    customer_id, 
    above_avg_order_cnt
FROM ranked
WHERE rnk <= 10;
...
```
##  5. Sales Percentage by Category

Show each category's contribution to total sales as a percentage
```sql
WITH total_sales_per_productcat AS (
  SELECT 
    c.category_id, 
    c.category_name, 
    SUM(od.quantity * od.unit_price * (1 - od.discount)) AS total_sales
  FROM categories c
  JOIN products p ON c.category_id = p.category_id
  JOIN order_details od ON p.product_id = od.product_id
  GROUP BY c.category_id, c.category_name
)

SELECT 
  category_id, 
  category_name, 
  ROUND((total_sales::numeric / SUM(total_sales::numeric) OVER()) * 100, 2) AS sales_prop
FROM total_sales_per_productcat
ORDER BY sales_prop DESC;
...
```
##  6. Top Products per Category

Return top 3 products by sales within each category
```sql
WITH sales_per_product AS (
  SELECT 
    c.category_name, 
    p.product_name, 
    ROUND(SUM(od.quantity * p.unit_price * (1 - od.discount))::numeric,2) AS total_sales,
	ROW_NUMBER() OVER(PARTITION BY category_name ORDER BY SUM(od.quantity * p.unit_price * (1 - od.discount)) DESC) AS rnk
  FROM categories c
  JOIN products p ON c.category_id = p.category_id
  JOIN order_details od ON p.product_id = od.product_id
  GROUP BY c.category_name, p.product_name
)
SELECT 
  category_name, 
  product_name, 
  total_sales
FROM sales_per_product
WHERE rnk < 4
ORDER BY category_name, total_sales DESC

...
```

## 7. Top 20% Customers by Total Purchase Volume

Identify top 20% of customers by total purchase value
```sql
WITH ranking AS (
  SELECT 
    c.contact_name, 
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS total_purchase_vol,
    PERCENT_RANK() OVER(ORDER BY SUM(od.unit_price * od.quantity * (1 - od.discount)) DESC) AS rnk
  FROM customers c
  JOIN orders o ON c.customer_id = o.customer_id
  JOIN order_details od ON o.order_id = od.order_id
  GROUP BY c.contact_name
)

SELECT 
  contact_name AS customer, 
  total_purchase_vol
FROM ranking
WHERE rnk <= 0.2
ORDER BY total_purchase_vol DESC;

...
```

## 8. Employee Sales Performance vs Average

Compare employee total sales against the average across all employees
```sql
WITH performance_per_employee AS (
  SELECT 
    o.employee_id, 
    e.first_name || ' ' || e.last_name AS employee_name, 
    ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric, 2) AS sales_total
  FROM orders o
  JOIN employees e ON o.employee_id = e.employee_id
  JOIN order_details od ON o.order_id = od.order_id
  GROUP BY o.employee_id, employee_name
),

avg_performance AS (
  SELECT ROUND(AVG(sales_total), 2) AS avg_sales
  FROM performance_per_employee
)

SELECT 
  p.employee_name, 
  p.sales_total, 
  ROUND(p.sales_total - ap.avg_sales, 2) AS sales_vs_avg
FROM performance_per_employee p
CROSS JOIN avg_performance ap
ORDER BY sales_vs_avg DESC;