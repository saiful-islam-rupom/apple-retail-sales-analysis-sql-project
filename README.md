![Apple Image](Photo/apple_image.png)
# Apple Retail Sales Analysis SQL Project

## Project Overview  
This project highlights advanced SQL techniques through the analysis of over half million sales records from a global retail dataset, covering products, category, stores, sales, and warranty claims. It solves more than 20 real-world business problems such as identifying best and worst-selling products by location, spotting product quality issues via warranty trends, analyzing yearly sales growth, evaluating store performance, and exploring how pricing affects warranty claims etc. In this project, I utilize the PostgreSQL DBMS through the pgAdmin 4 GUI to address and solve various key business challenges.

## Project Objectives  
This project analyzes Apple retail sales and warranty data to solve key business challenges using advanced SQL queries. It provides data-driven insights to optimize sales, improve product quality, and enhance customer service.

### Key Business Problems Solved:  
- **Sales & Inventory Optimization** – Identifies top-selling products, sales trends, and high-performing stores.  
- **Store & Business Growth Analysis** – Tracks year-over-year sales growth and regional store performance.  
- **Product Quality & Warranty Insights** – Detects high-claim products, early failures, and defect trends.  
- **Customer Service Improvement** – Analyzes warranty response times and repair costs to enhance service.  

## About the Datasets  
This project utilizes a large-scale Apple retail dataset containing over half million sales rows. Moreover, separate datasets of products, category, stores, and warranty claim records also included in this project which can be combined together and extract different business insights as well as sales performance, customer purchases, and product reliability across different regions etc. from these datasets.

## Entity Relationship Diagram (ERD): 
![ER Diagram](Photo/ERD.png)
### Entities & Attributes: 
The project includes five main tables: 
- **products** (product_id, product_name, category_id, launch_date, price)  
- **category** (category_id, category_name)  
- **stores** (store_id, store_name, city, country)  
- **sales** (sale_id, sale_date, store_id, product_id, quantity)  
- **warranty** (claim_id, claim_date, sale_id, repair_status)  

## Key Business Questions & Solutions using SQL
### Here, Business questions organized by complexity: Basic to Medium (Problem statement 1–10), Intermediate (Problem statement 11–15), and Advanced (Problem statement 16–21)

#### **Problem statement 1:** Identify the number of stores available in each country.
#### **Solution:**
```sql
SELECT
	country,
	COUNT(*) AS total_stores
FROM stores
GROUP BY country
ORDER BY total_stores DESC;
```
![Solution1](Photo/ERD.png)

#### **Problem statement 2:** Determine the total units sold by each store.
#### **Solution:**
```sql
SELECT
	st.store_id,
	st.store_name,
	SUM(sa.quantity) AS total_units_sold
FROM sales sa
JOIN stores st 
	ON sa.store_id = st.store_id
GROUP BY st.store_id
ORDER BY total_units_sold DESC;
```
![Solution2](Photo/ERD.png)

#### **Problem statement 3:** Determine the number of sales transactions that occurred in December 2023.
#### **Solution:**
```sql
SELECT COUNT(*) AS dec_2023_total_sales
FROM sales
WHERE sale_date >= '2023-12-01' AND sale_date <= '2023-12-31';
```
![Solution3](Photo/ERD.png)

#### **Problem statement 4:** Identify the number of stores with no recorded warranty claims.
#### **Solution:**
```sql
SELECT COUNT(*)
FROM stores st
WHERE st.store_id NOT IN (
	    				SELECT DISTINCT sa.store_id FROM sales sa
						RIGHT JOIN warranty w 
							ON sa.sale_id = w.sale_id
							);
```
![Solution4](Photo/ERD.png)

#### **Problem statement 5:** Determine the percentage of warranty claims categorized as 'Warranty Void'.
#### **Solution:**
```sql
SELECT 
	ROUND((((SELECT COUNT(*) FROM warranty WHERE repair_status = 'Warranty Void')* 100.0)/(SELECT COUNT(*) FROM warranty)),2)
	AS warranty_void_percentage;
```
![Solution5](Photo/ERD.png)

