# Walmart-Sales-Data-Analysis-Using-MySQL
This project focuses on the comprehensive analysis of the Walmart sales dataset through MySQL. The primary objectives are data cleaning and exploratory data analysis (EDA) to extract actionable insights and uncover significant trends.

-- Drop the database if it exists
DROP DATABASE IF EXISTS salesDataWalmart;

-- Create a new database named salesDataWalmart if it does not already exist
CREATE DATABASE IF NOT EXISTS salesDataWalmart;

-- Select the newly created database for use
USE salesDataWalmart;

-- Drop the WALMARTSALES table if it exists to avoid duplication
DROP TABLE IF EXISTS WALMARTSALES;

-- Create the WALMARTSALES table with the specified schema
CREATE TABLE IF NOT EXISTS WALMARTSALES(
    INVOICE_ID VARCHAR(30) NOT NULL PRIMARY KEY,
    BRANCH VARCHAR(5) NOT NULL,
    CITY VARCHAR(30) NOT NULL,
    CUSTOMER_TYPE VARCHAR(30) NOT NULL,
    GENDER VARCHAR(10) NOT NULL,
    PRODUCT_LINE VARCHAR(100) NOT NULL,
    UNIT_PRICE DECIMAL(10,2) NOT NULL,
    QUANTITY INT NOT NULL,
    VAT FLOAT NOT NULL,
    TOTAL DECIMAL(12,4) NOT NULL,
    DATE DATETIME NOT NULL,
    TIME TIME NOT NULL,
    PAYMENT_METHOD VARCHAR(15) NOT NULL,
    COGS DECIMAL(10,2) NOT NULL,
    GROSS_MARGIN_PCT FLOAT NOT NULL,
    GROSS_INCOME DECIMAL(12,4) NOT NULL,
    RATING FLOAT NOT NULL
);

-- Feature Engineering: Add a column to categorize the time of day
-- Determine the time of day and categorize as MORNING, AFTERNOON, or EVENING
SELECT TIME,
    (CASE
        WHEN TIME BETWEEN "00:00:00" AND "12:00:00" THEN "MORNING"
        WHEN TIME BETWEEN "12:01:00" AND "16:00:00" THEN "AFTERNOON"
        ELSE "EVENING"
    END
    ) AS TIME_OF_DAY
FROM WALMARTSALES;

-- Add a TIME_OF_DAY column to the table
ALTER TABLE WALMARTSALES ADD COLUMN TIME_OF_DAY VARCHAR(20);

-- Update the TIME_OF_DAY column with appropriate values
UPDATE WALMARTSALES
SET TIME_OF_DAY = (
    CASE
        WHEN TIME BETWEEN "00:00:00" AND "12:00:00" THEN "MORNING"
        WHEN TIME BETWEEN "12:01:00" AND "16:00:00" THEN "AFTERNOON"
        ELSE "EVENING"
    END
);

-- Feature Engineering: Add a column for the day of the week
-- Determine the day of the week for each sale
SELECT DATE,
    DAYNAME(DATE) AS DAY_NAME
FROM WALMARTSALES;

-- Add a DAY_NAME column to the table
ALTER TABLE WALMARTSALES ADD COLUMN DAY_NAME VARCHAR(10);

-- Update the DAY_NAME column with the appropriate values
UPDATE WALMARTSALES
SET DAY_NAME = DAYNAME(DATE);

-- Feature Engineering: Add a column for the month name
-- Determine the month name for each sale
SELECT DATE,
    MONTHNAME(DATE) AS MONTH_NAME
FROM WALMARTSALES;

-- Add a MONTH_NAME column to the table
ALTER TABLE WALMARTSALES ADD COLUMN MONTH_NAME VARCHAR(10);

-- Update the MONTH_NAME column with the appropriate values
UPDATE WALMARTSALES
SET MONTH_NAME = MONTHNAME(DATE);

