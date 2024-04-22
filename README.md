# Bike Strore Analysis: SQL
## Table of Content
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Exploring data](#exploring-data)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Recommendations](#recommendations)
### Project Overview
This project analyzes sales data from a bike store database, using SQL queries to identify top-selling products, most profitable stores, and sales trends. The analysis provides insights into customer behavior and facilitates more effective inventory management and sales strategies.
### Data Sources
Dataset : https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database
### Tools
- Excel - Data Cleaning
- SQL Server - Data Analysis

### Data Cleaning/Preparation
To prepare the data for analysis, the following tasks were performed:
1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting.
### Exploring data
Exploring the dataset in order to answer the following questions:
1.	Which store has the biggest sale?
2.	What year had the biggest total revenue?
3.	Which month had the biggest total revenue?
4.	What are the top 10 sold products?
5.	What are the top categories?
6.	What is the customer status (whether they are repeat customers or one-time customers)?
7.	What is the average sale per year?
### Data Analysis
```sql
-- which store has the biggest sale

WITH total_revenue AS (
    SELECT s.store_name AS store_name,
           o.store_id AS store_id,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
    FROM bike.dbo.stores$ AS s
    INNER JOIN bike.dbo.orders$ AS o ON s.store_id = o.store_id
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
    GROUP BY s.store_name, o.store_id
), total_sales_all_stores AS (
    SELECT SUM(total_sales) AS total_sales_all
    FROM total_revenue
)
SELECT store_name,
       store_id,
       total_sales,
       ROUND((total_sales / total_sales_all) * 100, 2) AS percentage_of_total_sales
FROM total_revenue, total_sales_all_stores
ORDER BY total_sales DESC;

--year with the biggest total revenue

WITH total_sales AS (
    SELECT s.store_name AS store_name,
           YEAR(o.order_date) AS order_year,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
    FROM bike.dbo.stores$ AS s
    INNER JOIN bike.dbo.orders$ AS o ON s.store_id = o.store_id
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
    GROUP BY s.store_name, YEAR(o.order_date) 
)
SELECT TOP 1 order_year,
       SUM(total_sales) AS total_sales_all
FROM total_sales
GROUP BY order_year
ORDER BY total_sales_all DESC; 

WITH total_sales AS (
    SELECT s.store_name AS store_name,
           YEAR(o.order_date) AS order_year,
           MONTH(o.order_date) AS order_month,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
    FROM bike.dbo.stores$ AS s
    INNER JOIN bike.dbo.orders$ AS o ON s.store_id = o.store_id
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
    GROUP BY s.store_name, YEAR(o.order_date), MONTH(o.order_date)
)
SELECT TOP 1 order_month,
       order_year,
       SUM(total_sales) AS total_sales_all
FROM total_sales
GROUP BY order_year, order_month
ORDER BY total_sales_all DESC;

--top 10 solds products

WITH total_sales AS (
    SELECT p.product_id,
           p.product_name,
           SUM(oi.quantity) AS total_quantity,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales_all
    FROM bike.dbo.order_items$ AS oi
    INNER JOIN bike.dbo.products$ AS p ON p.product_id = oi.product_id
    GROUP BY p.product_id, p.product_name
)
SELECT TOP 10
       product_id,
       product_name,
       total_quantity, 
       total_sales_all 
FROM total_sales
ORDER BY total_quantity DESC; 

-- top 10 customers
WITH total_sales AS (
    SELECT c.customer_id,
           c.first_name,
           c.last_name,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales_all
    FROM bike.dbo.orders$ AS o
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
    INNER JOIN bike.dbo.customers$ AS c ON o.customer_id = c.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT TOP 10
       customer_id,
       first_name,
       last_name, 
       total_sales_all 
FROM total_sales
ORDER BY total_sales_all DESC;


--top categories

WITH total_sales AS (
    SELECT c.category_name,
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales_all
    FROM bike.dbo.categories$ AS c
    INNER JOIN bike.dbo.products$ AS p ON c.category_id = p.category_id
    INNER JOIN bike.dbo.order_items$ AS oi ON p.product_id = oi.product_id
    GROUP BY c.category_name
)
SELECT category_name, SUM(total_sales_all) AS total_sales
FROM total_sales
GROUP BY category_name
ORDER BY total_sales DESC;

--customer status

WITH customer_stats AS (
    SELECT
        o.customer_id,
		c.first_name,
		c.last_name,
        SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_spent,
        COUNT(DISTINCT o.order_id) AS total_orders
    FROM bike.dbo.orders$ AS o
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
	INNER JOIN bike.dbo.customers$ AS c ON o.customer_id = c.customer_id
    GROUP BY o.customer_id, c.first_name, c.last_name
)

SELECT  
    customer_id,
	first_name,
	last_name,
    CASE WHEN total_orders > 1 THEN 'repeat buyer'
         ELSE 'one-time buyer'
    END AS purchase_frequency
FROM customer_stats;

--average sale per year
WITH yearly_sales AS (
    SELECT
        YEAR(order_date) AS order_year,
        SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS yearly_total_sales
    FROM bike.dbo.orders$ AS o
    INNER JOIN bike.dbo.order_items$ AS oi ON o.order_id = oi.order_id
    GROUP BY YEAR(order_date)
)

SELECT  
    order_year,
    yearly_total_sales,
    AVG(yearly_total_sales) OVER() AS average_sales_per_year
FROM yearly_sales
ORDER BY order_year;
```

### Results

- The store with the highest sales is Baldwin Bike, which accounts for nearly 67% of the total sales.
- The year with the highest sales is 2017, with a total sales sum of $3,379,800.96.
- The month with the highest sales is April 2018, with a total sales sum of $782,882.22.
- The most frequently purchased product is Surly Ice Cream Truck Frameset - 2016.
- The customer who spent the most spent $34,390.37.
- The most frequently purchased category is mountain bikes, followed by road bikes.
- Average sales per year:
  2016: $2418808.912533,
  2017: $2418808.912533,
  2018: $2418808.912533.

### Recommendations
1. Enhance marketing efforts for stores other than Baldwin Bike, emphasizing the unique features and benefits of their products.
2. Develop targeted promotions and advertising campaigns for mountain bikes to capitalize on their popularity.
3. Implement customer loyalty programs to incentivize repeat purchases and foster long-term relationships with customers.
4. Collaborate with manufacturers to ensure a diverse and high-quality product range that meets the needs and desires of customers.






