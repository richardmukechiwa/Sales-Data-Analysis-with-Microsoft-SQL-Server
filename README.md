# Sales-Data-Analysis-Project

# Overview

This project analyzes product sales and revenue data to gain insights into sales trends, product performance, and category-wise revenue distribution.

## Data

**Date:** Date of sale.

**Product ID:** Unique identifier for each product.

**Product Name:** Name of the product.

**Category:** Category the product belongs to (in this case: Electronics, Clothing).

**Units Sold:** Number of units sold on each date.

**Revenue:** Revenue generated from the sale of the product.

## SQL ANALYSIS

-- selecting all the data on the table
```sql
   SELECT  *  FROM   dbo.sales_data; 
```

-- 1. Calculate Total Revenue by Product
```sql

SELECT   product_id,
         product_name,
         SUM(revenue) as Total_Revenue
FROM     dbo.sales_data
GROUP BY product_id,
         product_name
ORDER BY Total_Revenue DESC;
```

-- 2. Determine the Product with the Most Units Sold
```sql
SELECT    product_id,
          product_name,
		  MAX(units_sold) AS most_units_sold
FROM      dbo.sales_data
GROUP BY  product_id,
          product_name
ORDER BY  most_units_sold DESC;
```

-- 3. Identify Top-Selling Product by Revenue for Each Day
```sql
WITH      TOP_SELLING_PRODUCT AS
(
SELECT    DAY(date) as day,
          product_id,
          product_name,
		  revenue,
          ROW_NUMBER()OVER(PARTITION BY DAY(date) ORDER BY revenue DESC) AS top_selling
FROM      dbo.sales_data
GROUP  BY DAY(date),
          product_id,
		  product_name,
		  revenue
)
SELECT    day,
          product_id,
          product_name,
		  revenue,
		  top_selling
FROM      TOP_SELLING_PRODUCT
GROUP BY  day,
          product_id,
          product_name,
		  revenue,
		  top_selling
HAVING    top_selling = 1;
```

-- 4. Percentage Contribution of Each Product to the Total Revenue
```sql
WITH       CTE_Revenue AS 
 
           (
			   SELECT  product_id, 
			           revenue,
			           product_name
					   
			   FROM    dbo.sales_data
			   GROUP BY           product_name,
					              product_id, 
			                      revenue
			            
			) ,
           
		   CTE_Tot_rev AS 

           (
				SELECT   SUM(revenue) as total_rev
				FROM     dbo.sales_data
				
            )
SELECT     product_id,
           product_name,
		   revenue,
		   total_rev,
         (revenue * 100.00)/total_rev AS '% Contribution'
FROM       CTE_Revenue
CROSS JOIN CTE_Tot_rev;
```

--5 Identify Low-Performing Products (Revenue Below Average)
```sql
SELECT     product_id,
           product_name,
		   SUM(revenue) AS total_revenue
FROM       dbo.sales_data
GROUP BY   product_id,
           product_name,
		   revenue
HAVING     SUM(revenue) < (SELECT  AVG(revenue) FROM  dbo.sales_data);
```		   

-- 7. Track Sales Growth (Day-over-Day)
```sql
WITH		CTE_Growth AS
			(
			SELECT     DISTINCT(date) AS daily,
					   SUM(revenue)OVER(PARTITION BY date) AS daily_total
			FROM       dbo.sales_data
			GROUP BY   date,
					   revenue
			),
			CTE_prev_day AS 
			(
			SELECT     daily,
					   daily_total AS revenue,
					   LAG(daily_total, 1) OVER(ORDER BY daily) AS total_previous_day
			FROM       CTE_Growth
			GROUP BY   daily,
					   daily_total
			)
SELECT      daily,
            revenue,
			total_previous_day,
			(revenue - total_previous_day) AS revenue_growth
FROM        CTE_prev_day
GROUP BY    daily,
            revenue,
```			total_previous_day;

-- 8. Best Performing Category by Units Sold
```sql
SELECT      category,
            SUM(units_sold) AS tot_units_sold
FROM        dbo.sales_data
GROUP BY    category;
```
-- 9. Calculate the Average Revenue per Unit for Each Product
```sql
SELECT      product_id,
            product_name,
			SUM(revenue) AS tot_rev,
			SUM(units_sold) AS tot_units,
			SUM(revenue)/ SUM(units_sold) AS Average_Rev_unit
FROM        dbo.sales_data
GROUP BY    product_id,
            product_name;
```
-- 10. Highest Revenue-Generating Day
```sql
SELECT      TOP 1
            date,
            SUM(revenue) AS revenue_day
FROM        dbo.sales_data
GROUP BY    date,
            revenue
ORDER BY    revenue_day DESC;

```
-- 11. Top 3 Products by Revenue in the Electronics Category
```sql
SELECT      TOP 3 
            product_id,
            product_name,
            category,
            SUM(revenue) AS total_revenue
FROM        dbo.sales_data
GROUP BY    product_id,
            product_name,
            category
HAVING      category LIKE 'Electronics'
ORDER BY    total_revenue DESC;
```
-- 12  Compare the average revenue generated per unit sold for the two categories: Clothing and Electronics.
```sql
SELECT       category,
             SUM(units_sold) AS tot_units,
             SUM(revenue) AS total_rev,
			 SUM(revenue) / SUM(units_sold) AS avg_rev_unit
FROM         dbo.sales_data
GROUP BY     category;
```
-- 13. Identify Sales Drop in Clothing Category
```sql
WITH         CTE_Sales_drop AS
             (
             SELECT       date,
						  category,
						  SUM(revenue) AS revenue,
						  SUM(units_sold)  AS units_sold

			 FROM         dbo.sales_data
			 GROUP BY     date,
						  category
			 HAVING       category LIKE 'Clothing'
			 )
SELECT       date,
             revenue,
             units_sold,
			 LEAD(revenue, 1)OVER(ORDER BY  date) AS revenue_trend,
			 LEAD(units_sold, 1)OVER(ORDER BY date) AS units_sold_trend,
			 (revenue - LEAD(revenue, 1)OVER(ORDER BY  date)) AS Rev_diff,
			 (units_sold - LEAD(units_sold, 1)OVER(ORDER BY date)) AS units_sold_diff,
			 CASE  
			       WHEN   (revenue - LEAD(revenue, 1)OVER(ORDER BY  date)) > 0 THEN 'revenue_drop'
			 ELSE  'No_rev_drop' END AS revenue_trend_check,
			 CASE
			       WHEN   (units_sold - LEAD(units_sold, 1)OVER(ORDER BY date)) > 0 THEN 'sales_drop'
			 ELSE   
				  'No_drop_units'  END  AS   sales_status     
FROM         CTE_Sales_drop
GROUP BY     revenue,
             units_sold,
			 date;
```
-- 14. Cumulative Revenue for Each Product by Day
```sql
SELECT        date,
              product_id,
              product_name,
			  revenue,
			  SUM(revenue) OVER(PARTITION BY product_id, product_name ORDER BY date) AS cumulative_revenue
FROM          dbo.sales_data
GROUP BY       date,
              product_id,
              product_name,
			  revenue;
```
			 
             


