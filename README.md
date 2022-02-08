# data_exploration_and_analysis
This is an SQL-based project that will help us to explore and analyze the Sales database and provide insights to stakeholders.
-- Select and activate the salesDB database
USE salesDB;


-- Retrieve the tables from the salesDB database
SELECT table_name from INFORMATION_SCHEMA.TABLES;

-- Get the total record of trasactions with duplicate values
SELECT COUNT(*) FROM transactions2;


-- Get the count of transactions based on currency used with duplicate values
SELECT currency, COUNT(*) [No. of Currency] FROM transactions2
GROUP BY currency ORDER BY 1;


-- Create a non duplicate transaction view
CREATE VIEW nonDuplicate_Txn
AS
SELECT DISTINCT * FROM transactions2;

-- Get records from the newly created view
SELECT * FROM nonDuplicate_Txn;


-- Get the records from the product table
SELECT * FROM products;


-- Get the records from the markets table
SELECT markets_code, markets_name, zone FROM markets ORDER BY markets_code;


-- Get the overall total Revenue and quantities sold
SELECT SUM(salesAmt) [Monthly Total Sales], SUM(sales_qty) [Total Quantity Sold]
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate;

-- Get the monthly total revenue and quantities sold (From the highest to the least)
SELECT Month_name, SUM(salesAmt) [Monthly Total Sales], SUM(sales_qty) [Total Quantity Sold]
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY month_name
ORDER BY 2 DESC;


-- Get the yearly total Revenue and quantities sold
SELECT sYear, SUM(salesAmt) [Yearly Total Sales], SUM(sales_qty) [Total Quantity Sold] 
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY sYear
ORDER BY sYear DESC;


-- Get the total Revenue and quantity sold by Year and Month
SELECT sYear, month_name, SUM(salesAmt) [Total Sales], SUM(sales_qty) [Total Quantity] 
FROM nonDuplicate_Txn n 
join date_tbl d on n.order_date = d.sDate
GROUP BY syear, month_name
ORDER BY sYear,2;


-- Get the total Revenue based on product_code
SELECT p.Product_code, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Quantities Sold] 
FROM nonDuplicate_Txn t
INNER JOIN products p
ON t.product_code = p.product_code
GROUP BY p.Product_code
ORDER BY p.Product_code;


-- Get the total Revenue based on markets_name (From the highest)
SELECT markets_name, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN markets m
ON t.market_code = m.markets_code
GROUP BY markets_name
ORDER BY SUM(salesAmt) DESC, markets_name;


-- Top three months with the highest Revenue
SELECT TOP 3 Month_name, SUM(salesAmt) [Monthly Total Sales] 
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY month_name
ORDER BY SUM(sales_amount) DESC;


-- Year with the highest Revenue
SELECT TOP 1 sYear, SUM(salesAmt) [Yearly Total Sales] 
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY sYear
ORDER BY SUM(sales_amount) DESC;


-- Top 5 Markets with the highest Revenue
SELECT TOP 5 markets_name, SUM(salesAmt) [Total Sales by Markets], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN markets m
ON t.market_code = m.markets_code
GROUP BY markets_name
ORDER BY SUM(sales_amount) DESC;


-- Showing Revenue of Markets by Year and month
SELECT syear, month_name, markets_name, SUM(salesAmt) [Total Sales by Markets], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN markets m
ON t.market_code = m.markets_code
INNER JOIN date_tbl d ON t.order_date = d.sdate
GROUP BY syear, month_name, markets_name
ORDER BY 1,SUM(sales_amount) DESC;


-- Top 5 Products with the highest Revenue
SELECT TOP 5 p.Product_code, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN products p
ON t.product_code = p.product_code
GROUP BY p.Product_code
ORDER BY SUM(sales_amount) DESC;


-- Showing Revenue of Products by Year and Month
SELECT syear, month_name, p.Product_code, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sdate
GROUP BY syear, month_name, p.Product_code
ORDER BY syear, month_name, SUM(sales_amount) DESC;


