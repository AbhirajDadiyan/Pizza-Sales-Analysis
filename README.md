# Pizza-Sales-Analysis
Pizza Sales Analysis Project

Introduction 
This project conducts a detailed analysis of pizza sales of a pizza stores to gain insights into the business performance. 
Table of Contents
1.	Data description
2.	Installation and Usage
3.	Situation
4.	Analysis Overview
5.	SQL code
6.	Results and Discussion
7.	Conclusions
Data Description
The dataset includes one table: ‘pizza sales’ that contains details about the pizza category, pizza size, cost, ingredients, quantity sold, order day and timestamps.
Installation and Usage
To run this project ensure you have PostgreSQL or any other tool capable of running SQL. You will need to create a new database in your DB and use the code provided in SQL code to create the tables then import the data from the CSV file. You will also need Microsoft Excel (anything after 2016 version should be good) to create visualizations. The CSV and excel file can be found here *insert link to google drive for csv and excel file*
Situation
The business requires us to create the following KPI’s to gain performance insights. We need to create the following metrics:
1.	Total Revenue
2.	Average Order Value
3.	Total Pizzas Sold
4.	Total Orders
5.	Average Pizzas Per Order
To understand various aspects of the pizza sales data to gain insights and understand key trends, I am going to create the following visualizations in Excel.
1.	Daily Trend for Total Order
2.	Hourly Trend for Total Orders
3.	Percentage of Sales by Pizza Category
4.	Percentage of Pizza Sales by Pizza Siza
5.	Total Pizzas sold by Pizza Category
6.	Top 5 Best Sellers by Total Pizzas 
7.	Top 5 Worst Sellers by Total Pizzas Sold
SQL Code
/*
 Date Created: December 2nd, 2023
Created by: Abhi S Dadiyan
Description: Creating a table named 'pizza_sales' with multiple columns. 
This table is designed to store data from a CSV file, with appropriate data types 
for each column including integer, double precision, and varying character types.
*/
```
CREATE TABLE 
	pizza_sales (
    -- 'pizza_id' column, type INT (integer)
    pizza_id INT,
    
    -- 'order_id' column, type INT (integer)
    order_id INT,
    
    -- 'pizza_name_id' column, type VARCHAR (variable character) with a maximum length of 250 characters
    pizza_name_id VARCHAR(250),
    
    -- 'quantity' column, type INT (integer)
    quantity INT,
    
    -- 'order_date' column, type DATE to store calendar dates
    order_date DATE,
    
    -- 'order_time' column, type TIME to store time of day
    order_time TIME,
    
    -- 'unit_price' column, type DOUBLE PRECISION for storing floating-point numbers
    unit_price DOUBLE PRECISION,
    
    -- 'total_price' column, type DOUBLE PRECISION for storing floating-point numbers
    total_price DOUBLE PRECISION,
    
    -- 'pizza_size' column, type VARCHAR with a maximum length of 250 characters
    pizza_size VARCHAR(250),
    
    -- 'pizza_category' column, type VARCHAR with a maximum length of 250 characters
    pizza_category VARCHAR(250),
    
    -- 'pizza_ingredients' column, type VARCHAR with a maximum length of 250 characters
    pizza_ingredients VARCHAR(250),
    
    -- 'pizza_name' column, type VARCHAR with a maximum length of 250 characters
    pizza_name VARCHAR(250)
);
```
--Using SUM to find total revenue, Using ROUND to limit the decimals to 2, using
--CAST to change the sum type into numeric as ROUND cannot take a SUM type of double precision
```
SELECT
    ROUND(CAST(SUM(total_price) AS NUMERIC), 2) AS Total_Revenue
FROM
    pizza_sales;
```
 
--Creating a query to find average order value, Using DISTINCT COUNT of order id
--to  make sure that the average isnt skewed, using ROUND and CAST for the same reason as previous query
```
SELECT
	ROUND(CAST(SUM(Total_price)/COUNT(DISTINCT order_id )AS NUMERIC),2)AS Avg_order_value
FROM
	pizza_sales;
```
 
--Creating a Query to find total pizzas sold


```
SELECT
	SUM(quantity) AS Total_Pizzas_Sold
FROM
	pizza_sales;
```
 



--Creating a query to find total number of distinct orders
```
SELECT
	COUNT(DISTINCT(order_id)) AS Total_no_of_orders
FROM
	pizza_sales;
```
 
--Creating a query to find average pizzas per order
```
SELECT
	SUM(quantity)/COUNT(DISTINCT(order_id)) AS Avg_pizza_per_order
FROM
	pizza_sales;
```
 
--Creating a query to find average pizzas per order
--Using CAST at both numirator and denominator to ensure
--that the division is carried out in decimal arithmetic

