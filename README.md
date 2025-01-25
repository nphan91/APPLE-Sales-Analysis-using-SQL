# APPLE-Sales-Analysis-using-SQL
![Image](https://github.com/nphan91/APPLE-Sales-Analysis-using-SQL/blob/main/Apple%20Logo.png)
## Overview
# SQL Queries

# 1. Find the number of stores in each country.
```sql
SELECT [country], COUNT([store_id]) AS [store_count]
FROM [dbo].[stores]
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
    [dbo].[sales] S
JOIN 
    [dbo].[stores] ST
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
FROM [dbo].[sales]
WHERE FORMAT([sale_date], 'MM-yyyy') = '12-2023';

```
# 4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT COUNT(*) AS [stores_without_warranty_claims]
FROM [dbo].[stores]
WHERE [store_id] NOT IN (
    SELECT DISTINCT S.[store_id]
    FROM [dbo].[sales] AS S
    RIGHT JOIN [dbo].[warranty] AS W
    ON S.[sale_id] = W.[sale_id]
);

```

# 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT 
    ROUND(
        CAST(COUNT([claim_id]) AS FLOAT) / CAST((SELECT COUNT(*) FROM [dbo].[warranty]) AS FLOAT) * 100, 
        2
    ) AS [warranty_void_percentage]
FROM 
    [dbo].[warranty]
WHERE 
    [repair_status] = 'Warranty Void';
```

# 6. Ientify which store had the highest total units sold in the last year.
```sql
SELECT TOP 1 
    s.[store_id], 
    st.[store_name],  
    SUM(s.[quantity]) AS [total_units_sold]
FROM 
    [dbo].[sales] s
JOIN 
    [dbo].[stores] st ON s.[store_id] = st.[store_id]
WHERE 
    s.[sale_date] >= DATEADD(YEAR, -1, GETDATE())
GROUP BY 
    s.[store_id], 
    st.[store_name]
ORDER BY 
    SUM(s.[quantity]) DESC;
```
# 7. Count the number of unique products sold in the last year.
```sql
SELECT 
    COUNT(DISTINCT s.[product_id]) AS [unique_products_sold]
FROM 
    [dbo].[sales] s
WHERE 
    s.[sale_date] >= DATEADD(YEAR, -1, GETDATE());
```

# 8. Find the average price of products in each category.
```sql
SELECT 
    c.[category_id], 
    c.[category_name], 
    AVG(p.[price]) AS [average_price]
FROM 
    [dbo].[products] p
JOIN 
    [dbo].[categories] c ON p.[category_id] = c.[category_id]
GROUP BY 
    c.[category_id], 
    c.[category_name];
```

# 9. How many warranty claims were filed in 2020?
```sql
SELECT 
    COUNT([claim_id]) AS [warranty_claims_2020]
FROM 
    [dbo].[warranty]
WHERE 
    YEAR([claim_date]) = 2020;
```
# 10 For each store, identify the best-selling day based on highest quantity sold.
```sql
SELECT 
    t.[store_id], 
    FORMAT(t.[sale_date], 'dddd') AS [day_name], 
    t.[total_unit_sold]
FROM 
    (
        SELECT 
            s.[store_id], 
            s.[sale_date], 
            SUM(s.[quantity]) AS [total_unit_sold], 
            RANK() OVER (PARTITION BY s.[store_id] ORDER BY SUM(s.[quantity]) DESC) AS [rank]
        FROM 
            [dbo].[sales] s
        GROUP BY 
            s.[store_id], 
            s.[sale_date]
    ) AS t
WHERE 
    t.[rank] = 1;
```

# 11. Identify the least selling product in each country for each year based on total units sold.
```sql
WITH RankedSales AS (
    SELECT 
        st.[country], 
        p.[product_name], 
        SUM(s.[quantity]) AS [total_qty_sold],
        YEAR(s.[sale_date]) AS [year],
        RANK() OVER (PARTITION BY st.[country], YEAR(s.[sale_date]) ORDER BY SUM(s.[quantity]) ASC) AS [rank]
    FROM 
        [dbo].[sales] s
    JOIN 
        [dbo].[stores] st ON s.[store_id] = st.[store_id]
    JOIN 
        [dbo].[products] p ON s.[product_id] = p.[product_id]
    GROUP BY 
        st.[country], 
        p.[product_name], 
        YEAR(s.[sale_date])
)
SELECT 
    [country], 
    [product_name], 
    [total_qty_sold], 
    [year]
FROM 
    RankedSales
WHERE 
    [rank] = 1
ORDER BY 
    [country], 
    [year], 
    [total_qty_sold];
```

# 12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT 
    COUNT(w.[claim_id]) AS [claims_within_180_days]
FROM 
    [dbo].[warranty] w
JOIN 
    [dbo].[sales] s ON w.[product_id] = s.[product_id]
WHERE 
    DATEDIFF(DAY, s.[sale_date], w.[claim_date]) <= 180;
```

# 13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT 
    COUNT(w.[claim_id]) AS [claims_for_recent_products]
FROM 
    [dbo].[warranty] w
JOIN 
    [dbo].[products] p ON w.[product_id] = p.[product_id]
WHERE 
    p.[launch_date] >= DATEADD(YEAR, -2, CURRENT_DATE);
```

# 14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT 
    YEAR(s.[sale_date]) AS [year], 
    MONTH(s.[sale_date]) AS [month], 
    SUM(s.[quantity]) AS [total_qty_sold]
FROM 
    [dbo].[sales] s
JOIN 
    [dbo].[stores] st ON s.[store_id] = st.[store_id]
WHERE 
    st.[country] = 'USA'
    AND s.[sale_date] >= DATEADD(YEAR, -3, CURRENT_DATE)
GROUP BY 
    YEAR(s.[sale_date]), 
    MONTH(s.[sale_date])
HAVING 
    SUM(s.[quantity]) > 5000
ORDER BY 
    [year], 
    [month];
```

# 15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT TOP 1
    p.[category], 
    COUNT(w.[claim_id]) AS [total_claims]
FROM 
    [dbo].[warranty] w
JOIN 
    [dbo].[products] p ON w.[product_id] = p.[product_id]
WHERE 
    p.[launch_date] >= DATEADD(YEAR, -2, CURRENT_DATE)
GROUP BY 
    p.[category]
ORDER BY 
    [total_claims] DESC;
```

# 16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
    st.[country],
    COUNT(s.[sale_id]) AS [total_sales],
    COUNT(w.[claim_id]) AS [total_claims],
    (COUNT(w.[claim_id]) * 100.0 / COUNT(s.[sale_id])) AS [warranty_claim_percentage]
FROM 
    [dbo].[sales] s
JOIN 
    [dbo].[stores] st ON s.[store_id] = st.[store_id]
LEFT JOIN 
    [dbo].[warranty] w ON s.[sale_id] = w.[sale_id]
GROUP BY 
    st.[country]
ORDER BY 
    [warranty_claim_percentage] DESC;
```

# 17. Analyze the year-by-year growth ratio for each store.
```sql
WITH YearlySales AS (
    SELECT 
        st.[store_id],
        YEAR(s.[sale_date]) AS [year],
        SUM(s.[quantity]) AS [total_qty_sold]
    FROM 
        [dbo].[sales] s
    JOIN 
        [dbo].[stores] st ON s.[store_id] = st.[store_id]
    GROUP BY 
        st.[store_id], YEAR(s.[sale_date])
),
GrowthRatio AS (
    SELECT 
        y1.[store_id],
        y1.[year],
        y1.[total_qty_sold],
        (y1.[total_qty_sold] - y2.[total_qty_sold]) * 1.0 / y2.[total_qty_sold] AS [growth_ratio]
    FROM 
        YearlySales y1
    LEFT JOIN 
        YearlySales y2 ON y1.[store_id] = y2.[store_id] AND y1.[year] = y2.[year] + 1
)
SELECT 
    st.[store_id],
    st.[store_name],
    gr.[year],
    ISNULL(gr.[growth_ratio], 0) AS [growth_ratio]
FROM 
    GrowthRatio gr
JOIN 
    [dbo].[stores] st ON gr.[store_id] = st.[store_id]
ORDER BY 
    st.[store_id], gr.[year];
```

Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
Bonus Question
Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
