# APPLE-Sales-Analysis-using-SQL
![Image](https://github.com/nphan91/APPLE-Sales-Analysis-using-SQL/blob/main/Apple%20Logo.png)
## Overview
# SQL Queries

# 1. Find the number of stores in each country.
```sql
SELECT [country], COUNT([store_id]) AS [store_count]
FROM [stores]
GROUP BY [country]
ORDER BY [store_count] DESC;
```
# 2. Calculate the total number of units sold by each state
```sql
SELECT 
    S.[store_id], 
    ST.[store_name], 
    SUM(S.[quantity]) AS [total_unit_sold]
FROM 
    [sales] S
JOIN 
    [stores] ST
ON 
    S.[store_id] = ST.[store_id]
GROUP BY 
    S.[store_id], ST.[store_name]
ORDER BY 
    SUM(S.[quantity]) DESC;
```
# 3. Identify how many sales occurred in December 2023.
```SQL
SELECT COUNT([sale_id]) AS [total_sales]
FROM [sales]
WHERE FORMAT([sale_date], 'MM-yyyy') = '12-2023';
```
# 4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT COUNT(*) AS [stores_without_warranty_claims]
FROM [stores]
WHERE [store_id] NOT IN (
    SELECT DISTINCT S.[store_id]
    FROM [sales] AS S
    RIGHT JOIN [warranty] AS W
    ON S.[sale_id] = W.[sale_id]
);
```

# 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT 
    ROUND(
        CAST(COUNT([claim_id]) AS FLOAT) / CAST((SELECT COUNT(*) FROM [warranty]) AS FLOAT) * 100, 
        2
    ) AS [warranty_void_percentage]
FROM 
    [warranty]
WHERE 
    [repair_status] = 'Warranty Void';

```

Identify which store had the highest total units sold in the last year.
Count the number of unique products sold in the last year.
Find the average price of products in each category.
How many warranty claims were filed in 2020?
For each store, identify the best-selling day based on highest quantity sold.
Medium to Hard (5 Questions)
Identify the least selling product in each country for each year based on total units sold.
Calculate how many warranty claims were filed within 180 days of a product sale.
Determine how many warranty claims were filed for products launched in the last two years.
List the months in the last three years where sales exceeded 5,000 units in the USA.
Identify the product category with the most warranty claims filed in the last two years.
Complex (5 Questions)
Determine the percentage chance of receiving warranty claims after each purchase for each country.
Analyze the year-by-year growth ratio for each store.
Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
Bonus Question
Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