#### **Problem statement 6:** Determine the store with the highest total units sold in 2024.
#### **Solution:**
```sql
SELECT 
	st.store_id, 
	st.store_name, 
	SUM(sa.quantity) AS units_sold 
FROM sales sa
JOIN stores st
	ON sa.store_id = st.store_id
WHERE sa.sale_date >= '2024-01-01' AND sa.sale_date <= '2024-12-31' -- Year 2024
GROUP BY st.store_id, st.store_name
ORDER BY units_sold DESC 
LIMIT 1;
```
![Solution6](Photo/ERD.png)

#### **Problem statement 7:** Calculate the total quantity of each product sold between 2020 and 2024.
#### **Solution:**
```sql
SELECT 
	p.product_id, 
	p.product_name, 
	SUM(sa.quantity) AS total_units_sold 
FROM sales sa
JOIN products p
	ON sa.product_id = p.product_id
WHERE sa.sale_date >= '2020-01-01' AND sa.sale_date <= '2024-12-31' -- Year(2020-2024)
GROUP BY p.product_id, p.product_name
ORDER BY total_units_sold DESC;
```
![Solution7](Photo/ERD.png)

#### **Problem statement 8:** Calculate the average price of products within each category.
#### **Solution:**
```sql
SELECT 
	c.category_id, 
	c.category_name, 
	AVG(p.price) AS average_price
FROM category c
JOIN products p
	ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY average_price DESC;
```
![Solution8](Photo/ERD.png)

#### **Problem statement 9:** What is the total number of warranty claims filed in 2020, 2021, and 2024 combined?
#### **Solution:**
```sql
SELECT 
    COUNT(*) AS number_of_warranty_claim
FROM warranty
WHERE EXTRACT(YEAR FROM claim_date) IN (2020, 2021, 2024);
```
![Solution9](Photo/ERD.png)

#### **Problem statement 10:** For each store, identify the best-selling day of the week based on highest quantity of units sold.
#### **Solution:**
```sql
SELECT 
	t.store_id,
	st.store_name,
	t.day_name,
	t.total_units_sold
FROM(
	SELECT 
	store_id,  
	TO_CHAR(sale_date, 'Day') as day_name,
	SUM(quantity) AS total_units_sold,
	DENSE_RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS rank
	FROM sales
	GROUP BY store_id, day_name
	) t
JOIN stores st
	ON t.store_id = st.store_id
WHERE rank = 1;
```
![Solution10](Photo/ERD.png)

#### **Problem statement 11:** Identify the least selling product in each country for each year based on total units sold.
#### **Solution:**
```sql
WITH product_ranking
AS
(
SELECT
st.country,
EXTRACT(YEAR FROM sa.sale_date) AS year,
p.product_name,
SUM(sa.quantity) AS quantity,
DENSE_RANK() OVER(PARTITION BY st.country,EXTRACT(YEAR FROM sa.sale_date) ORDER BY SUM(sa.quantity) ASC) AS rank  
FROM products p
JOIN sales sa
	ON p.product_id = sa.product_id
JOIN stores st
	ON sa.store_id = st.store_id
GROUP BY st.country, year, p.product_name
)
SELECT * FROM product_ranking
WHERE rank = 1;
```
![Solution11](Photo/ERD.png)

#### **Problem statement 12:** Determine the number of warranty claims filed within 180 days of the corresponding product sale.
#### **Solution:**
```sql
SELECT COUNT(*) FROM sales sa
RIGHT JOIN warranty w
	ON sa.sale_id = w.sale_id
WHERE (w.claim_date - sa.sale_date) <= 180;
```
![Solution12](Photo/ERD.png)

#### **Problem statement 13:** For products launched within the last two years, calculate the total units sold and the number of those units with associated warranty claims.
#### **Solution:**
```sql
SELECT 
	p.product_id,
	p.product_name,
	COUNT(sa.product_id) AS total_units_sold,
	COUNT(w.claim_id) AS number_of_warranty_claimed
FROM sales sa
LEFT JOIN warranty w
	ON w.sale_id = sa.sale_id
LEFT JOIN products p
	ON sa.product_id = p.product_id
WHERE CURRENT_DATE - p.launch_date <= 730 -- 2 years
GROUP BY p.product_id, p.product_name;
```
![Solution13](Photo/ERD.png)

