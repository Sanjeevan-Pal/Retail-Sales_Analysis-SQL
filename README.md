# Retail Sales Analysis Project using SQL
##  Overview

This project involves an end-to-end data analysis of a 
retail sales dataset using Microsoft SQL Server (MSSQL).
The goal is to explore the data, uncover key business 
insights, and answer real-world business questions through 
structured SQL queries.

The analysis covers:
- Sales trends across different time periods
- Customer behavior and top performing customers
- Category-wise and gender-wise sales performance
- Shift-based purchase patterns (Morning/Afternoon/Evening)
- Best performing months by average sales

**Tool:** Microsoft SQL Server (MSSQL/SSMS)
**Skills:** Data Cleaning, EDA, Advanced SQL

## Project Structure
### 1. Creating Database
- The project starts by creating the database `SQL_PROJECT1`
```sql
CREATE DATABASE SQL_PROJECT1
USE SQL_PROJECT1
```
- A table is created named `Retail_Sales` to store the data of the sales. The table contains the column of transaction_ID, sales_date, sale_time, customer_ID, gender,age, category, quantity, price per unit, cogs, and total sales.
```sql
CREATE TABLE Retail_Sales(
	Transaction_ID INT PRIMARY KEY,
	Sales_Date DATE,
	Sale_time TIME,
	Customer_ID INT,
	Gender VARCHAR(7),
	Age INT,
	Category VARCHAR(15),
	Quantity INT,
	Price_per_unit FLOAT,
	Cogs FLOAT,
	Total_Sales FLOAT
	)
INSERT INTO Retail_Sales
SELECT * 
FROM SQL_PROJECT1.dbo.RETAIL;
```
- The column names, their data-type and description about them is given below:-

| Column | Data Type | Description |
|---|---|---|
| Transaction_ID | INT | Unique transaction ID (Primary Key) |
| Sales_Date | DATE | Date of transaction |
| Sale_Time | TIME | Time of transaction |
| Customer_ID | INT | Unique customer identifier |
| Gender | VARCHAR(7) | Customer gender |
| Age | INT | Customer age |
| Category | VARCHAR(15) | Product category |
| Quantity | INT | Units purchased |
| Price_per_unit | FLOAT | Price per unit |
| Cogs | FLOAT | Cost of goods sold |
| Total_Sales | FLOAT | Total sale amount |

---

### 2. Data Cleaning
- Checked NULL values across all 11 columns.
```sql
SELECT *
FROM Retail_sales
WHERE
	Transaction_ID IS NULL
	OR
	Sales_date IS NULL
	OR 
	Sale_Time IS NULL
	OR 
	Customer_ID IS NULL
	OR 
	Gender IS NULL
	OR
	Age IS NULL
	OR 
	Category IS NULL
	OR 
	Quantity IS NULL
	OR 
	Price_per_unit IS NULL
	OR 
	Cogs IS NULL
	OR 
	Total_Sales IS NULL;
```
- By checking the NULL values, it is found that there are only 13 rows where NULL values are present. 
- Deleted incomplete/corrupted records using DELETE + IS NULL, it is done to ensure data-accuracy and integrity.
```sql
DELETE FROM Retail_Sales
WHERE 
	Transaction_ID IS NULL
	OR
	Sales_date IS NULL
	OR 
	Sale_Time IS NULL
	OR 
	Customer_ID IS NULL
	OR 
	Gender IS NULL
	OR
	Age IS NULL
	OR 
	Category IS NULL
	OR 
	Quantity IS NULL
	OR 
	Price_per_unit IS NULL
	OR 
	Cogs IS NULL
	OR 
	Total_Sales IS NULL;
```

### 3. Data Exploration
- **Total Transactions:** Counting total number of transactions recorded in the dataset
```sql
SELECT COUNT(*) AS [Total Sales]
FROM Retail_Sales;
```
- **Total Unique Customers:** Counting total number of unique customers who made purchases
```sql
SELECT COUNT(DISTINCT(Customer_ID)) as [Total Customers]
FROM Retail_Sales;
```
- **Total Categories:** Counting total number of distinct product categories available
```sql
SELECT COUNT(DISTINCT(Category)) AS [Distinct Category]
FROM Retail_Sales;
```
- **Names of the Categories:** Retrieving the names of all available product categories
```sql
SELECT DISTINCT Category AS [Types of Categories]
FROM Retail_Sales;
```
- So after the data-exploration of this dataset, it is found that there are total of 1987 sales were made, in which there are 155 numbers of unique customers are present.
- Also, it has also been found that there are total of 3 categories of purchases customers are making, and they are beauty, electronics and clothing.
---

### 4. Business Related Analysis

#### 1. Sales on a specific date
Retrieved all transactions on 2022-11-05
```sql
SELECT*
FROM Retail_Sales
WHERE Sales_Date LIKE '2022-11-05';
```

