# Retail Sales Data Analysis Using SQL

In this project I explore retail sales data using SQL. The original data is a csv that I imported into pgAdmin4. 

I wrote SQL queries to do the following:
1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.


**CREATING A TABLE**
```sql
DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales 
	(transactions_id INT PRIMARY KEY,
	sale_date DATE,
	sale_time TIME,	
	customer_id	INT,
	gender VARCHAR(15),
	age	INT,
	category VARCHAR(20),	
	quantity INT,
	price_per_unit FLOAT,
	cogs FLOAT,
	total_sale FLOAT
)
```



--**DATA CLEANING - ADDRESSING NULL VALUES**
```sql
SELECT * FROM retail_sales
WHERE transactions_id IS NULL;

SELECT * FROM retail_sales
WHERE sale_date IS NULL;

SELECT * FROM retail_sales
	WHERE 
	transactions_id IS NULL
	OR
	sale_date IS NULL
	OR
	sale_time IS NULL
	OR
	gender IS NULL
	OR
	category IS NULL
	OR
	quantity IS NULL
	OR
	cogs IS NULL
	OR
	total_sale IS NULL;
```

--Delete the rows with null values

```sql
DELETE FROM retail_sales
	WHERE 
	transactions_id IS NULL
	OR
	sale_date IS NULL
	OR
	sale_time IS NULL
	OR
	gender IS NULL
	OR
	category IS NULL
	OR
	quantity IS NULL
	OR
	cogs IS NULL
	OR
	total_sale IS NULL;
```

-To check the number of null values left after deleting missing values, reuse the code we wrote to find null values.

```sql
SELECT * FROM retail_sales
	WHERE 
	transactions_id IS NULL
	OR
	sale_date IS NULL
	OR
	sale_time IS NULL
	OR
	gender IS NULL
	OR
	category IS NULL
	OR
	quantity IS NULL
	OR
	cogs IS NULL
	OR
	total_sale IS NULL;
```



**DATA EXPLORATION**

--How many sales do we have?

```sql
SELECT COUNT(*) as total_sale from retail_sales
```

--How many UNIQUE customers?

```sql
SELECT COUNT(DISTINCT customer_id) from retail_sales
```

--How many unique categories do we have?

```sql
SELECT COUNT(DISTINCT category) from retail_sales
```




**DATA ANALYSIS AND KEY BUSINESS PROBLEMS AND ANSWERS**

--1. Retrieve all columns for sales made on '2022-11-05'

```sql
SELECT * 
FROM retail_sales 
WHERE sale_date = '2022-11-05'
```

--2. Retrieve all transactions where the category is 'Clothing' and the quantity sold is more than or equal to 4 in Nov 2022

```sql
SELECT *
FROM retail_sales
WHERE 
	category = 'Clothing'
	AND
	TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
	AND
	quantity >= 4
```

--3. Calculate the total sales (total_sale) for each category.

```sql
SELECT 
	category,
	SUM(total_sale) as net_sale,
	COUNT(*) as total_orders
	FROM retail_sales
	GROUP BY category
```

--4. Find the average age of customers who perchased items from the 'beauty' category.

```sql
SELECT
	ROUND(AVG(age)) as average_age
	FROM retail_sales
	WHERE category = 'Beauty'
```

--5. Find all transactions where the total_sale is greater than 1000.

```sql
SELECT *
from retail_sales
WHERE total_sale > 1000
```

--6. Find the total number of transactions (transaction_id) made by each gender in each category.

```sql
SELECT 
category,
gender,
COUNT(*) as trans_id
from retail_sales 
GROUP BY gender, category
ORDER BY 1
```

--7. Calculate the average sale for each month, find out the best selling month in each year.

```sql
SELECT 
	y_year,
	m_month,
	avg_total_sale
FROM
(	
SELECT 
EXTRACT(YEAR FROM sale_date) as y_year,
EXTRACT(MONTH FROM sale_date) as m_month,
AVG(total_sale) as avg_total_sale,
RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1,2
) as T1
WHERE RANK = 1
```

--8. Find the top 5 customers based on the highest total sales.


```sql
SELECT 
	distinct customer_id,
	SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```



--9. Find the number of unique customers who purchased items from each category.

```sql
SELECT 
	distinct category,
	count(DISTINCT customer_id) as unique_customers
FROM retail_sales
GROUP BY 1
```	

--10. Create each shift and number of orders (Example Morning <=12, Afternoon Between 12 & 17, Evening > 17 )

```sql
WITH hourly_sale
AS
(
SELECT *, 
	CASE
		WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END as shift
FROM retail_sales
)
SELECT 
	shift,
	COUNT(*) AS total_orders
from hourly_sale
GROUP BY shift
```
 
--END OF PROJECT.