#### **Problem statement 14:** Identify the months within the past three years during which sales in the USA exceeded 2500 units.
#### **Solution:**
```sql
SELECT 
	TO_CHAR(sa.sale_date, 'MM-YYYY') as month,
	SUM(sa.quantity) AS total_units_sold
FROM sales sa
JOIN stores st
	ON sa.store_id = st.store_id
WHERE st.country = 'USA' AND sa.sale_date >= CURRENT_DATE - INTERVAL '3 years'
GROUP BY month
HAVING SUM(sa.quantity) > 2500;
```
![Solution14](Photo/ERD.png)

#### **Problem statement 15:** Identify the product category along with the number of warranty claims filed in the last two years.
#### **Solution:**
```sql
SELECT 
	c.category_id,
	c.category_name,
	COUNT(w.claim_id) AS total_warranty_claims
FROM warranty w
JOIN sales sa
	ON w.sale_id = sa.sale_id
JOIN products p
	ON sa.product_id = p.product_id
JOIN category c
	ON p.category_id = c.category_id
WHERE w.claim_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY c.category_id, c.category_name
ORDER BY total_warranty_claims DESC;
```
![Solution15](Photo/ERD.png)

#### **Problem statement 16:** Calculate the percentage of chance of receiving warranty claims after each purchase for each country.
#### **Solution:**
```sql
SELECT 
	st.country,
	COUNT(sa.sale_id) AS total_sales,
	COUNT(w.claim_id) AS total_claims,
	ROUND((COUNT(w.claim_id) * 100.0 / COUNT(sa.sale_id)),2) AS warranty_claim_percentage
FROM sales sa
LEFT JOIN warranty w
	ON sa.sale_id = w.sale_id
LEFT JOIN stores st
	ON sa.store_id = st.store_id
GROUP BY st.country
ORDER BY warranty_claim_percentage DESC;
```
![Solution16](Photo/ERD.png)

#### **Problem statement 17:** Analyze the year-by-year growth ratio for each store.
#### **Solution:**
```sql
SELECT 
	store_id,
	store_name,
	year,
	SUM(final_price) AS total_sales,
	ROUND((((SUM(final_price) - LAG(SUM(final_price)) OVER (PARTITION BY store_id ORDER BY year))*100)
	/LAG(SUM(final_price)) OVER (PARTITION BY store_id ORDER BY year))::NUMERIC,3) AS growth_percentage
FROM 
(
SELECT 
	st.store_id AS store_id,
	st.store_name AS store_name,
	EXTRACT(YEAR FROM sa.sale_date) AS year,
	(sa.quantity * p.price) AS final_price
FROM stores st
JOIN sales sa
	ON st.store_id = sa.store_id
JOIN products p
	ON sa.product_id = p.product_id
)
GROUP BY store_id, store_name, year;
```
![Solution17](Photo/ERD.png)

#### **Problem statement 18:** Calculate the relationship between product price and warranty claims for products sold in the last three years, segmented by price range.
#### **Solution:**
```sql
SELECT 
	CASE
		WHEN p.price < 500 THEN 'Affordable(1$-499$)'
		WHEN p.price BETWEEN 500 AND 999 THEN 'Moderate(500$-999$)'
		ELSE 'Premium(999$+)'
	END AS price_segment,
	COUNT(sa.sale_id) AS number_of_total_sales,
	COUNT(w.claim_id) AS number_of_total_claims,
	ROUND((((COUNT(w.claim_id))*100.0)/COUNT(sa.sale_id)),2) AS percentage_of_claims
FROM warranty w
RIGHT JOIN sales sa
	ON w.sale_id = sa.sale_id
LEFT JOIN products p
	ON p.product_id = sa.product_id
WHERE sa.sale_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY price_segment;
```
![Solution18](Photo/ERD.png)