-- Get a list of unique cities from the WALMARTSALES table
SELECT DISTINCT CITY
FROM WALMARTSALES;

-- Get a list of unique combinations of cities and branches
SELECT DISTINCT CITY, BRANCH
FROM WALMARTSALES;

-- Count the number of unique product lines in the WALMARTSALES table
SELECT COUNT(DISTINCT PRODUCT_LINE)
FROM WALMARTSALES;

-- Find the most common payment method and its count
SELECT PAYMENT_METHOD, COUNT(PAYMENT_METHOD) AS COUNT
FROM WALMARTSALES
GROUP BY PAYMENT_METHOD
ORDER BY COUNT DESC;

-- Identify the product line with the highest number of sales
SELECT PRODUCT_LINE, COUNT(PRODUCT_LINE) AS COUNT
FROM WALMARTSALES
GROUP BY PRODUCT_LINE
ORDER BY COUNT DESC;

-- Calculate total revenue for each month
SELECT MONTH_NAME AS MONTH,
    ROUND(SUM(TOTAL), 2) AS TOTAL_REVENUE
FROM WALMARTSALES
GROUP BY MONTH_NAME
ORDER BY TOTAL_REVENUE DESC;

-- Find the month with the highest total cost of goods sold (COGS)
SELECT MONTH_NAME AS MONTH, SUM(COGS) AS COGS
FROM WALMARTSALES
GROUP BY MONTH_NAME
ORDER BY COGS DESC;

-- Determine which product line generated the highest revenue
SELECT PRODUCT_LINE, ROUND(SUM(TOTAL), 2) AS TOTAL_REVENUE
FROM WALMARTSALES
GROUP BY PRODUCT_LINE
ORDER BY TOTAL_REVENUE DESC;

-- Identify the city with the highest total revenue
SELECT CITY, ROUND(SUM(TOTAL), 2) AS TOTAL_REVENUE
FROM WALMARTSALES
GROUP BY CITY
ORDER BY TOTAL_REVENUE DESC;

-- Find the city and branch combination with the highest revenue
SELECT CITY, BRANCH, ROUND(SUM(TOTAL), 2) AS TOTAL_REVENUE
FROM WALMARTSALES
GROUP BY CITY, BRANCH
ORDER BY TOTAL_REVENUE DESC;

-- Determine which product line had the largest total VAT
SELECT PRODUCT_LINE, ROUND(SUM(VAT), 2) AS TOTAL_VAT
FROM WALMARTSALES
GROUP BY PRODUCT_LINE
ORDER BY TOTAL_VAT DESC;

-- Compare each product line’s total sales to the average sales
-- Classify product lines as 'Good' or 'Bad' based on whether their sales are above or below the average
SELECT 
    PRODUCT_LINE,
    CASE
        WHEN SUM(TOTAL) > (SELECT AVG(TOTAL) FROM WALMARTSALES) THEN 'Good'
        ELSE 'Bad'
    END AS PERFORMANCE_REMARKS
FROM WALMARTSALES
GROUP BY PRODUCT_LINE;

-- Identify branches that sold more products than the average quantity sold
SELECT BRANCH,
    SUM(QUANTITY) AS QTY
FROM WALMARTSALES
GROUP BY BRANCH
HAVING SUM(QUANTITY) > (SELECT AVG(QUANTITY) FROM WALMARTSALES);

-- Determine the most common product line for each gender
SELECT GENDER, PRODUCT_LINE, COUNT(GENDER) AS TOTAL_COUNT
FROM WALMARTSALES
GROUP BY GENDER, PRODUCT_LINE
ORDER BY TOTAL_COUNT DESC;

-- Calculate the average rating for each product line
SELECT PRODUCT_LINE, ROUND(AVG(RATING), 2) AS AVG_RATING
FROM WALMARTSALES
GROUP BY PRODUCT_LINE
ORDER BY AVG_RATING DESC;
