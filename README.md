## SQL Server - Exploratory Data Analysis /EDA/ Project
## Introduction

This project explores sales, customer, and product data using SQL Server.  
The goal is to understand business performance, customer demographics, and product distribution through structured queries and key metrics.  

**SQL techniques used:**  
- Querying (SELECT, DISTINCT)  
- Filtering (WHERE)  
- Aggregating (SUM, AVG, MIN, MAX)  
- Grouping (GROUP BY)  
- Join and set operations (JOINs, UNION ALL)

## Project Structure

| File | Description |
|------|-------------|
| `dim_customers.csv` | Customer dataset |
| `dim_products.csv` | Product dataset |
| `fact_sales.csv` | Sales transactions dataset |
| `SQL_First_Project.sql` | SQL queries for exploratory data analysis |

Dimension Analysis
===================================

This section focuses on exploring the **dimensions of the business data** (customers, products, and sales) providing a foundation for understanding the structure and key metrics.

### 1\. Customer Locations

The query lists all **distinct countries** where customers are located.

**Purpose:** To get a sense of the geographic reach of the business and identify where customers are coming from.
```sql
SELECT DISTINCT country

FROM gold.dim_customers;
```

### 2\. Main Products

This query retrieves all **distinct categories, subcategories, and product names**.

**Purpose:** To understand the product catalog, including which categories and subcategories exist, and to identify the main products offered.
```sql
SELECT DISTINCT category, subcategory, product_name

FROM gold.dim_products

ORDER BY 1,2,3;
```
### 3\. First and Last Order Dates

This analysis finds the **earliest and latest order dates**, as well as the **range in years** between them.

**Purpose:** To understand the time span of sales data and the history of business transactions.
```sql

SELECT MIN(order_date) FirstOrderDate,

MAX(order_date) LastOrderDate,

DATEDIFF(year, MIN(order_date), MAX(order_date)) OrderRangeY

FROM gold.fact_sales;
```
### 4\. Customer Age Analysis

This query calculates the **oldest, youngest, and average age of customers** based on their birthdates.

**Purpose:** To analyze the demographic composition of the customer base and assess the age distribution of customers.
```sql

SELECT DATEDIFF(year,MIN(birthdate), GETDATE()) Oldest_CustomerAge,

DATEDIFF(year,MAX(birthdate), GETDATE()) Youngest_CustomerAge,

AVG(DATEDIFF(year, birthdate, GETDATE())) AS Avg_CustomerAge

FROM gold.dim_customers;
```
### 5\. Business Key Metrics

This set of queries provides **high-level metrics** about sales, orders, products, and customers:

*   **Total Sales:** Sum of all sales amounts.
    
*   **Items Sold:** Total number of items sold.
    
*   **Average Price:** Average price of sold items.
    
*   **Number of Orders:** Count of unique orders.
    
*   **Number of Products:** Total number of products in the catalog.
    
*   **Number of Customers:** Total number of customers in the database.
    

**Purpose:** To give a quick overview of the overall business performance, enabling stakeholders to understand key figures at a glance.
```sql

SELECT 'Total Sales' AS Measure_Name,

SUM(sales_amount) as Measure_Value

FROM gold.fact_sales

UNION ALL

SELECT 'Items Sold' AS Measure_Name,

SUM(quantity) AS Items_Quantity_Sold

FROM gold.fact_sales

UNION ALL

SELECT 'Average Price' AS Measure_Name,

AVG(price) AS Avg_Price

FROM gold.fact_sales

UNION ALL

SELECT 'Number of Orders' AS Measure_Name,

COUNT(DISTINCT Order_number) AS Num_Orders

FROM gold.fact_sales

UNION ALL

SELECT 'Number of Products' AS Measure_Name,

COUNT(product_key) AS Num_Products

FROM gold.dim_products

UNION ALL

SELECT 'Number of Customers' AS measure_name,

COUNT(customer_key) AS Num_Customers

FROM gold.dim_customers;
```
**Magnitude Analysis**
----------------------

This section of the project focuses on understanding the **scale and distribution** of customers, products, and sales in the business. Each analysis provides insights into key business metrics.

### 1\. Total Number of Customers by Country

This analysis counts how many customers exist in each country.

**Purpose:** To understand the geographic distribution of the customer base and identify which regions have the largest number of customers.
```sql

SELECT country, COUNT(customer_id) AS number_of_customers

FROM gold.dim_customers

GROUP BY country

ORDER BY count(customer_id) DESC;
```
### 2\. Total Customers by Gender

