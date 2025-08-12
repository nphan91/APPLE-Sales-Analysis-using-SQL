# APPLE-Sales-Analysis-using-SQL
![Image](https://github.com/nphan91/APPLE-Sales-Analysis-using-SQL/blob/main/Apple%20Logo.png)
## Overview
This project is focused on analyzing the sales data of Apple products across various stores and regions, with the goal of gaining valuable insights into product performance, trends, and customer behavior. By leveraging SQL queries to extract and analyze data, we aim to track key metrics such as the number of units sold, warranty claims, and sales growth over time.
![Image](https://github.com/nphan91/APPLE-Sales-Analysis-using-SQL/blob/main/erd.png)
## Purpose of the Project
The purpose of this project is to provide a comprehensive analysis of Apple's sales performance, identify patterns and trends, and assess the effectiveness of warranty management. The analysis will cover various aspects, including:
- Sales by Region: Understanding the distribution of sales across different countries and states to identify key markets.
- Product Performance: Analyzing the sales trends of individual products over different time periods, such as from launch to 6 months, 6-12 months, and beyond 18 months.
- Warranty Claims Analysis: Investigating warranty claims to assess the quality of products and understand the frequency of claims, including claims marked as "Warranty Void."
- Store Performance: Identifying top-performing stores based on total units sold, as well as analyzing sales trends on a year-by-year basis.
- Sales Growth and Correlation: Analyzing the correlation between product price and warranty claims, as well as tracking the year-on-year growth ratio for each store.

## Key Outcomes
- Insights into Sales Trends: We will gain insights into how Apple products are performing over time, segmented by different periods from launch. This will allow us to identify which products have long-term success and which may need further attention.
- Understanding of Market Segmentation: By analyzing sales data across various countries and states, we will identify which regions are most lucrative and which may need more targeted marketing efforts.
- Warranty Claim Trends: This project will highlight potential issues with products based on warranty claims, and help identify the percentage of claims that result in "Paid Repaired" status, signaling the quality of Apple’s customer support.
- Store and Product Performance: By focusing on the best-selling stores and products, we will uncover the key drivers of sales success and identify areas for improvement.
- Strategic Recommendations: Based on the analysis, recommendations can be made for optimizing product offerings, managing warranty claims more effectively, and focusing on high-performing stores or regions.
This comprehensive analysis will assist Apple in making data-driven decisions for improving product performance, enhancing customer satisfaction, and optimizing sales strategies globally.

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
    ST.[state],  
    SUM(S.[quantity]) AS [total_unit_sold]
FROM 
    [dbo].[sales] S
JOIN 
    [dbo].[stores] ST ON S.[store_id] = ST.[store_id]
GROUP BY 
    ST.[state]
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
SELECT COUNT(*) AS stores_without_warranty_claims
FROM [dbo].[stores] ST
WHERE NOT EXISTS (
    SELECT 1
    FROM [dbo].[warranty] W
    JOIN [dbo].[sales] S ON W.[sale_id] = S.[sale_id]
    WHERE S.[store_id] = ST.[store_id]
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
    ROUND(COUNT(w.[claim_id]) * 100.0 / COUNT(s.[sale_id]), 2) AS warranty_claim_percentage
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
        SUM(s.[quantity] * p.[price]) AS [total_sales]
    FROM 
        [dbo].[sales] s
    JOIN 
        [dbo].[stores] st ON s.[store_id] = st.[store_id]
    JOIN
        [dbo].[products] p ON s.[product_id] = p.[product_id]
    GROUP BY 
        st.[store_id], YEAR(s.[sale_date])
)
SELECT 
    st.[store_id],
    st.[store_name],
    ys.[year],
    ISNULL(
        (ys.[total_sales] - LAG(ys.[total_sales]) OVER (PARTITION BY ys.[store_id] ORDER BY ys.[year])) 
        * 1.0 / LAG(ys.[total_sales]) OVER (PARTITION BY ys.[store_id] ORDER BY ys.[year]), 0
    ) AS [growth_ratio]
FROM 
    YearlySales ys
JOIN 
    [dbo].[stores] st ON ys.[store_id] = st.[store_id]
ORDER BY 
    ys.[store_id], ys.[year];

```

# 18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
WITH ProductWarrantyData AS (
    SELECT 
        p.[product_id],
        p.[product_name],
        p.[price],
        COUNT(w.[claim_id]) AS [warranty_claims],
        CASE 
            WHEN p.[price] < 500 THEN 'Low Price'
            WHEN p.[price] BETWEEN 500 AND 1000 THEN 'Mid Price'
            WHEN p.[price] > 1000 THEN 'High Price'
        END AS [price_range]
    FROM 
        [dbo].[sales] s
    JOIN 
        [dbo].[products] p ON s.[product_id] = p.[product_id]
    LEFT JOIN 
        [dbo].[warranty] w ON s.[sale_id] = w.[sale_id]
    WHERE 
        w.[claim_date] >= DATEADD(YEAR, -5, GETDATE()) -- Filter for warranty claims in the last 5 years
    GROUP BY 
        p.[product_id], p.[product_name], p.[price]
)
SELECT 
    [price_range],
    CORR([price], [warranty_claims]) AS [price_warranty_correlation]
FROM 
    ProductWarrantyData
GROUP BY 
    [price_range]
ORDER BY 
    [price_range];
```

# 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH ClaimData AS (
    SELECT 
        st.[store_id],
        st.[store_name],
        COUNT(w.[claim_id]) AS [total_claims],
        SUM(CASE WHEN w.[repair_status] = 'Paid Repaired' THEN 1 ELSE 0 END) AS [paid_repaired_claims]
    FROM 
        [dbo].[warranty] w
    JOIN 
        [dbo].[stores] st ON w.[store_id] = st.[store_id]
    GROUP BY 
        st.[store_id], st.[store_name]
)
SELECT TOP 1
    [store_id],
    [store_name],
    [total_claims],
    [paid_repaired_claims],
    ROUND(
        (CAST([paid_repaired_claims] AS FLOAT) / [total_claims]) * 100, 2
    ) AS [paid_repaired_percentage]
FROM 
    ClaimData
ORDER BY 
    [paid_repaired_percentage] DESC;

```

# 20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH MonthlySales AS (
    SELECT 
        st.store_id,
        st.store_name,
        YEAR(s.sale_date) AS year,
        MONTH(s.sale_date) AS month,
        SUM(s.quantity * p.price) AS monthly_sales
    FROM 
        dbo.sales s
    JOIN 
        dbo.stores st ON s.store_id = st.store_id
    JOIN 
        dbo.products p ON s.product_id = p.product_id
    WHERE 
        s.sale_date >= DATEADD(YEAR, -4, GETDATE()) -- Past 4 years
    GROUP BY 
        st.store_id, st.store_name, YEAR(s.sale_date), MONTH(s.sale_date)
)
SELECT 
    ms.store_id,
    ms.store_name,
    ms.year,
    ms.month,
    ms.monthly_sales,
    SUM(ms.monthly_sales) OVER (
        PARTITION BY ms.store_id 
        ORDER BY ms.year, ms.month
    ) AS running_total_sales
FROM 
    MonthlySales ms
ORDER BY 
    ms.store_id, ms.year, ms.month;

```

# 21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
WITH SalesTrends AS (
    SELECT 
        p.product_id,
        p.product_name,
        CASE 
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 0 AND 6 THEN '0-6 months'
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 7 AND 12 THEN '6-12 months'
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 13 AND 18 THEN '12-18 months'
            ELSE '18+ months'
        END AS sales_period,
        SUM(s.quantity * pp.price) AS total_sales_in_period
    FROM dbo.sales s
    JOIN dbo.products p ON s.product_id = p.product_id
    JOIN dbo.product_prices pp ON s.product_id = pp.product_id
    WHERE s.sale_date >= p.launch_date
    GROUP BY 
        p.product_id,
        p.product_name,
        CASE 
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 0 AND 6 THEN '0-6 months'
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 7 AND 12 THEN '6-12 months'
            WHEN DATEDIFF(MONTH, p.launch_date, s.sale_date) BETWEEN 13 AND 18 THEN '12-18 months'
            ELSE '18+ months'
        END
)
SELECT 
    product_id,
    product_name,
    sales_period,
    total_sales_in_period
FROM SalesTrends
ORDER BY 
    product_id,
    CASE sales_period
        WHEN '0-6 months' THEN 1
        WHEN '6-12 months' THEN 2
        WHEN '12-18 months' THEN 3
        ELSE 4
    END;

```
