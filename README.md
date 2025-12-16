# SQL-Based Analytical Study of ZEPTO Products


<img zpeto" />



### Overview

This project presents an in-depth SQL-driven analysis of ZEPTO’s product catalog dataset. The objective is to extract meaningful insights into product pricing, stock availability, discount patterns, inventory management, and category-wise performance. By leveraging advanced SQL queries in pgAdmin, the project addresses real business problems and provides actionable insights for e-commerce operations.

The analysis explores various dimensions such as pricing trends, discount analysis, stock availability, inventory valuation, product categorization, and revenue contributions.

### Objectives

Examine product distribution and availability across categories.

Identify high-value and premium products and analyze discounts.

Explore inventory weight, stock status, and slow-moving items.

Calculate category-wise revenue, inventory value, and revenue contribution percentages.

Detect pricing anomalies and underpriced/overpriced products.

Categorize products based on weight, price, or discount for better decision-making.

## Schema
CREATE TABLE zepto (
  sku_id SERIAL PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp NUMERIC(8,2),
  discountPercent NUMERIC(5,2),
  availableQuantity INTEGER,
  discountedSellingPrice NUMERIC(8,2),
  weightInGms INTEGER,
  outOfStock BOOLEAN,
  quantity INTEGER
);

--data exploration

--count of rows
select count(*) from zepto;

--sample data
SELECT * FROM zepto
LIMIT 10;

--null values
SELECT * FROM zepto
WHERE name IS NULL
OR
category IS NULL
OR
mrp IS NULL
OR
discountPercent IS NULL
OR
discountedSellingPrice IS NULL
OR
weightInGms IS NULL
OR
availableQuantity IS NULL
OR
outOfStock IS NULL
OR
quantity IS NULL;

--different product categories
SELECT DISTINCT category
FROM zepto
ORDER BY category;

--products in stock vs out of stock
SELECT outOfStock, COUNT(sku_id)
FROM zepto
GROUP BY outOfStock;

--product names present multiple times
SELECT name, COUNT(sku_id) AS "Number of SKUs"
FROM zepto
GROUP BY name
HAVING count(sku_id) > 1
ORDER BY count(sku_id) DESC;

--data cleaning

--products with price = 0
SELECT * FROM zepto
WHERE mrp = 0 OR discountedSellingPrice = 0;

DELETE FROM zepto
WHERE mrp = 0;

--convert paise to rupees
UPDATE zepto
SET mrp = mrp / 100.0,
discountedSellingPrice = discountedSellingPrice / 100.0;

SELECT mrp, discountedSellingPrice FROM zepto;

--data analysis

-- Q1. Find the top 10 best-value products based on the discount percentage.

SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;

--Q2.What are the Products with High MRP but Out of Stock

SELECT DISTINCT name,mrp
FROM zepto
WHERE outOfStock = TRUE and mrp > 300
ORDER BY mrp DESC;

--Q3.Calculate Estimated Revenue for each category

SELECT category,
SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue;

-- Q4. Find all products where MRP is greater than ₹500 and discount is less than 10%.

SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;

-- Q5. Identify the top 5 categories offering the highest average discount percentage.

SELECT category,
ROUND(AVG(discountPercent),2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;

-- Q6. Find the price per gram for products above 100g and sort by best value.

SELECT DISTINCT name, weightInGms, discountedSellingPrice,
ROUND(discountedSellingPrice/weightInGms,2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram;

--Q7.Group the products into categories like Low, Medium, Bulk.
SELECT DISTINCT name, weightInGms,
CASE WHEN weightInGms < 1000 THEN 'Low'
	WHEN weightInGms < 5000 THEN 'Medium'
	ELSE 'Bulk'
	END AS weight_category
FROM zepto;

--Q8.What is the Total Inventory Weight Per Category 
SELECT category,
SUM(weightInGms * availableQuantity) AS total_weight
FROM zepto
GROUP BY category
ORDER BY total_weight;

--Q9. Which products generate the highest inventory value?

SELECT 
    name,
    category,
    mrp * availableQuantity AS inventory_value
FROM zepto
ORDER BY inventory_value DESC
LIMIT 10;

--Q10. Category-wise Stock Availability Status

SELECT 
    category,
    COUNT(*) FILTER (WHERE outOfStock = FALSE) AS in_stock,
    COUNT(*) FILTER (WHERE outOfStock = TRUE) AS out_of_stock
FROM zepto
GROUP BY category;

--Q11. Identify Slow-Moving Inventory

SELECT 
    name,
    availableQuantity,
    quantity
FROM zepto
WHERE availableQuantity > 5 AND quantity < 2
ORDER BY availableQuantity DESC;


--Q12. Products with Discount Higher Than Category Average

SELECT z.name, z.category, z.discountPercent
FROM zepto z
JOIN (
    SELECT category, AVG(discountPercent) AS avg_discount
    FROM zepto
    GROUP BY category
) c
ON z.category = c.category
WHERE z.discountPercent > c.avg_discount
ORDER BY z.discountPercent DESC;

--Q13. Rank Products by Discount Within Each Category

SELECT 
    name,
    category,
    discountPercent,
    RANK() OVER (PARTITION BY category ORDER BY discountPercent DESC) AS discount_rank
FROM zepto;

--Q14. Revenue Contribution Percentage per Product

SELECT 
    name,
    ROUND(
        (discountedSellingPrice * availableQuantity) /
        SUM(discountedSellingPrice * availableQuantity) OVER () * 100, 2
    ) AS revenue_percentage
FROM zepto
ORDER BY revenue_percentage DESC;

--Q15. Identify Premium Products with Low Discounts

SELECT 
    name,
    mrp,
    discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 5
ORDER BY mrp DESC;


--Q16. Category-wise Average Price per Gram

SELECT 
    category,
    ROUND(AVG(discountedSellingPrice / NULLIF(weightInGms,0)), 2) AS avg_price_per_gram
FROM zepto
GROUP BY category
ORDER BY avg_price_per_gram;

--Q17. Detect Price Anomalies (Selling Above MRP)

SELECT *
FROM zepto
WHERE discountedSellingPrice > mrp;

--Q18. Identify Products with Zero Discount

SELECT name, mrp
FROM zepto
WHERE discountPercent = 0
ORDER BY mrp DESC;

--Q19. Category-wise Revenue Share Percentage
```sql
SELECT 
    category,
    ROUND(
        SUM(discountedSellingPrice * availableQuantity) /
        SUM(SUM(discountedSellingPrice * availableQuantity)) OVER () * 100, 2
    ) AS revenue_share_percent
FROM zepto
GROUP BY category
ORDER BY revenue_share_percent DESC;
```

--Q20. Identify Underpriced Heavy Products
```sql
SELECT 
    name,
    weightInGms,
    discountedSellingPrice
FROM zepto
WHERE weightInGms > 1000 AND discountedSellingPrice < 200
ORDER BY weightInGms DESC;
```


