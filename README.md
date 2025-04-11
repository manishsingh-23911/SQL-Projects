# SQL project in Retail Sales Analysis 

## Project Overview

**Project Title**: Retail Sales Analysis   
**Database**: `sales_analysis_project`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. 

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `sales_analysis_project`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE sales_analysis_project;

DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales
			(
				transactions_id	INT PRIMARY KEY,
				sale_date DATE,
				sale_time TIME,
				customer_id INT,
				gender VARCHAR(15),
				age INT,
				category VARCHAR (15),
				quantity INT,
				price_per_unit FLOAT,
				cogs FLOAT,
				total_sale FLOAT
			);
```
- **preview data and check the number of rows**
```
SELECT * FROM retail_sales
LIMIT 10;

SELECT COUNT (*) FROM retail_sales;
```

### 2. Data Cleaning

- Checking for null values
```
SELECT * FROM retail_sales
WHERE 
	transactions_id IS NULL
	OR
	sale_date IS NULL
	OR
	sale_time IS NULL
	OR
	customer_id IS NULL 
	OR
	gender IS NULL 
	OR
	age IS NULL
	OR
	category IS NULL
	OR
	quantity IS NULL
	OR
	price_per_unit IS NULL
	OR
	cogs IS NULL
	OR 
	total_sale IS NULL;
```
- Deleting rows which have null values (ignoring null values for age)
     Check results before committing
     ROLLBACK;  -- to undo
     COMMIT;    -- to confirm
```
BEGIN;

DELETE FROM retail_sales
WHERE 
	transactions_id IN (679,746,1225)

COMMIT;
```
    

### 3. Data Exploration
- How many sales do we have?
```
SELECT COUNT(*) FROM retail_sales;
```
 -or:
```
SELECT COUNT(transactions_id) FROM retail_sales;
```

- How many customers do we have?
```
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
```

- what product categories do we have?
```
SELECT DISTINCT category FROM retail_sales;
```


### 4. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **retrieve sales data for all sales made on '2022-11-05'**:
```sql
SELECT * 
FROM retail_sales 
WHERE sale_date = '2022-11-05';

```

2. **retrieve all transactions where the category is clothing**:
```sql
SELECT * 
FROM retail_sales 
WHERE category = 'Clothing';
```

3. **Find out the total sales in each category**:
```sql
SELECT SUM(total_sale), category
FROM retail_sales 
GROUP BY category;

```

4. **retrieve all transaction where category is 'Clothing' and the quantity sold is more than or equal to 4 in the month of November 2022**:
```sql
SELECT * FROM retail_sales
WHERE category = 'Clothing'
	AND quantity >= 4
	AND sale_date BETWEEN '2022-10-31' AND '2022-12-01'; 
```

5. **find out the average age of customers who have transactions in 'Beauty' category**:
```sql
SELECT ROUND(AVG(age),2) as average_age, category
FROM retail_sales
WHERE category = 'Beauty'
GROUP BY category; 
```

6. **retrieve all transactions where total_sale is greater than 1000**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000;
```

7. **find out total number of transactions made by each gender in each category**:
```sql
SELECT gender, category, COUNT(transactions_id) as total_transactions
FROM retail_sales 
GROUP BY category, gender
ORDER BY 2;
```

8. **find out the average sales in each month. and also the highest selling month of each year**:
  - Average sale in each month -
```sql
SELECT 
	EXTRACT(YEAR FROM sale_date) as year,
	EXTRACT(MONTH FROM sale_date) as month,
	ROUND(AVG(total_sale)::numeric ,2) as avg_sale
FROM retail_sales
GROUP BY 1, 2
ORDER BY 1, 2;
```
  - Highest selling month in each year
```sql
SELECT 
	year,
	month,
	avg_sale
FROM
(
	SELECT 
		EXTRACT(YEAR FROM sale_date) as year,
		EXTRACT(MONTH FROM sale_date) as month,
		ROUND(AVG(total_sale)::numeric ,2) as avg_sale,
		RANK() OVER(PARTITION BY EXTRACT (YEAR FROM sale_date) ORDER BY AVG(total_sale)DESC) as rank
	FROM retail_sales
	GROUP BY 1, 2
) as t1
WHERE rank = 1;
```




9. **Find the top 5 customers based on highest total sales.**:
```sql
SELECT 
	customer_id,
	SUM(total_sale) as sales_per_customer
FROM retail_sales
GROUP BY customer_id
ORDER BY 2 DESC
LIMIT 5;
```

10. **Fing how many unique customers have made purchased from all 3 categories of products**:
```sql
SELECT COUNT (*)
FROM
(
SELECT customer_id as unique_customers
FROM retail_sales
WHERE category IN ('Clothing', 'Beauty', 'Electronics')
GROUP BY customer_id
HAVING COUNT(DISTINCT category) = 3
) as SUB;
```

11. **Find the number of customers who made purchases in each category**
```sql
SELECT 
	category,
	COUNT(DISTINCT customer_id) as no_of_customers
FROM retail_sales
GROUP BY category
ORDER BY no_of_customers
```

12. **Create 3 shifts(before 12pm, b/w 12pm to 5 pm, after 5 pm) with number of orders per shift**
```sql
WITH hourly_sale
AS
(
SELECT *,
	CASE
		WHEN EXTRACT (HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT (HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END as shift
FROM retail_sales
)

SELECT 
	shift,
	COUNT(*) as total_orders
FROM hourly_sale
GROUP BY shift
```



## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.