#### **Problem statement 19:** Identify the store with the highest percentage of 'Paid Repaired' claims relative to the total number of claims filed.
#### **Solution:**
```sql
SELECT 
	st.store_id,
	st.store_name,
	COUNT(w.claim_id) AS total_claims,
	COUNT(CASE WHEN w.repair_status = 'Paid Repaired' THEN 1 END) AS claims_with_paid_repaired,
    ROUND(COALESCE(
		(COUNT(CASE WHEN w.repair_status = 'Paid Repaired' THEN 1 END) * 100.0) / NULLIF(COUNT(w.claim_id), 0),
        0),2) AS percentage_of_paid_repaired
FROM stores st
LEFT JOIN sales sa
	ON st.store_id = sa.store_id
LEFT JOIN warranty w
	ON sa.sale_id = w.sale_id
GROUP BY st.store_id, st.store_name
ORDER BY percentage_of_paid_repaired DESC;
```
![Solution19](Photo/ERD.png)

#### **Problem statement 20:** Calculate the 'monthly running total' and the 'rolling average of the last 3 months' of sales for each store.
#### **Solution:**
```sql
SELECT 
	st.store_id,
	st.store_name,
	TO_CHAR(DATE_TRUNC('month', sale_date), 'YYYY-MM') AS month,
	SUM(p.price * sa.quantity) as monthly_total_revenue,
	SUM(SUM(p.price * sa.quantity)) OVER 
		(PARTITION BY st.store_id ORDER BY TO_CHAR(DATE_TRUNC('month', sa.sale_date), 'YYYY-MM')
		ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    	) AS monthly_running_total,
	ROUND(AVG(SUM(p.price * sa.quantity)) OVER (
        PARTITION BY st.store_id
        ORDER BY TO_CHAR(DATE_TRUNC('month', sale_date), 'YYYY-MM')
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    	)::NUMERIC,2) AS rolling_avg_last_3_months
FROM stores st
LEFT JOIN sales sa
	ON st.store_id = sa.store_id
LEFT JOIN products p
	ON sa.product_id = p.product_id
GROUP BY st.store_id, st.store_name, month;
```
![Solution20](Photo/ERD.png)

#### **Problem statement 21:** Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
#### **Solution:**
```sql
SELECT 
	p.product_id,
	p.product_name,
	CASE
        WHEN AGE(sa.sale_date, p.launch_date) < INTERVAL '6 months' THEN '0-6 months'
        WHEN AGE(sa.sale_date, p.launch_date) >= INTERVAL '6 months' AND AGE(sa.sale_date, p.launch_date) < INTERVAL '12 months' THEN '6-12 months'
        WHEN AGE(sa.sale_date, p.launch_date) >= INTERVAL '12 months' AND AGE(sa.sale_date, p.launch_date) < INTERVAL '18 months' THEN '12-18 months'
        ELSE '18+ months'
    END AS sales_period_from_launch_date,
	SUM(sa.quantity) as total_units_sold
FROM sales as sa
JOIN products as p
	ON sa.product_id = p.product_id
GROUP BY 1, 3
ORDER BY 1 ASC, 4 DESC;
```
![Solution21](Photo/ERD.png)

## Skills highlighted in this Project 
This project showcases advanced SQL skills applied to business analytics, including data extraction, transformation, and optimization. Key techniques used include complex joins, aggregation, window functions, and date/time analysis. The project focuses on evaluating sales performance, customer behavior, product segmentation, and profitability. It also supports business decision-making through trend and correlation analysis while ensuring query efficiency with optimization strategies.

## Conclusion
This project highlights how SQL can drive data-informed decisions in a retail setting by analyzing over half million rows of sales and warranty data. Key insights include identifying top-selling products, tracking seasonal and regional sales trends, evaluating product quality through warranty claims, and assessing customer satisfaction based on service metrics. These findings support strategic improvements in sales, inventory management, product development, and customer service, ultimately boosting profitability and customer loyalty.
