# UK E-Commerce Sales Analysis
## Overview
This project analyzes sales data from a UK-based non-store online retail company, focusing on customer behavior, revenue trends, and product performance. By examining transactions from December 1, 2010, to December 9, 2011, we uncover valuable business insights that can inform strategic decisions.

## Table of Contents
- [Overview](#overview)
- [Business Challenge](#business-challenge)
- [Key Insights](#key-insights)
  - [Seasonal Sales Trends](#1-seasonal-sales-trends)
  - [Revenue by Country](#2-revenue-by-country)
  - [Order Volume by Hour](#3-order-volume-by-hour)
  - [Top-Selling Products](#4-top-selling-products)
  - [Customer & Geographical Insights](#5-customer--geographical-insights)
- [Visuals](#visuals)
- [Repository Contents](#repository-contents)
- [Tools Used](#tools-used)
- [Conclusion](#conclusion)

## Business Challenge
The company primarily sells unique all-occasion gifts, with many customers being wholesalers. Understanding sales trends, customer behavior, and product performance is crucial for optimizing stock levels, marketing efforts, and overall revenue.

## Key Insights
### 1. Seasonal Sales Trends
---
-	November and December recorded the highest sales, with £1.46M and £1.18M, respectively.
-	This suggests a strong seasonal impact, likely driven by holiday shopping and increased consumer spending.
-	Businesses can leverage this insight by ramping up inventory and marketing efforts in the months leading up to peak sales periods.
### 2. Revenue by Country
---
-	The United Kingdom dominates total revenue with £8.12M, followed by the Netherlands (£284.64K) and EIRE (£263.59K).
-	This suggests that these regions are key revenue drivers, and businesses can tailor marketing campaigns and promotions to further engage customers in these markets.
### 3. Order Volume by Hour
---
-	Order volume across all countries peaked between 10 AM and 4 PM, with the highest spike at 12 PM, recording 77,573 orders.
-	Businesses can optimize promotions, customer support, and website performance during these hours to enhance user experience and increase conversions.
### 4. Top-Selling Products
---
Every product tells a story, revealing customer preferences and purchasing trends. The data highlights which items resonated most with buyers, uncovering patterns that can guide future inventory and marketing strategies.
-	The most sold product was World War 2 Gliders Assorted Designs, with 53,751 units purchased. This suggests that nostalgic, novelty items appeal to a broad audience and maintain strong demand.
-	The highest revenue-generating product was the Regency Cake Stand 3 Tier, generating £164,399 in total sales. This reflects a strong market for premium, decorative home and event items, likely driven by catering businesses and individuals purchasing for special occasions.
-	These insights highlight the importance of understanding consumer preferences, allowing businesses to tailor marketing strategies, prioritize inventory, and maximize revenue opportunities.
### 5. Customer & Geographical Insights
---
-	The dataset contains transactions from 4,315 unique customers, each contributing to the overall sales volume.
-	The top customer identified was Customer ID 14911 from EIRE, with a purchase count of 248 transactions, making them the highest purchase frequency customer.
-	Understanding high-value customers and their geographic locations helps businesses personalize marketing strategies and improve customer engagement.
-	The dataset also highlights Stock Code 23843, "PAPER CRAFT, LITTLE BIRDIE," as the most returned product, with 80,995 returns.
-	The United Kingdom recorded the highest total quantity of returns, with 462,633 returned units.
-	Understanding product return trends helps businesses address potential quality issues, improve customer satisfaction, and adjust inventory management strategies.
## Visuals
---
The following dashboards provide a comprehensive view of the analysis:
-	Sales Review Dashboard: Revenue trends, number of customers, total number of transactions, order volume, and customer spending patterns.
-	Customer & Geographical Insights Dashboard: Total transactions by customer and their respective countries, geographical distribution of returns, monthly financial overview (total sales, refunds, and revenue), and stock code vs. return quantity.
## Repository Contents
---
-	sales3.csv – Cleaned dataset used for analysis
-	Dashboard Images – Key visualizations from the analysis
-	SQL Queries – Data cleaning, processing, and exploration
## Tools Used
---
-	SQL (MySQL) for data cleaning, analysis, and querying structured data
-	Power BI for visualization
## •	Conclusion
---
This analysis provides actionable insights into seasonal trends, customer behavior, and top-performing products. Businesses can leverage these findings to optimize marketing campaigns, inventory management, and customer engagement strategies.
For the full project and code, visit the GitHub Repository.

```sql
-- CREATING A NEW DATABASE CALLED SALES
CREATE DATABASE sales;

-- MAKING USE OF THE DATABASE
USE sales;

-- CREATING A TABLE TO IMPORT THE SALES DATASET
CREATE TABLE SALES 
(
InvoiceNo VARCHAR(255),
StockCode VARCHAR(255),
`Description` VARCHAR(255),
Quantity INT,
InvoiceDate DATETIME,
UnitPrice FLOAT,
CustomerID INT,
Country VARCHAR(255)
);

-- TO VIEW WHAT THE NEWLY CREATED TABLE LOOK LIKE
SELECT *
FROM SALES;

-- LOADING ONLINE STORE2 DATASET INTO SALES TABLE
LOAD DATA INFILE 'Online store2.csv' INTO TABLE sales
FIELDS TERMINATED BY ','
IGNORE 1 LINES; -- GOT AN ERROR BECAUSE MYSQL HAVE A DESIGNATED DIRECTORY FOR SAVING ITS FILE('C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/)

SHOW VARIABLES LIKE 'secure_file_priv'; -- TO SEE THE DIRECTORY FOR SAVING MYSQL FILES

-- LOADING ONLINE STORE2 DATASET INTO SALES TABLE USING MYSL DIRECTORY
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Online store2.csv' 
INTO TABLE sales
FIELDS TERMINATED BY ','
IGNORE 1 LINES; -- IT DIDN'T WORK

-- ADDING ENCLOSED AND REPLACING VALUE COLUMNS CONTAINING EMPTY FIELD TO NULL
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Online store2_UTF8.csv'
INTO TABLE sales
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'  -- This ensures quoted text fields are treated correctly
IGNORE 1 LINES
(InvoiceNo, StockCode, Description, @Quantity, @InvoiceDate, @UnitPrice, @CustomerID, Country)
SET 
    Quantity = NULLIF(@Quantity, ''), 
    InvoiceDate = STR_TO_DATE(@InvoiceDate, '%Y-%m-%d %H:%i:%s'),
    UnitPrice = NULLIF(@UnitPrice, ''),
    CustomerID = NULLIF(@CustomerID, ''); -- IT WORKED, 541,909 ROWS WERE IMPORTED INTO SALES TABLE

-- TRY TO TROUBLESHOOT WHERE THE ACTUAL PROBLEM CAME FROM, 
-- BY CREATING A NEW TABLE LIKE THE SALES TABLE THEN ADDING JUST THE ENCLOSURE(ENCLOSED BY '"')
-- WHEN LOADING THE DATASET INTO THE NEWLY CREATED SALES2 TABLE
CREATE TABLE sales2
LIKE SALES;

SELECT *
FROM sales3;

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Online store2_UTF8.csv' 
INTO TABLE sales3
FIELDS TERMINATED BY ','
ENCLOSED BY '"' 
IGNORE 1 LINES; -- IT WORKED 541,909 ROWS WERE IMPORTED INTO SALES2 TABLE

-- N0W WE WILL be working with THE SALES 2 TABLE

-- viewing the new table
SELECT * 
FROM sales3;

-- lets try cleaning the columns. While scrutinizing the dataset on excel(we filled the columns with blank fields, so there's no blank or null fields)

-- Removing duplcates 
-- Since we can't use the invoiceNO to remove duplicates because some rows contains same invoiceNO with different products,
-- we will be making use of row number() and cte to remove duplicated rows

-- TO FIRST VIEW THE DUPLICATED ROWS BEFORE DELETING
with duplicated_rows as 
(
SELECT *, 
ROW_NUMBER() OVER(PARTITION BY InvoiceNO, StockCode, `Description`, Quantity, InvoiceDate, UnitPrice, CustomerID, Country ORDER BY InvoiceNO
) AS ROW_NUM
FROM Sales3
)
SELECT * 
FROM duplicated_rows
WHERE ROW_NUM > 1;

WITH DuplicateRows AS (
    SELECT *, 
           ROW_NUMBER() OVER (
               PARTITION BY InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country 
               ORDER BY InvoiceDate
           ) AS row_num
    FROM sales3
)
SELECT *, 
       CASE 
           WHEN row_num = 1 THEN 'Original'
           ELSE 'Duplicate'
       END AS Duplicate_Status
FROM DuplicateRows; 

-- SINCE OUR DATA HAS DUPLICATED ROWS WE WILL NEED TO ADD AN ID COLUMN WHICH HAS AN AUTO INCREMEMENT AND A PRIMARY KEY
ALTER TABLE SALES3 ADD COLUMN ID INT AUTO_INCREMENT PRIMARY KEY;
SELECT *
FROM sales3;

-- DELETING THE DUPLICATED ROWS USING THE ID COLUMN
DELETE FROM SALES2
 WHERE ID IN (
      SELECT ID FROM ( 
           SELECT id, 
		   ROW_NUMBER() OVER (
		   PARTITION BY InvoiceNO, StockCode, `Description`, Quantity, InvoiceDate, UnitPrice, CustomerID, Country 
		   ORDER BY InvoiceNO
	       ) AS ROW_num
           FROM sales3
	  ) AS temp_table
WHERE ROW_NUM > 1 ); -- 5,268 ROWS WHERE DELETED FROM THE TABLE

-- STANDARDIZING EACH COLUMN
SELECT *
FROM SALES3
LIMIT 20;

-- InvoiceNo
-- Checking for spaces
SELECT InvoiceNo
FROM SALES3
WHERE InvoiceNo like '% ' or ' %'; -- no spaces

-- checking for special character 
SELECT InvoiceNO
FROM SALES3
WHERE InvoiceNO REGEXP '[^a-zA-Z0-9]'; -- no special character

-- Stockcode

SELECT *
FROM SALES3
LIMIT 20;

SELECT Stockcode
FROM SALES3
WHERE Stockcode like '% ' or ' %'; -- no spaces

-- checking special characters
SELECT DISTINCT Stockcode
FROM SALES3
WHERE Stockcode REGEXP '[^a-zA-Z0-9]'; -- contains ('gift_0001_20', 'BANK CHARGES'). WE WON'T BE CLEANING IT 

-- Description 
-- checking for spaces
SELECT `Description`
FROM SALES3
WHERE `Description` like '% ' or ' %'; -- returned 111,555 rows

SELECT `Description`, TRIM(`Description`)
FROM SALES3
WHERE `Description` like '% ' or ' %';

-- TRIMING THE COLUMN
UPDATE SALES3
SET `DESCRIPTION` = TRIM(`Description`)
					WHERE `DESCRIPTION` LIKE '% ' OR `DESCRIPTION` LIKE ' %'; -- 112371 CHANGED 

-- CHECKING IF THERE ARE STILL LEADING OR TRAILING SPACES
SELECT `Description`
FROM SALES3
WHERE `Description` like '% ' or ' %'; -- NO SPACES

-- CHECKING FOR SPECIAL CHARACTERS
SELECT DISTINCT `Description`
FROM SALES3
WHERE `Description` REGEXP '[^a-zA-Z0-9, ('',+,.-?/,!) ""]'; -- There are special characters but they are unique to each product

-- UNITPRICE
SELECT UnitPrice 
FROM SALES3;

-- checking for spaces
SELECT UNITPRICE
FROM SALES3
WHERE UNITPRICE LIKE '% ' OR ' %'; -- NO SPACES

-- checking for special characters
SELECT *
FROM SALES3
WHERE UnitPrice REGEXP '[^0-9, .]'; -- there is special characters like '-11062.1' with adescription of 'Adjust bad debt'

-- CustomerID
SELECT CustomerID
FROM SALES3
WHERE CustomerID LIKE '% ' OR ' %'; -- NO SPACES

-- checking for special characters
SELECT CustomerID 
FROM SALES3
WHERE  CustomerID REGEXP '[^0-9]'; -- No special CHARACTERS

-- COUNTRY
SELECT DISTINCT COUNTRY
FROM SALES3
ORDER BY COUNTRY ASC; -- 38 COUNTRIES(NO SPECIAL CHARACTERS)

SELECT COUNTRY
FROM SALES3
WHERE COUNTRY LIKE '% ' OR ' %'; -- NO SPACES

-- EXPLORATORY ANALYSIS
 #Customer Behavior Analysis
 
-- TOP TEN CUSTOMERS WITH THE HIGHEST PURCHASE  AND THE NUMBER OF TIMES THE MADE AN ORDER
 SELECT CUSTOMERID, COUNT(*) AS Purchase_count, SUM(QUANTITY) AS NETQUANTITY, TRUNCATE(SUM(UNITPRICE * QUANTITY),0) AS Total_spent
 FROM SALES3
 WHERE CUSTOMERID != 0
 GROUP BY CUSTOMERID 
 HAVING NETQUANTITY > 0 
 ORDER BY Total_spent DESC
 LIMIT 10;



-- FindING the most frequently bought product for each customer,
 -- WE WILL BE USING CTES
 WITH CLEANED_SALES AS
 (
 SELECT CUSTOMERID, STOCKCODE, DESCRIPTION, SUM(QUANTITY) AS NET_QUANTITY
 FROM SALES3
 WHERE UNITPRICE > 0 AND CUSTOMERID IS NOT NULL
 GROUP BY CUSTOMERID, STOCKCODE, DESCRIPTION
 HAVING NET_QUANTITY > 0
 ),
MOST_PURCHASE_GOODS AS
(
SELECT CUSTOMERID, STOCKCODE, `DESCRIPTION`, NET_QUANTITY AS TOTAL_QUANTITY,
ROW_NUMBER() OVER(PARTITION BY CUSTOMERID ORDER BY NET_QUANTITY DESC
)AS RANK_NUM
FROM CLEANED_SALES
) 
SELECT CUSTOMERID, STOCKCODE, `DESCRIPTION`, TOTAL_QUANTITY, RANK_NUM
FROM MOST_PURCHASE_GOODS
WHERE RANK_NUM = 1;

-- Identify the most valuable customers based on: Recency (how recently they bought)
-- Frequency (how often they buy) -- Monetary value (how much they spend)
SELECT CUSTOMERID,
	   MAX(INVOICEDATE) AS LAST_DATE, 
       COUNT(DISTINCT INVOICENO) AS PURCHASE_FREQUENCY, 
       TRUNCATE(SUM(QUANTITY * UNITPRICE), 0) AS TOTAL_SPENT
FROM SALES3
WHERE CUSTOMERID != 0
GROUP BY CUSTOMERID
HAVING SUM(QUANTITY) > 0 AND SUM(QUANTITY * UNITPRICE) > 0
ORDER BY TOTAL_SPENT DESC, LAST_DATE DESC; 

-- Finding out which countrY or regions generate the most sales
CREATE VIEW COUNTRIES_WITH_THE_HIGHEST_PURCHASE AS
SELECT COUNTRY, 
       TRUNCATE(SUM(QUANTITY * UNITPRICE), 0) AS Total_Revenue,
       COUNT(DISTINCT CUSTOMERID) AS Unique_Customers
FROM SALES3
GROUP BY COUNTRY
HAVING SUM(QUANTITY) > 0
ORDER BY Total_Revenue DESC;

-- Creating a view to analyze the number of purchases made by customers. 
CREATE VIEW customer_purchase_frequencyy AS  
SELECT  
    CustomerID AS CUSTOMERID,  
    Country AS COUNTRY,  
    YEAR(InvoiceDate) AS PURCHASE_YEAR,  
    MONTH(InvoiceDate) AS PURCHASE_MONTH,  
    COUNT(DISTINCT InvoiceNo) AS PURCHASE_FREQUENCY,  
    TRUNCATE(SUM(CASE WHEN Quantity > 0 THEN Quantity * UnitPrice ELSE 0 END), 0) AS TOTAL_PURCHASES,  
    TRUNCATE(SUM(CASE WHEN Quantity < 0 THEN Quantity * UnitPrice ELSE 0 END), 0) AS TOTAL_REFUNDS,  
    TRUNCATE(SUM(Quantity * UnitPrice), 0) AS NET_TOTAL_SPENT  
FROM sales3  
WHERE CustomerID <> 0  
GROUP BY CustomerID, Country, PURCHASE_YEAR, PURCHASE_MONTH  
HAVING SUM(Quantity) > 0 AND SUM(Quantity * UnitPrice) > 0  
ORDER BY PURCHASE_YEAR, PURCHASE_MONTH, NET_TOTAL_SPENT DESC;

-- analyzing monthly revenue
SELECT  
    Country AS COUNTRY,  
    MONTH(InvoiceDate) AS ORDER_MONTH,  
    CustomerID,  
    COUNT(DISTINCT InvoiceNo) AS TOTAL_ORDERS,  
    TRUNCATE(SUM(Quantity * UnitPrice), 0) AS TOTAL_REVENUE  
FROM sales3  
GROUP BY Country, ORDER_MONTH, CustomerID  
HAVING SUM(Quantity) > 0  
ORDER BY Country, ORDER_MONTH, CustomerID;

CREATE VIEW orders_by_month_countryy AS -- creating view for the monthly revenue
SELECT  
    Country AS COUNTRY,  
    MONTH(InvoiceDate) AS ORDER_MONTH,  
    CustomerID,  
    COUNT(DISTINCT InvoiceNo) AS TOTAL_ORDERS,  
    TRUNCATE(SUM(Quantity * UnitPrice), 0) AS TOTAL_REVENUE  
FROM sales3  
GROUP BY Country, ORDER_MONTH, CustomerID  
HAVING SUM(Quantity) > 0  
ORDER BY Country, ORDER_MONTH, CustomerID;


-- Creating a view to analyze the products with the highest return rates by customers.
CREATE VIEW highreturn_product_analysis AS 
SELECT  
    Country AS COUNTRY,  
    MONTH(InvoiceDate) AS RETURN_MONTH,  
    StockCode AS STOCKCODE,  
    SUM(Quantity) AS TOTAL_RETURNS  
FROM sales3  
WHERE Quantity < 0  
GROUP BY Country, RETURN_MONTH, StockCode  
HAVING StockCode <> '72140F'  
ORDER BY TOTAL_RETURNS DESC;

--  identifying peak sales periods for better business decisions.
SELECT HOUR(InvoiceDate) AS Purchase_Hour,  
    COUNT(*) AS NUMBER_OF_TIMES,  
    SUM(Quantity) AS Order_Volume,  
    Country AS COUNTRY 
FROM sales3  
GROUP BY Purchase_Hour, Country  
HAVING SUM(Quantity) > 0;
    
CREATE VIEW order_volume_by_hours AS 
SELECT HOUR(InvoiceDate) AS Purchase_Hour,  
    COUNT(*) AS NUMBER_OF_TIMES,  
    SUM(Quantity) AS Order_Volume,  
    Country AS COUNTRY  
FROM sales3  
GROUP BY Purchase_Hour, Country  
HAVING SUM(Quantity) > 0;


-- Top ten product sales by country and month
SELECT Country AS COUNTRY,  
    MONTH(InvoiceDate) AS ORDER_MONTH,  
    StockCode AS STOCKCODE,  
    Description AS DESCRIPTION,  
    SUM(Quantity) AS TOTAL_QUANTITY_SOLD,  
    TRUNCATE(SUM(Quantity * UnitPrice), 0) AS TOTAL_REVENUE  
FROM sales3  
WHERE Description <> 'UNKNOWN'  
GROUP BY Country, ORDER_MONTH, StockCode, Description  
HAVING SUM(Quantity) > 0
ORDER BY Country, ORDER_MONTH, TOTAL_QUANTITY_SOLD DESC
LIMIT 10;

CREATE VIEW top_ten_products_by_month_country AS 
SELECT  
    Country AS COUNTRY,  
    MONTH(InvoiceDate) AS ORDER_MONTH,  
    StockCode AS STOCKCODE,  
    Description AS DESCRIPTION,  
    SUM(Quantity) AS TOTAL_QUANTITY_SOLD,  
    TRUNCATE(SUM(Quantity * UnitPrice), 0) AS TOTAL_REVENUE  
FROM sales3  
WHERE Description <> 'UNKNOWN'  
GROUP BY Country, ORDER_MONTH, StockCode, Description  
HAVING SUM(Quantity) > 0  
ORDER BY Country, ORDER_MONTH, TOTAL_QUANTITY_SOLD DESC;

-- analyzing customer spending behavior by country and month
SELECT CustomerID AS CUSTOMERID,  
    Country AS COUNTRY,  
    YEAR(InvoiceDate) AS PURCHASE_YEAR,  
    MONTH(InvoiceDate) AS PURCHASE_MONTH,  
    COUNT(*) AS Purchase_count,  
    COUNT(DISTINCT CAST(InvoiceDate AS DATE)) AS Days_active,  
    SUM(Quantity) AS NETQUANTITY,  
    TRUNCATE(SUM(UnitPrice * Quantity), 0) AS Total_spent,  
    TRUNCATE(SUM(UnitPrice * Quantity) / COUNT(*), 0) AS Avg_spent_per_order  
FROM sales3  
WHERE CustomerID <> 0  
GROUP BY CustomerID, Country, PURCHASE_YEAR, PURCHASE_MONTH  
HAVING NETQUANTITY > 0 AND Total_spent > 0  
ORDER BY Total_spent DESC;

CREATE VIEW top_customers_and_avg_spend_per_day AS 
SELECT  
    CustomerID AS CUSTOMERID,  
    Country AS COUNTRY,  
    YEAR(InvoiceDate) AS PURCHASE_YEAR,  
    MONTH(InvoiceDate) AS PURCHASE_MONTH,  
    COUNT(*) AS Purchase_count,  
    COUNT(DISTINCT CAST(InvoiceDate AS DATE)) AS Days_active,  
    SUM(Quantity) AS NETQUANTITY,  
    TRUNCATE(SUM(UnitPrice * Quantity), 0) AS Total_spent,  
    TRUNCATE(SUM(UnitPrice * Quantity) / COUNT(*), 0) AS Avg_spent_per_order  
FROM sales3  
WHERE CustomerID <> 0  
GROUP BY CustomerID, Country, PURCHASE_YEAR, PURCHASE_MONTH  
HAVING NETQUANTITY > 0 AND Total_spent > 0  
ORDER BY Total_spent DESC;

-- to extracts a list of unique customers who have made valid purchases.
SELECT  
    CustomerID AS CUSTOMERID  
FROM sales3  
WHERE CustomerID <> 0  
GROUP BY CustomerID  
HAVING SUM(Quantity) > 0 AND SUM(Quantity * UnitPrice) > 0;

CREATE VIEW unique_customers AS 
SELECT  
    CustomerID AS CUSTOMERID  
FROM sales3  
WHERE CustomerID <> 0  
GROUP BY CustomerID  
HAVING SUM(Quantity) > 0 AND SUM(Quantity * UnitPrice) > 0;

-- Saving the file in mysql directory
SELECT * 
FROM sales3  
INTO OUTFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/sales3.csv'  
FIELDS TERMINATED BY ','  
ENCLOSED BY '"'  
LINES TERMINATED BY '\n';
```










 











