-- Top 5 Customers with the highest Revenue
SELECT TOP 5 c.customer_name, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN customers c
ON t.customer_code = c.customer_code
GROUP BY c.customer_name
ORDER BY SUM(sales_amount) DESC;


-- Showing Revenue of Customers by Year and Month
SELECT syear, month_name, customer_name, SUM(salesAmt) [Total Sales by Products], SUM(sales_qty) [Total Products Sold]
FROM nonDuplicate_Txn t
INNER JOIN customers c ON t.customer_code = c.customer_code
INNER JOIN date_tbl d ON t.order_date = d.sdate
GROUP BY syear, month_name, customer_name
ORDER BY syear, SUM(sales_amount) DESC;


-- Top five months with the least Revenue
SELECT TOP 5 Month_name, SUM(salesAmt) [Monthly Total Sales], SUM(sales_qty) [Total Quantities Sold]
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY month_name
ORDER BY SUM(sales_amount);


-- Year with the least Revenue
SELECT TOP 1 sYear, SUM(salesAmt) [Yearly Total Sales], SUM(sales_qty) [Total Quantities Sold]
FROM nonDuplicate_Txn t
INNER JOIN date_tbl d
ON t.order_date = d.sDate
GROUP BY sYear
ORDER BY SUM(sales_amount);


-- Top 5 Markets with the least Revenue
SELECT TOP 5 markets_name, SUM(salesAmt) [Total Sales by Products] 
FROM nonDuplicate_Txn t
INNER JOIN markets m
ON t.market_code = m.markets_code
GROUP BY markets_name
ORDER BY SUM(sales_amount);


-- Top 5 product_code with the least total sales
SELECT TOP 5 p.Product_code, SUM(salesAmt) [Total Sales by Products] 
FROM nonDuplicate_Txn t
INNER JOIN products p
ON t.product_code = p.product_code
GROUP BY p.Product_code
ORDER BY SUM(salesAmt);


-- Retrieve rolling total Revenue partitioned by product_code
SELECT d.sYear, p.Product_code, t.order_date, t.salesAmt, SUM(t.salesAmt) OVER (PARTITION BY p.product_code ORDER BY p.product_code, t.order_date) [Total Sales by Products] 
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sDate
--WHERE d.sYear IN(2017,2018,2019,2020)
--GROUP BY d.sYear, p.Product_code
ORDER BY 1,2,3;


-- Get the total quantities of products sold for each year
SELECT d.sYear, p.Product_code, SUM(t.sales_qty) AS [Total Product Sold Per Year]
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sDate
GROUP BY d.sYear, p.Product_code
ORDER BY 1, 3 DESC, 2;


-- Get the revenue of products for each year
SELECT d.sYear, p.Product_code, SUM(t.salesAmt) AS [Total Product Sales Per Year]
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sDate
GROUP BY d.sYear, p.Product_code
ORDER BY 1,3 DESC,2;


-- Get the total Revenue of products in each market for each year
SELECT d.sYear, m.markets_name, p.Product_code, SUM(t.salesAmt) AS [Total Product Sales Per Year]
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sDate
INNER JOIN markets m ON t.market_code = m.markets_code
GROUP BY d.sYear, m.markets_name, p.Product_code
ORDER BY 1, 2, 4 DESC;


-- Get the revenue of products in each zone for each year
SELECT d.sYear, m.zone, p.Product_code, SUM(t.salesAmt) AS [Total Product Sales Per Year]
FROM nonDuplicate_Txn t
INNER JOIN products p ON t.product_code = p.product_code
INNER JOIN date_tbl d ON t.order_date = d.sDate
INNER JOIN markets m ON t.market_code = m.markets_code
GROUP BY d.sYear, m.zone, p.Product_code
ORDER BY 1, 2, 4 DESC;
