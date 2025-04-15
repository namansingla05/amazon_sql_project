---

# **Amazon USA Sales Analysis Project**

### **Difficulty Level: Advanced**

---

## **Project Overview**

I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behavior, product performance, and sales trends using PostgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

The project also focuses on data cleaning, handling null values, and solving real-world business problems using structured queries.

An ERD diagram is included to visually represent the database schema and relationships between tables.

---

![ERD Scratch](https://github.com/namansingla05/amazon_sql_project/blob/main/ERD.png)

## **Database Setup & Design**

### **Schema Structure**

```sql
-- Amazon Project - Advanced SQL 


-- Hi please make sure to follow below hirearary to import the data
-- 1st import to category table
-- 2nd Import to customers
-- 3rd Import to sellers
-- 4th Import to Products
-- 5th Import to orders
-- 6th Import to order_items
-- 7th Import to payments
-- 8th Import to shippings table
-- 9th Import to Inventory Table

DROP TABLE IF EXISTS shippings;
DROP TABLE IF EXISTS payments;
DROP TABLE IF EXISTS inventory;
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS sellers;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS category;


-- category TABLE
CREATE TABLE category
(
category_id	INT PRIMARY KEY,
category_name VARCHAR(20)
);

-- customers TABLE

CREATE TABLE customers
(
customer_id INT PRIMARY KEY,	
first_name	VARCHAR(20),
last_name	VARCHAR(20),
state VARCHAR(20)
);

-- sellers TABLE
CREATE TABLE sellers
(
seller_id INT PRIMARY KEY,
seller_name	VARCHAR(25),
origin VARCHAR(15)
);

-- updating data types
ALTER TABLE sellers
ALTER COLUMN origin TYPE VARCHAR(10)
;

-- products table
CREATE TABLE products
(
product_id INT PRIMARY KEY,	
product_name VARCHAR(50),	
price	FLOAT,
cogs	FLOAT,
category_id INT, -- FK 
CONSTRAINT product_fk_category FOREIGN KEY(category_id) REFERENCES category(category_id)
);

-- orders
CREATE TABLE orders
(
order_id INT PRIMARY KEY, 	
order_date	DATE,
customer_id	INT, -- FK
seller_id INT, -- FK 
order_status VARCHAR(15),
CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
CONSTRAINT orders_fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);

CREATE TABLE order_items
(
order_item_id INT PRIMARY KEY,
order_id INT,	-- FK 
product_id INT, -- FK
quantity INT,	
price_per_unit FLOAT,
CONSTRAINT order_items_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- payment TABLE
CREATE TABLE payments
(
payment_id	
INT PRIMARY KEY,
order_id INT, -- FK 	
payment_date DATE,
payment_status VARCHAR(20),
CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);


CREATE TABLE shippings
(
shipping_id	INT PRIMARY KEY,
order_id	INT, -- FK
shipping_date DATE,	
return_date	 DATE,
shipping_providers	VARCHAR(15),
delivery_status VARCHAR(15),
CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);



CREATE TABLE inventory
(
inventory_id INT PRIMARY KEY,
product_id INT, -- FK
stock INT,
warehouse_id INT,
last_stock_date DATE,
CONSTRAINT inventory_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- End of schemas
```

---

## **Task: Data Cleaning**

I cleaned the dataset by:
- **Removing duplicates**: Duplicates in the customer and order tables were identified and removed.
- **Handling missing values**: Null values in critical fields (e.g., customer address, payment status) were either filled with default values or handled using appropriate methods.

---

## **Handling Null Values**

Null values were handled based on their context:
- **Customer addresses**: Missing addresses were assigned default placeholder values.
- **Payment statuses**: Orders with null payment statuses were categorized as “Pending.”
- **Shipping information**: Null return dates were left as is, as not all shipments are returned.

---

## **Objective**

The primary objective of this project is to showcase SQL proficiency through complex queries that address real-world e-commerce business challenges. The analysis covers various aspects of e-commerce operations, including:
- Customer behavior
- Sales trends
- Inventory management
- Payment and shipping analysis
- Forecasting and product performance
  

## **Identifying Business Problems**

Key business problems identified:
1. Low product availability due to inconsistent restocking.
2. High return rates for specific product categories.
3. Significant delays in shipments and inconsistencies in delivery times.
4. High customer acquisition costs with a low customer retention rate.

---

## **Solving Business Problems**

### Solutions Implemented:
1. Top Selling Products
Query the top 10 products by total sales value.
Challenge: Include product name, total quantity sold, and total sales value.

```sql
-- Creating new column
ALTER TABLE order_items
ADD COLUMN total_sale FLOAT;


SELECT * FROM order_items;

UPDATE order_items
SET total_sale = quantity * price_per_unit;
SELECT * FROM order_items;

SELECT * FROM order_items
ORDER BY quantity DESC;

select distinct order_status from orders;

SELECT 
	oi.product_id,
	p.product_name,
	sum(oi.quantity)  as total_Quantity,
	round(SUM(oi.total_sale)::numeric,2) as total_sale
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON p.product_id = oi.product_id
where o.order_status='Completed'
GROUP BY 1, 2
ORDER BY 4 DESC
LIMIT 10;
```

2. Revenue by Category
Calculate total revenue generated by each product category.
Challenge: Include the percentage contribution of each category to total revenue.

```sql
select c.category_name,
round(SUM(oi.total_sale)::numeric,2) as revenue,
round(round(SUM(oi.total_sale)::numeric,2)*100/
(select round(sum(total_sale)::numeric,2)
from order_items as oi
join 
orders as o on oi.order_id=o.order_id
WHERE o.order_status = 'Completed')::numeric,2) as revenue_percentage
from orders as o
join 
order_items as oi on oi.order_id = o.order_id
JOIN 
products as p ON p.product_id = oi.product_id
join 
Category as c on p.category_id=c.category_id
where o.order_status='Completed'
group by 1;
```

3. Average Order Value (AOV)
Compute the average order value for each customer.
Challenge: Include only customers with more than 5 orders.

```sql
select c.customer_id,
CONCAT(c.first_name, ' ',  c.last_name) as full_name,
round((sum(oi.total_sale)/COUNT(o.order_id))::numeric,2) AS avg_order_value,
COUNT(o.order_id) as total_orders
from order_items as oi
join 
orders as o on oi.order_id=o.order_id
join 
customers as c on o.customer_id=c.customer_id
where o.order_status='Completed'
group by 1,2
having COUNT(DISTINCT o.order_id)>5;
```

4. Monthly Sales Trend
Query monthly total sales over the past year.
Challenge: Display the sales trend, grouping by month, return current_month sale, last month sale!

```sql
SELECT 
	year,
	month,
	total_sale as current_month_sale,
	LAG(total_sale) OVER(ORDER BY year, month) as last_month_sale
FROM ---
(
SELECT 
	EXTRACT(MONTH FROM o.order_date) as month,
	EXTRACT(YEAR FROM o.order_date) as year,
	ROUND(
			SUM(oi.total_sale::numeric)
			,2) as total_sale
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
and o.order_status='Completed'
GROUP BY 1, 2
ORDER BY year, month
) as t1
```


5. Customers with No Purchases
Find customers who have registered but never placed an order.
Challenge: List customer details and the time since their registration.

```sql
--Approach 1
select c.*
from customers as c
left join 
orders as o on c.customer_id=o.customer_id
where o.order_id is null
```
```sql
--Approach 2
SELECT *
	-- reg_date - CURRENT_DATE
FROM customers
WHERE customer_id NOT IN
(SELECT DISTINCT customer_id FROM orders);
```

6. Least-Selling Categories by State
Identify the least-selling product category for each state.
Challenge: Include the total sales for that category within each state.

```sql
with state_category_rank as
(select c.state,ct.category_name,sum(oi.quantity) as total_quantity,
rank() over(partition by c.state order by sum(oi.quantity) asc) as rank
from category as ct
join
products as p on ct.category_id=p.category_id
join
order_items as oi on p.product_id=oi.product_id
join 
orders as o on oi.order_id=o.order_id
join 
customers as c on o.customer_id=c.customer_id
where o.order_status='Completed'
group by 1,2 order by 1) --Skipping order by doesn’t affect results
select * from state_category_rank where rank=1;
```


7. Customer Lifetime Value (CLTV)
Calculate the total value of orders placed by each customer over their lifetime.
Challenge: Rank customers based on their CLTV.

```sql
select c.customer_id,
CONCAT(c.first_name, ' ',  c.last_name) as full_name,
sum(oi.total_sale) as CLTV,
dense_rank() OVER( ORDER BY SUM(total_sale) DESC) as customer_ranking
from order_items as oi
join 
orders as o on oi.order_id=o.order_id
join 
customers as c on o.customer_id=c.customer_id
--where o.order_status='Completed'
group by 1,2 order by 3 desc;
```


8. Inventory Stock Alerts
Query products with stock levels below a certain threshold (e.g., less than 10 units).
Challenge: Include last restock date and warehouse information.

```sql
SELECT 
	i.inventory_id,
	p.product_name,
	i.stock as current_stock_left,
	i.last_stock_date,
	i.warehouse_id
FROM inventory as i
join 
products as p
ON p.product_id = i.product_id
WHERE stock < 10;
```

9. Shipping Delays
Identify orders where the shipping date is later than 3 days after the order date.
Challenge: Include customer, order details, and delivery provider.

```sql
SELECT 
	c.*,
	o.*,
	s.shipping_providers,
s.shipping_date - o.order_date as days_took_to_ship
FROM orders as o
JOIN
customers as c
ON c.customer_id = o.customer_id
JOIN 
shippings as s
ON o.order_id = s.order_id
WHERE s.shipping_date - o.order_date > 3;
```

10. Payment Success Rate 
Calculate the percentage of successful payments across all orders.
Challenge: Include breakdowns by payment status (e.g., failed, pending).

```sql
SELECT 
	p.payment_status,
	COUNT(*) as total_cnt,
	round(COUNT(*)/(SELECT COUNT(*) FROM payments)::numeric * 100::numeric,2)
FROM orders as o
JOIN
payments as p
ON o.order_id = p.order_id
GROUP BY 1
```

11. Top Performing Sellers
Find the top 5 sellers based on total sales value.
Challenge: Include both successful and failed orders, and display their percentage of successful orders.

```sql
WITH top_sellers
AS
(SELECT 
	s.seller_id,
	s.seller_name,
	SUM(oi.total_sale) as total_sale
FROM orders as o
JOIN
sellers as s
ON o.seller_id = s.seller_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5
),

sellers_reports
AS
(SELECT 
	o.seller_id,
	ts.seller_name,
	o.order_status,
	COUNT(*) as total_orders
FROM orders as o
JOIN 
top_sellers as ts
ON ts.seller_id = o.seller_id
WHERE 
	o.order_status NOT IN ('Inprogress', 'Returned')
	
GROUP BY 1, 2, 3
)
SELECT 
	seller_id,
	seller_name,
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END) as Completed_orders,
	SUM(CASE WHEN order_status = 'Cancelled' THEN total_orders ELSE 0 END) as Cancelled_orders,
	SUM(total_orders) as total_orders,
	Round(SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END)/
	SUM(total_orders) * 100::numeric,2) as successful_orders_percentage
	
FROM sellers_reports
GROUP BY 1, 2;
```


12. Product Profit Margin
Calculate the profit margin for each product (difference between price and cost of goods sold).
Challenge: Rank products by their profit margin, showing highest to lowest.
*/


```sql
SELECT 
	oi.product_id,
	p.product_name,
	round(SUM(total_sale - (p.cogs * oi.quantity))::numeric,2) as profit,
	SUM(total_sale - (p.cogs * oi.quantity))*100/sum(total_sale) as profit_margin,
	dense_rank() over(order by 
	SUM(total_sale - (p.cogs * oi.quantity))*100/sum(total_sale) desc)
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON p.product_id = oi.product_id
where o.order_status='Completed'
GROUP BY 1, 2;
```

13. Most Returned Products
Query the top 10 products by the number of returns.
Challenge: Display the return rate as a percentage of total units sold for each product.

```sql
SELECT oi.product_id,p.product_name,
COUNT(*) as total_unit_sold,
SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_returned,
Round(SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END)::numeric/COUNT(*)::numeric * 100,2) as return_percentage
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON p.product_id = oi.product_id
GROUP BY 1, 2
ORDER BY 5 DESC
LIMIT 10;
```

14. Inactive Sellers
Identify sellers who haven’t made any sales in the last 6 months.
Challenge: Show the last sale date and total sales from those sellers.

```sql
WITH inactive_sellers AS(
SELECT * FROM sellers
WHERE seller_id NOT IN(SELECT seller_id 
FROM orders WHERE order_date >= CURRENT_DATE - INTERVAL '6 months')
),
last_orders AS(
SELECT o.seller_id,
MAX(o.order_date) AS last_sale_date FROM orders o
JOIN 
inactive_sellers s ON s.seller_id = o.seller_id
GROUP BY 1
)

SELECT lo.seller_id,lo.last_sale_date,
SUM(oi.total_sale) AS last_sale_amount
FROM last_orders lo
JOIN 
orders o ON o.seller_id = lo.seller_id AND o.order_date = lo.last_sale_date
JOIN 
order_items oi ON oi.order_id = o.order_id
GROUP BY 1,2;
```


15. IDENTITY customers into returning or new
if the customer has done more than 5 return categorize them as returning otherwise new
Challenge: List customers id, name, total orders, total returns

```sql
with cx_returns as
(select c.customer_id,
CONCAT(c.first_name, ' ',  c.last_name) as full_name,
count(o.order_id) as Total_orders,
sum(case when o.order_status='Returned' then 1 else 0 End) as Total_returns
from customers as c
join 
orders as o on c.customer_id=o.customer_id
group by 1,2)

select *,CASE WHEN Total_returns > 5 THEN 'Returning_customers' ELSE 'New'
END as cx_category from cx_returns order by total_returns desc;
```


16. Top 5 Customers by Orders in Each State
Identify the top 5 customers with the highest number of orders for each state.
Challenge: Include the number of orders and total sales for each customer.
```sql
with top_customers as
(select c.customer_id,
CONCAT(c.first_name, ' ',  c.last_name) as full_name,c.state,
count(o.order_id) as total_orders,
round(sum(oi.total_sale)::numeric,2) as total_sales,
dense_rank() over(partition by c.state order by count(o.order_id) desc) as rank
from customers as c
join 
orders as o on c.customer_id=o.customer_id
join 
order_items as oi on o.order_id=oi.order_id
group by 1,2,3)

select * from top_customers where rank<=5;
```

17. Revenue by Shipping Provider
Calculate the total revenue handled by each shipping provider.
Challenge: Include the total number of orders handled and the average delivery time for each provider.

```sql
select s.shipping_providers,
count(o.order_id) as orders_handled,
round(sum(oi.total_sale)::numeric,2) as total_sales,
round(avg(s.shipping_date - o.order_date)::numeric,4) as average_days
from shippings as s
join 
orders as o on s.order_id=o.order_id
join 
order_items as oi on o.order_id=oi.order_id
group by 1;
```

18. Top 10 product with highest decreasing revenue ratio compare to last year(2022) and current_year(2023)
Challenge: Return product_id, product_name, category_name, 2022 revenue and 2023 revenue decrease ratio at end Round the result
Note: Decrease ratio = cr-ls/ls* 100 (cs = current_year ls=last_year)

```sql
with sale_per_year as
(select p.product_id,p.product_name,c.category_name,
Extract(year from o.order_date) as year,
round(SUM(oi.total_sale)::numeric,2) as total_sale
FROM orders as o
JOIN
order_items as oi ON oi.order_id = o.order_id
JOIN 
products as p ON p.product_id = oi.product_id
join 
category as c on p.category_id=c.category_id
where o.order_status='Completed'
GROUP BY 1,2,3,4
ORDER BY 2,4),

sale_per_year_22_23 as
(select product_id,product_name,category_name,
sum(case when year=2022 then total_sale else 0 end) as revenue_2022,
sum(case when year=2023 then total_sale else 0 end) as revenue_2023
from sale_per_year group by 1,2,3 order by 2)

SELECT *,
ROUND((revenue_2023 - revenue_2022) * 100.0 / revenue_2022, 2) AS growth_percentage
FROM sale_per_year_22_23
where revenue_2022>revenue_2023
order by growth_percentage desc limit 10;
```


19. Final Task: Stored Procedure
Create a stored procedure that, when a product is sold, performs the following actions:
Inserts a new sales record into the orders and order_items tables.
Updates the inventory table to reduce the stock based on the product and quantity purchased.
The procedure should ensure that the stock is adjusted immediately after recording the sale.

```SQL
CREATE OR REPLACE PROCEDURE add_sales
(
p_order_id INT,
p_customer_id INT,
p_seller_id INT,
p_order_item_id INT,
p_product_id INT,
p_quantity INT
)
LANGUAGE plpgsql
AS $$

DECLARE 
-- all variable
v_count INT;
v_price FLOAT;
v_product VARCHAR(50);

BEGIN
-- Fetching product name and price based p id entered
	SELECT 
		price, product_name
		INTO
		v_price, v_product
	FROM products
	WHERE product_id = p_product_id;
	
-- checking stock and product availability in inventory	
	SELECT 
		COUNT(*) 
		INTO
		v_count
	FROM inventory
	WHERE 
		product_id = p_product_id
		AND 
		stock >= p_quantity;
		
	IF v_count > 0 THEN
	-- add into orders and order_items table
	-- update inventory
		INSERT INTO orders(order_id, order_date, customer_id, seller_id)
		VALUES
		(p_order_id, CURRENT_DATE, p_customer_id, p_seller_id);

		-- adding into order list
		INSERT INTO order_items(order_item_id, order_id, product_id, quantity, price_per_unit, total_sale)
		VALUES
		(p_order_item_id, p_order_id, p_product_id, p_quantity, v_price, v_price*p_quantity);

		--updating inventory
		UPDATE inventory
		SET stock = stock - p_quantity
		WHERE product_id = p_product_id;
		
		RAISE NOTICE 'Thank you product: % sale has been added also inventory stock updates',v_product; 

	ELSE
		RAISE NOTICE 'Thank you for for your info the product: % is not available', v_product;

	END IF;


END;
$$
```



**Testing Store Procedure**
call add_sales
(
25005, 2, 5, 25004, 1, 14
);

---

## **Learning Outcomes**

This project enabled me to:
- Design and implement a normalized database schema.
- Clean and preprocess real-world datasets for analysis.
- Use advanced SQL techniques, including window functions, subqueries, and joins.
- Conduct in-depth business analysis using SQL.
- Optimize query performance and handle large datasets efficiently.

---

## **Conclusion**

This advanced SQL project successfully demonstrates my ability to solve real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.

By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.

---
