# Coffee-shop-Sales-Performance-Analysis
Sales trend analysis and inventory analysis project of a real coffee business dataset

## Table of Content:
- [PROJECT OVERVIEW](##PROJECT-OVERVIEW)
- [RESULTS/FINDINGS](#RESULTS/FINDINGS)
- [RECOMMENDATIONS](#RECOMMENDATIONS)
- [ANALYSIS CODE](#ANALYSIS-CODE)

## PROJECT OVERVIEW

  TDGSDF G

## DATA SOURCE


## TOOLS
- Github: Data Downloading 
- SQL Server: Data Analysis
- PowerBI: Data Visulization


## DATA SOURCING

## Exploring Data Analysis
Data Analysis and key questions have been answered belowed:

#### 1. Coffee Consumers Count
  - How many people in each city are estimated to consume coffee, given that 25% of the population does?

#### 2. Total Revenue from Coffee Sales
  - What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
  
#### 3. Sales Count for Each Product
  - How many units of each coffee product have been sold?
  
#### 4. Average Sales Amount per City
  - What is the average sales amount per customer in each city?
  
#### 5. City Population and Coffee Consumers
  - Provide a list of cities along with their populations and estimated coffee consumers.
  
#### 6. Top Selling Products by City
  - What are the top 3 selling products in each city based on sales volume?
  
#### 7. Customer Segmentation by City
  - How many unique customers are there in each city who have purchased coffee products?
  
#### 9. Average Sale vs Rent
  - Find each city and their average sale per customer and avg rent per customer
  
#### 10. Monthly Sales Growth
  - Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).
  
#### 11. Market Potential Analysis
  - Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer

## RESULTS/FINDINGS

## RECOMMENDATIONS


## ANALYSIS CODE
```sql
-- REPORTS AND DATA ANALYSIS 
-- 1.Coffee Consumers Count
-- How many people in each city are estimated to consume coffee, given that 25% of the population does?
SELECT 
		city_name,		
		ROUND((population *0.25)/1000000,2) AS Coffee_customer_by_millions,
		city_rank
FROM city 
ORDER BY 2 DESC;

-- 2.Total Revenue from Coffee Sales
--What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
SELECT 
	ci.city_name,
	SUM(s.total) AS Total_Revenue
FROM sales AS s
JOIN customers AS c
ON s.customer_id = c.customer_id
JOIN city AS ci
ON c.city_id = ci.city_id
WHERE
	EXTRACT(YEAR FROM s.sale_date) =2024
AND
	EXTRACT(QUARTER FROM s.sale_date) = 1
GROUP BY city_name
ORDER BY Total_Revenue DESC;

--3.Sales Count for Each Product
--How many units of each coffee product have been sold?
SELECT 
	p.product_name,
	COUNT(sale_id) AS total_orders
FROM products AS p
LEFT JOIN 
sales as s
ON p.product_id = s.product_id
GROUP BY 1
ORDER BY 2 DESC;

--4.Average Sales Amount per City
--What is the average sales amount per customer in each city?
SELECT 
	ci.city_name,
	COUNT(DISTINCT c.customer_id) AS total_customer,
	SUM(s.total) AS total_sales,
	ROUND(SUM(s.total)::numeric /COUNT(DISTINCT c.customer_id)::numeric,2) AS avg_sales_per_customer
FROM sales AS s
JOIN customers AS c
ON s.customer_id = c.customer_id
JOIN city AS ci
ON c.city_id = ci.city_id
GROUP BY 1
ORDER BY 4 DESC;

--5. City Population and Coffee Consumers(25%)
--Provide a list of cities along with their populations and estimated coffee consumers.
--RETURN city names, population, total current customers, and estimated customers(25%)

WITH city_table AS 
	(SELECT 
		city_name AS city,
		ROUND((population *0.25)/1000000,2)AS estimated_coffee_consumers_bymillions
	FROM city
	),

city_customer AS
	(SELECT
		ci.city_name,
		COUNT(DISTINCT c.customer_id) AS current_customers
	FROM sales AS s
	JOIN customers AS c
	ON s.customer_id = c.customer_id
	JOIN city AS ci
	ON c.city_id = ci.city_id
	GROUP BY ci.city_name
	)
	
SELECT 
	city_table.city,
	city_table.estimated_coffee_consumers_bymillions,
	city_customer.current_customers
FROM city_table
JOIN city_customer
ON city_table.city = city_customer.city_name;


-- 6.Top Selling Products by City
-- What are the top 3 selling products in each city based on sales volume?
-- solution : city_name, product_name, and saleID
SELECT *
FROM 
(SELECT 
	c.city_name,
	p.product_name,
	COUNT(s.sale_id) AS total_orders,
	DENSE_RANK() OVER (PARTITION BY c.city_name ORDER BY COUNT(s.sale_id) DESC) as rank
FROM sales AS s
JOIN products AS p
ON 	s.product_id = p.product_id
JOIN customers AS cs
ON s.customer_id = cs.customer_id
JOIN city AS c
ON cs.city_id = c.city_id
GROUP BY 1,2
) AS t1
WHERE rank <=3;

--7. Customer Segmentation by City
--How many unique customers are there in each city who have purchased coffee products?
SELECT 
	ci.city_name AS city,
	COUNT(DISTINCT c.customer_id) AS customer
FROM products as p
JOIN sales as s
ON p.product_id = s.product_id
JOIN customers as c
ON s.customer_id = c.customer_id
JOIN city as ci
ON c.city_id = ci.city_id
WHERE
	s.product_id IN (1,2,3,4,5,6,7,8,9,10,11,12,13,14)
GROUP BY 1
ORDER BY 2 DESC;

--8.Average Sale vs Rent
--Find each city and their average sale per customer and avg rent per customer
WITH avg_sale AS
	(SELECT 
		DISTINCT ci.city_name as city,
		SUM(s.total) as total_sales,
		COUNT(DISTINCT c.customer_id) as total_customer,
		ROUND(SUM(DISTINCT s.total::numeric)/COUNT(DISTINCT c.customer_id::numeric),2) as avg_sale_customer
	FROM sales as s
	JOIN customers as c
	ON s.customer_id = c.customer_id
	JOIN city as ci
	ON c.city_id = ci.city_id
	GROUP BY 1),

avg_rent AS 
	(SELECT
		city_name,
		estimated_rent
	FROM city
	)

SELECT
	city,
	avg_sale_customer,
	estimated_rent,
	total_sales,
	ROUND(estimated_rent::numeric/total_customer::numeric,2) as avg_estimated_rent_per_customer
FROM avg_sale 
INNER JOIN avg_rent
ON avg_sale.city = avg_rent.city_name
ORDER BY 5 DESC;



--9.Monthly Sales Growth
--Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).
--BY EACH CITY
--city,month,total sales

WITH
monthly_sales
AS
	(SELECT
		ci.city_name,
		EXTRACT(month FROM s.sale_date) as month,
		EXTRACT(year FROM s.sale_date) as year,
		SUM(s.total) AS total_sale
	FROM sales as s
	JOIN customers as c
	ON s.customer_id = c.customer_id
	JOIN city as ci
	ON c.city_id = ci.city_id
	GROUP BY 1,2,3
	ORDER BY 1,3,2 ASC),

last_monthly_sale AS
	(SELECT
		city_name,
		month,
		year,
		total_sale AS current_month_sale,
		LAG(total_sale,1) OVER (PARTITION BY city_name ORDER BY year,month) AS last_month_sale
	FROM monthly_sales
	)

SELECT 
	city_name,
	month,
	year,
	current_month_sale,
	last_month_sale,
	ROUND(
		(current_month_sale-last_month_sale)::numeric/last_month_sale::numeric*100,2
		) as pert_sales_growth
FROM
	last_monthly_sale
WHERE last_month_sale IS NOT NULL;



--10.Market Potential Analysis
--Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer(25%)
WITH avg_sale AS
	(SELECT 
		DISTINCT ci.city_name as city,
		SUM(s.total) as total_sales,
		COUNT(DISTINCT c.customer_id) as total_customer,
		ROUND(SUM(DISTINCT s.total::numeric)/COUNT(DISTINCT c.customer_id::numeric),2) as avg_sale_customer
	FROM sales as s
	JOIN customers as c
	ON s.customer_id = c.customer_id
	JOIN city as ci
	ON c.city_id = ci.city_id
	GROUP BY 1),

avg_rent AS 
	(SELECT
		city_name,
		estimated_rent,
		ROUND(population*0.25/1000000,2) AS estimated_coffee_consumer
	FROM city
	)

SELECT
	city,
	total_sales,
	total_customer,
	estimated_rent AS total_rent,
	estimated_coffee_consumer
FROM avg_sale 
INNER JOIN avg_rent
ON avg_sale.city = avg_rent.city_name
ORDER BY total_sales DESC
LIMIT 3;