This query counts unique customers for each gender.

**Purpose:** To analyze the demographic composition of the customer base and evaluate gender distribution.
```sql

SELECT gender, COUNT(DISTINCT customer_id) AS customers

FROM gold.dim_customers

GROUP BY gender

ORDER BY customers DESC;
```
### 3\. Total Products by Category

This analysis counts the number of products in each product category.

**Purpose:** To understand the diversity of products offered in different categories and identify which categories have the most items.
```sql

SELECT category, COUNT(product_id) Num_of_Products

FROM gold.dim_products

GROUP BY category

ORDER BY Num_of_Products DESC;
```
### 4\. Average Cost for Each Category

This calculation finds the average cost of products within each category.

**Purpose:** To assess pricing patterns and compare the average cost across different product categories.
```sql

SELECT category, AVG(cost) as Avg_Cost

FROM gold.dim_products

GROUP BY category

ORDER BY Avg_Cost DESC;
```
### 5\. Total Revenue Generated by Each Category

This analysis sums all sales amounts per product category by joining sales and product tables.

**Purpose:** To identify the most profitable categories and understand the revenue contribution of each category.
```sql

SELECT p.category,

SUM(s.sales_amount) Total_Revenue

FROM gold.fact_sales AS s

LEFT JOIN gold.dim_products AS p

ON p.product_key=s.product_key

GROUP BY p.category

ORDER BY Total_Revenue DESC;
```
### 6\. Total Revenue Generated by Each Customer

This query calculates the total revenue each customer has generated by joining sales and customer data.

**Purpose:** To identify high-value customers and prioritize them for loyalty programs or targeted marketing efforts.
```sql

SELECT c.customer_id, c.first_name, c.last_name,

SUM(s.sales_amount) AS Revenue_Per_Customer

FROM gold.fact_sales AS s

LEFT JOIN gold.dim_customers AS c

ON c.customer_key=s.customer_key

GROUP BY c.customer_id, c.first_name, c.last_name

ORDER BY Revenue_Per_Customer DESC;
```
### 7\. Distribution of Sold Items Across Countries

This analysis sums the total quantity of items sold per country.

**Purpose:** To understand market penetration, identify countries with the highest sales volumes, and guide geographic expansion strategies.
```sql

SELECT c.country, SUM(s.quantity) AS total_sold_items

FROM gold.dim_customers c

LEFT JOIN gold.fact_sales s

ON c.customer_key=s.customer_key

GROUP BY c.country

ORDER BY total_sold_items DESC;
```
Ranking Analysis
================

This section focuses on **ranking products based on revenue**, helping to identify which products drive the most value and which contribute the least.

### 1\. Top 5 Products by Revenue Generated

This query calculates the **total revenue for each product** and selects the top 5 products with the **highest revenue**.

**Purpose:** To identify the best-performing products, enabling targeted marketing, inventory planning, and strategic promotions.
```sql

SELECT TOP 5

p.product_name,

SUM(s.sales_amount) Total_Revenue

FROM gold.fact_sales AS s

LEFT JOIN gold.dim_products AS p

ON p.product_key=s.product_key

GROUP BY product_name

ORDER BY Total_Revenue DESC;
```
### 2\. Top 5 Products by Lowest Revenue Generated

This query calculates the **total revenue for each product** and selects the top 5 products with the **lowest revenue**.

**Purpose:** To identify underperforming products, helping the business decide whether to improve, promote, or discontinue them.

```sql

SELECT TOP 5

p.product_name,

SUM(s.sales_amount) Total_Revenue

FROM gold.fact_sales AS s

LEFT JOIN gold.dim_products AS p

ON p.product_key=s.product_key

GROUP BY product_name

ORDER BY Total_Revenue ASC;
```
## Conclusion

This project demonstrates how SQL Server can be used to analyze business data effectively.  
Through exploratory data analysis, key business metrics, and dimension analysis, we gained insights into customer demographics, product hierarchy, and sales performance.  The queries and techniques showcased here provide a strong foundation for more advanced analytics, reporting, and decision-making.  
Huge thanks to my teacher @DataWithBaraa for the wonderful SQL BootCamp. This project is part of his SQL Ultimate Course (30h).
[DataWithBaraa GitHub](https://github.com/DataWithBaraa)