#### 2. Clothing sales filter
Filtered Clothing category with quantity ≥ 4 in November 2022
```sql
SELECT*
FROM Retail_Sales
WHERE 
	Category = 'Clothing' 
	AND 
	Quantity >= 4
	AND 
	MONTH(Sales_Date) = 11
	AND
	YEAR(Sales_Date) = 2022;
```

#### 3. Total sales & purchases by category
Aggregated total revenue and order count per category
```sql

SELECT 
	Category, 
	SUM(Total_Sales) AS [Total Sales], 
	COUNT(Total_Sales) AS [Total Purchases]
FROM Retail_Sales
GROUP BY Category
ORDER BY SUM(Total_Sales);
```

#### 4. Average age of Beauty category customers
Found average customer age purchasing Beauty products
```sql
SELECT ROUND(AVG(Age),2) AS [Average Age of Beauty Category]
FROM Retail_Sales
WHERE Category LIKE 'Beauty';
```

#### 5. High value transactions
Retrieved all transactions where Total Sales > 1000
```sql
SELECT*
FROM Retail_Sales
WHERE Total_Sales>1000;
```

#### 6. Transactions by category and gender
Counted total transactions grouped by category and gender
```sql
SELECT
	Category,
	Gender,
	COUNT(*) AS [Total Transactions]
FROM Retail_Sales
GROUP BY 
	Category,
	Gender
ORDER BY 
	Category, 
	COUNT(*);
```

### 7. Best selling month per year
Used Window Functions (RANK + PARTITION BY YEAR) to find highest average sales month for each year
```sql
SELECT*
FROM (
	SELECT 
		YEAR(Sales_Date) AS [Year],
		MONTH(Sales_Date) AS [Month Number],
		ROUND(AVG(Total_Sales),2) AS [Average Sale],
		RANK() OVER(
			PARTITION BY YEAR(Sales_Date) 
			ORDER BY AVG(Total_Sales) DESC
			) AS [Rank]
	FROM Retail_Sales
	GROUP BY 
		YEAR(Sales_Date),
		MONTH(Sales_Date)
	) AS T1
WHERE RANK=1;
```

#### 8. Top 5 customers by total sales
Identified top 5 highest spending customers
```sql
SELECT TOP 5
	Customer_ID,
	SUM(Total_Sales) AS [Total Sales]
FROM Retail_Sales
GROUP BY Customer_ID
ORDER BY SUM(Total_Sales) DESC;
```

#### 9. Unique customers per category
Counted distinct customers per product category
```sql
SELECT
	Category,
	COUNT(DISTINCT(Customer_ID)) AS [Unique Customers]
FROM Retail_Sales
GROUP BY Category
ORDER BY COUNT(DISTINCT(Customer_ID));
```

#### 10. Sales by time shift
Classified transactions into Morning / Afternoon / Evening using CASE + DATEPART — solved using both Subquery and CTE
```sql
WITH ShiftCTE AS (
	SELECT*,
	CASE 
		WHEN DATEPART(HOUR, Sale_Time) < 12 THEN 'Morning'
		WHEN DATEPART(HOUR, Sale_Time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END AS [Shift]
	FROM Retail_Sales
	)
SELECT
	Shift,
	COUNT(*) AS [Total Purchases]
FROM ShiftCTE
GROUP BY Shift;
```

## 5. SQL Concepts Used
- DDL: CREATE DATABASE, CREATE TABLE
- DML: INSERT, DELETE, SELECT
- Aggregate Functions: SUM, COUNT, AVG, ROUND
- Window Functions: RANK, PARTITION BY
- CASE Statements
- CTEs (Common Table Expressions)
- Subqueries
- DATEPART for time-based analysis
- Filtering: WHERE, AND, OR, IS NULL, LIKE

## 6. Key Findings & Conclusions

- **Best Performing Month:** The highest average sales month was found for each year using Window Functions — helping identify seasonal trends.

- **Top Category by Revenue:** The highest revenue was generated by the category of the Electronics. It was almost 35% of the total revenue.

- **Top 5 Customers:** A small group of customers contribute significantly to overall revenue useful for loyalty programs

- **Gender & Category Insights:** In each category the major purchasers were Male, while in the beauty category female gender has been the major buyers, which is also a quite knowing fact.

- **Average Customer Age (Beauty):** Beauty category attracts customers of average age of 40 years.

- **High Value Transactions:** Multiple transactions exceeded ₹1000 in total sales indicating premium purchasing behavior.

- **Shift Analysis:** Evening shift recorded the highest number of purchases useful for staffing and promotions.

- **Unique Customers per Category:** Each category attracts a distinct customer base with minimal overlap.

#  Author
**Sanjeevan Pal**
- This project is part of my portfolio, showcasing my SQL skills essential for data-analyst roles.
- [LinkedIn](https://www.linkedin.com/in/sanjeevan-pal-60444737b/) | [GitHub](https://github.com/Sanjeevan-Pal)