```
SELECT
	ROUND((CAST(SUM(quantity) AS DECIMAL(10,2))/
	CAST(COUNT(DISTINCT(order_id)) AS DECIMAL(10,2))),2) AS Avg_pizza_per_order
FROM
	pizza_sales;
```
 
 
--creating a query to find daily trend for total orders
--using TO_CHAR to convert date values into the day of the week
--You might need to use DATEPART, EXTRACT or Strftime if you are using 
--MS SQL Server, MySQL or SQLite respectively
```
SELECT
    TO_CHAR(order_date, 'Day') as order_Day,
    COUNT(DISTINCT order_id) as total_orders
FROM
    pizza_Sales
GROUP BY
    TO_CHAR(order_date, 'Day');
```
 
--Creating a query to find the hourly trend for total orders
--Using EXTRACT here because we are interested in a numeric value of the time/day
```
SELECT
    EXTRACT(HOUR FROM order_time) AS order_hours,
    COUNT(DISTINCT order_id) AS total_orders
FROM
    pizza_sales
GROUP BY
    EXTRACT(HOUR FROM order_time)
ORDER BY
    EXTRACT(HOUR FROM order_time);
```
 
--creating a query to find percentage of sales by pizza category
--using subquery to compute the total sales of all categories
--this subquery runs once for the entire query, 
--providing a single total sales figure used in the percentage
--using cast to convert the data into numeric in order to round to 2 decimal places
```
SELECT
	 p.pizza_category
	,ROUND(CAST(SUM(total_price) AS NUMERIC),2) AS Total_Sales
	,ROUND(CAST(SUM(p.total_price)*100/
	 (SELECT 
	 	SUM(total_price)
	 FROM
	 	pizza_sales) AS NUMERIC),2) AS total_sales_percentage
FROM
	pizza_sales p
GROUP BY
	p.pizza_category;
```
 

--creating a query to filter total_sales_percentage by month
--change month to quarters to find % of different quarters
```
SELECT
	 p.pizza_category
	,ROUND(CAST(SUM(total_price) AS NUMERIC), 2) AS Total_Sales
	,ROUND(CAST(SUM(p.total_price) * 100 /
	(SELECT
	 	SUM(total_price) 
	 FROM 
	 	pizza_sales 
	 WHERE 
	 	EXTRACT(MONTH FROM order_date) = 1) AS NUMERIC), 2) AS total_sales_percentage
FROM
	pizza_sales p
WHERE 
	EXTRACT(MONTH FROM order_date) = 1 -- Filtering for January (month 1)
GROUP BY
	p.pizza_category;
```
 

--creating a query to find percentage of sales by Pizza Size
--using subquery for the same reason as the last query
```
SELECT
	 p.pizza_size
	,ROUND(CAST(SUM(total_price) AS NUMERIC),2) AS Total_Sales
	,ROUND(CAST(SUM(p.total_price)*100/
	(SELECT 
		SUM(total_price)
	FROM
		pizza_sales) AS NUMERIC),2) AS total_sales_percentage_by_size
FROM
	pizza_sales p
GROUP BY
	p.pizza_size
ORDER BY
	total_sales_percentage_by_size DESC;
``` 
--query that can filter total_percentage_by_size with quarters
--if you want to filter the query by month just change quarter to month and write the number of month you want to know the % sales of

```
SELECT
	 p.pizza_size
	,ROUND(CAST(SUM(total_price) AS NUMERIC), 2) AS Total_Sales
	,ROUND(CAST(SUM(p.total_price) * 100 / 
	(SELECT 
		 SUM(total_price)
	 FROM 
	 	pizza_sales
	 WHERE 
	 	EXTRACT(quarter FROM order_date) = 1) AS NUMERIC), 2) AS Total_sales_percentage_by_size
FROM
	pizza_sales p
WHERE EXTRACT(quarter FROM order_date) = 1 --change the number to see different quarters
GROUP BY
	p.pizza_size
ORDER BY
	Total_sales_percentage_by_size DESC;
```
 
--creating a query to find total quantity of pizza sold by category
```
SELECT
	 pizza_category
	,SUM(quantity) AS Total_Pizzas_Sold
FROM
	pizza_sales
GROUP BY
	pizza_category;
```
 
--creating a query to see TOP 5 best sellers by Pizza_name
```
SELECT
	 pizza_name
	,SUM(quantity) AS Total_Pizzas_Sold
FROM
	pizza_sales
GROUP BY
	pizza_name
ORDER BY
	Total_Pizzas_Sold DESC
LIMIT 5;
```
 
--Creating a query to see Bottom 5 sellers by pizza_name
```
SELECT
	 pizza_name
	,SUM(quantity) AS Total_Pizzas_Sold
FROM
	pizza_sales
GROUP BY
	pizza_name
ORDER BY
	Total_Pizzas_Sold ASC
LIMIT 5;
```
 
Results and Discussions
Conclusion

