# Northwind Sales Insights Project

This project uses the classic Northwind database to answer key business questions using SQL.

## Business Questions Answered

1. Rank employees by total sales
2. Calculate monthly running totals and MoM sales growth
3. Identify high-value customers
4. Analyse sales by product category
5. Find top-performing products per category
6. Identify top 20% customers by total purchase volume
7. Compare each employee's performance to the average

## Tools Used
- PostgreSQL
- SQL (CTEs, window functions, date functions)

## Schema
![Data Model](screenshots/data_model.png)  

## Sample Output
1. Employee Ranking  
![Employee Ranking](screenshots/ranking_employee_sales_performance.PNG)  
2. Running Total of Monthly Sales  
![Monthly Sales Running Total](screenshots/running_total_monthly_sales.PNG)  
3. Month on Month Sales  
![Month on Month Sales](screenshots/mom_sales_growth.PNG)  
4. Customers with Above Average Orders  
![Customers with Above Average Orders](screenshots/customers_above_avg_orders.PNG)
5. Sales Proportion by Category  
![Sales Proportion by Category](screenshots/sales_prop_by_category.PNG)  
6. Top Products per Category  
![Top Products per Category](screenshots/top_products_per_category.PNG)  
7. Top 20% Customers  
![Top 20% Customers](screenshots/top_20_pc_customers_sales_vol.PNG) 
8. Employee Sales Performance vs Average  
![Employee Sales Performance vs Average](screenshots/employee_sales_vs_average.PNG)
  


## Full SQL Logic

The complete set of SQL queries (including CTEs, window functions, and analytical logic) is available here:  
[View the SQL queries](sql_queries.md)