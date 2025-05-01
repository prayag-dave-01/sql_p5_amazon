# Amazon USA Sales Analysis Project

## Project Overview

**Project Title**: Amazon USA Sales Analysis  
**Level**: Advanced  
**Database**: `amazon_db`

---

![Amazon_project](https://github.com/prayag-dave-01/sql_p5_amazon/blob/main/Amazon_2024.png?raw=true)

This is a comprehensive SQL project based on an e-commerce platform - Amazon, where I analyzed a dataset of over 20,000 sales records to derive actionable business insights. The project involves solving 70 real-world business problems, including 20 advanced-level challenges and 54 basic to intermediate level scenarios.

I extensively queried the data to explore customer behavior, product performance, sales trends, and inventory management. The project reflects hands-on experience in data cleaning, handling null values, and solving analytical and operational problems using structured queries.

**Key business problems tackled include:**
- Revenue analysis across categories and regions.
- Customer segmentation based on purchasing patterns.
- Product performance tracking and optimization.
- Inventory and stock level insights for logistics

**SQL concepts and clauses used extensively throughout the project:**

- Data retrieval and filtering: `SELECT`, `WHERE`, `DISTINCT`, `IN`, `LIKE`, `BETWEEN`  
- Sorting and limiting: `ORDER BY`, `LIMIT`
- Aggregation and grouping: `GROUP BY`, `HAVING`, along with functions like `SUM`, `AVG`, `COUNT`, `MAX`, `MIN`
- Joins and set operations: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`
- Subqueries and CTEs: scalar and `correlated subqueries`, `WITH clause`
- Window functions: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LEAD()`, `LAG()`, `NTILE()`
- Conditional logic: `CASE`, `IF`, `nested conditions`
- Data manipulation and schema design: `INSERT`, `UPDATE`, `DELETE`, `CREATE TABLE`, `ALTER TABLE`, along with constraints such as `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`

An ERD diagram is included to visually represent the database schema and relationships between tables.

---

![ERD Scratch](https://github.com/prayag-dave-01/sql_p5_amazon/blob/main/Amazon%20-%20ERD.png?raw=true)

## **Database Setup & Design**

### **Schema Structure**

```sql
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
  state VARCHAR(20),
  address VARCHAR(5) DEFAULT ('xxxx')
);

-- sellers TABLE
CREATE TABLE sellers
(
  seller_id INT PRIMARY KEY,
  seller_name	VARCHAR(25),
  origin VARCHAR(15)
);

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

### **20 Advanced-Level challenges**

### Solutions Implemented:
Task 1. Top Selling Products
Query the top 10 products by total sales value.
Challenge: Include product name, total quantity sold, and total sales value.

```sql
SELECT 
	oi.product_id,
	p.product_name,
	SUM(oi.total_sale) as total_sale,
	COUNT(o.order_id)  as total_orders
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON p.product_id = oi.product_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 10;
```

Task 2. Revenue by Category
Calculate total revenue generated by each product category.
Challenge: Include the percentage contribution of each category to total revenue.

```sql
SELECT 
	p.category_id,
	c.category_name,
	SUM(oi.total_sale) as total_sale,
	SUM(oi.total_sale)/
					(SELECT SUM(total_sale) FROM order_items) 
					* 100
	as contribution
FROM order_items as oi
JOIN
products as p
ON p.product_id = oi.product_id
LEFT JOIN category as c
ON c.category_id = p.category_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```

Task 3. Average Order Value (AOV)
Compute the average order value for each customer.
Challenge: Include only customers with more than 5 orders.

```sql
SELECT 
	c.customer_id,
	CONCAT(c.first_name, ' ',  c.last_name) as full_name,
	SUM(total_sale)/COUNT(o.order_id) as AOV,
	COUNT(o.order_id) as total_orders --- filter
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
HAVING  COUNT(o.order_id) > 5;
```

Task 4. Monthly Sales Trend
Query monthly total sales over the past year.
Challenge: Display the sales trend, grouping by month, return current_month sale, last month sale!

```sql
SELECT 
     year,
     month,
     total_sale as current_month_sale,
     LAG(total_sale, 1) OVER(ORDER BY year, month) as last_month_sale
FROM ---
(
SELECT 
     EXTRACT(MONTH FROM o.order_date) as month,
     EXTRACT(YEAR FROM o.order_date) as year,
     ROUND(SUM(oi.total_sale::numeric), 2) as total_sale
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY 1, 2
ORDER BY year, month
) as t1;
```

Task 5. Customers with No Purchases
Find customers who have registered but never placed an order.
Challenge: List customer details and the time since their registration.

```sql
-- Approach 1
SELECT *
FROM customers
WHERE
    customer_id NOT IN (SELECT
                        DISTINCT customer_id
                        FROM orders
                       );
```
```sql
-- Approach 2
SELECT *
FROM customers as c
LEFT JOIN
orders as o
ON o.customer_id = c.customer_id
WHERE o.customer_id IS NULL;
```

Task 6. Least-Selling Categories by State
Identify the least-selling product category for each state.
Challenge: Include the total sales for that category within each state.

```sql
WITH ranking_table
AS
(
SELECT 
     c.state,
     cat.category_name,
     SUM(oi.total_sale) as total_sale,
     RANK() OVER(PARTITION BY c.state ORDER BY SUM(oi.total_sale) ASC) as rank
FROM orders as o
JOIN 
customers as c
ON o.customer_id = c.customer_id
JOIN
order_items as oi
ON o.order_id = oi. order_id
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN
category as cat
ON cat.category_id = p.category_id
GROUP BY 1, 2
)
SELECT 
*
FROM ranking_table
WHERE rank = 1;
```

Task 7. Customer Lifetime Value (CLTV)
Calculate the total value of orders placed by each customer over their lifetime.
Challenge: Rank customers based on their CLTV. If ties don't skip next

```sql
SELECT 
     c.customer_id,
     CONCAT(c.first_name, ' ',  c.last_name) as customer_name,
     ROUND(SUM(oi.total_sale::numeric), 2) as cltv,
     DENSE_RANK() OVER( ORDER BY SUM(total_sale) DESC) as rank
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1;
```

Task 8. Inventory Stock Alerts
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
JOIN
products as p
ON p.product_id = i.product_id
WHERE stock < 10;
```

Task 9. Shipping Delays
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
WHERE
    s.shipping_date - o.order_date > 3;
```

Task 10. Payment Success Rate 
Calculate the percentage of successful payments across all orders.
Challenge: Include breakdowns by payment status (e.g., failed, pending).

```sql
SELECT 
     p.payment_status,
     COUNT(*) as total_cnt,
     COUNT(*)::numeric/(SELECT COUNT(*) FROM payments)::numeric * 100
FROM orders as o
JOIN
payments as p
ON o.order_id = p.order_id
GROUP BY 1;
```

Task 11. Top Performing Sellers
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
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END)::numeric/
	SUM(total_orders)::numeric * 100 as successful_orders_percentage
FROM sellers_reports
GROUP BY 1, 2;
```

Task 12. Product Profit Margin
Calculate the profit margin for each product (difference between price and cost of goods sold).
Challenge: Rank products by their profit margin, showing highest to lowest.

```sql
SELECT 
	p.product_id,
	p.product_name,
	SUM(total_sale - (p.cogs * oi.quantity))/sum(total_sale) * 100 as profit_margin,
	DENSE_RANK() OVER(ORDER BY SUM(total_sale - (p.cogs * oi.quantity))/sum(total_sale) * 100 DESC) as product_ranking
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
GROUP BY 1, 2;
```

Task 13. Most Returned Products
Query the top 10 products by the number of returns.
Challenge: Display the return rate as a percentage of total units sold for each product.

```sql
SELECT 
	p.product_id,
	p.product_name,
	COUNT(*) as total_unit_sold,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_returned,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END)::numeric/COUNT(*)::numeric * 100 as return_percentage
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN orders as o
ON o.order_id = oi.order_id
GROUP BY 1, 2
ORDER BY 5 DESC;
```

Task 14. Orders Pending Shipment
Find orders that have been paid but are still pending shipment.
Challenge: Include order details, payment date, and customer information.

```sql
SELECT
    o.*,
    p.payment_date,
    c.*
FROM orders as o
JOIN 
    payments as p
    ON o.order_id = p.order_id
JOIN
    customers as c
    ON o.customer_id = c.customer_id
WHERE 
    p.payment_status = 'Payment Successed'
    AND 
    o.order_status = 'Inprogress';
```

Task 15. Inactive Sellers
Identify sellers who haven’t made any sales in the last 6 months.
Challenge: Show the last sale date and total sales from those sellers.

```sql
--Approach 1

SELECT
     s.seller_name,
	 max(o.order_date) as last_sale_date,
	 sum(oi.total_sale) as total_sales
FROM sellers as s
JOIN
    orders as o
    ON s.seller_id = o.seller_id
JOIN
    order_items as oi
    ON o.order_id = oi.order_id
WHERE 
    s.seller_id not in (select seller_id from sellers where order_date >= CURRENT_DATE - INTERVAL '6 month')
GROUP BY 1;
```
```sql
--Approach 2

WITH cte1 -- as these sellers has not done any sale in last 6 month
AS
(SELECT * FROM sellers
WHERE seller_id NOT IN (SELECT seller_id FROM orders WHERE order_date >= CURRENT_DATE - INTERVAL '6 month')
)

SELECT 
s.seller_name,
MAX(o.order_date) as last_sale_date,
SUM(oi.total_sale) as total_sales
FROM orders as o
JOIN 
cte1
ON cte1.seller_id = o.seller_id
JOIN order_items as oi
ON o.order_id = oi.order_id
GROUP BY 1;
```

Task 16. IDENTITY customers into returning or new
if the customer has done more than 5 return categorize them as returning otherwise new
Challenge: List customers id, name, total orders, total returns

```sql
SELECT
     customer_name,
     total_orders,
     total_returns,
     CASE
         WHEN total_returns > 5 THEN 'Returning_customers' ELSE 'New'
END as customer_category
FROM
(
SELECT 
     CONCAT(c.first_name, ' ', c.last_name) as customer_name,
     COUNT(o.order_id) as total_orders,
     SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_returns
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1;
```

Task 17. Top 5 Customers by Orders in Each State
Identify the top 5 customers with the highest number of orders for each state.
Challenge: Include the number of orders and total sales for each customer.
```sql
-- Approach 1 - Subquery

SELECT 
     DISTINCT c.customer_id,
	 CONCAT(c.first_name, ' ', c.last_name) as customer_name,
	 'Apple AirPods Pro' as product_purchased,
	 'Apple AirPods Max' as suggested_product
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON o.order_id = oi.order_id
JOIN 
products as p
ON oi.product_id = p.product_id
WHERE 
p.product_name = 'Apple AirPods Pro'
AND c.customer_id NOT IN (
                           SELECT 
                               DISTINCT c2.customer_id
                               FROM customers as c2
                               JOIN orders as o2
                               ON c2.customer_id = o2.customer_id
                               JOIN order_items as oi2
                               ON o2.order_id = oi2.order_id
                               JOIN products as p2
                               ON oi2.product_id = p2.product_id
                               WHERE p2.product_name = 'Apple AirPods Max'
                         );
```
```sql
--Approach 2 - CTE

WITH products_purchases AS
(
  SELECT 
      c.customer_id,
	  p.product_name
  FROM customers as c
  JOIN 
  orders as o
  ON c.customer_id = o.customer_id
  JOIN 
  order_items as oi
  ON o.order_id = oi.order_id
  JOIN 
  products as p
  ON oi.product_id = p.product_id
),
airpods_pro_customers AS
(
 SELECT DISTINCT customer_id
 FROM products_purchases
 WHERE product_name = 'Apple AirPods Pro'
),
airpods_max_customers AS
(
 SELECT DISTINCT customer_id
 FROM products_purchases
 WHERE product_name = 'Apple AirPods Max'
)
SELECT 
     DISTINCT c.customer_id,
     CONCAT(c.first_name, ' ', c.last_name) as customer_name,
     'Apple AirPods Pro' as product_purchased,
     'Apple AirPods Max' as suggested_product
FROM customers as c
JOIN 
airpods_pro_customers as apc
ON c.customer_id = apc.customer_id
WHERE 
c.customer_id NOT IN (
                       SELECT DISTINCT customer_id 
                       FROM airpods_max_customers
                     );  
```

Task 18. Revenue by Shipping Provider
Calculate the total revenue handled by each shipping provider.
Challenge: Include the total number of orders handled and the average delivery time for each provider.

```sql
SELECT 
     s.shipping_providers,
     COUNT(o.order_id) as order_handled,
     SUM(oi.total_sale) as total_sale,
     COALESCE(AVG(s.return_date - s.shipping_date), 0) as average_days
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
shippings as s
ON 
s.order_id = o.order_id
GROUP BY 1
```

Task 19. Top 10 product with highest decreasing revenue ratio compare to last year(2022) and current_year(2023)
Challenge: Return product_id, product_name, category_name, 2022 revenue and 2023 revenue decrease ratio at end Round the result
Note: Decrease ratio = cr-ls/ls* 100 (cs = current_year ls=last_year)

```sql
WITH last_year_sale
as
(
SELECT 
     p.product_id,
     p.product_name,
     SUM(oi.total_sale) as revenue
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON 
p.product_id = oi.product_id
WHERE
    EXTRACT(YEAR FROM o.order_date) = 2022
GROUP BY 1, 2
),

current_year_sale
AS
(
SELECT 
	p.product_id,
	p.product_name,
	SUM(oi.total_sale) as revenue
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON 
p.product_id = oi.product_id
WHERE EXTRACT(YEAR FROM o.order_date) = 2023
GROUP BY 1, 2
)

SELECT
	cs.product_id,
	ls.revenue as last_year_revenue,
	cs.revenue as current_year_revenue,
	ls.revenue - cs.revenue as rev_diff,
	ROUND((cs.revenue - ls.revenue)::numeric/ls.revenue::numeric * 100, 2) as reveneue_dec_ratio
FROM last_year_sale as ls
JOIN
current_year_sale as cs
ON ls.product_id = cs.product_id
WHERE 
    ls.revenue > cs.revenue
ORDER BY 5 DESC
LIMIT 10;
```

Task 20. FINAL TASK - Stored Procedure and TRIGGER  
Create a stored procedure that, when a product is sold, performs the following actions:
Inserts a new sales record into the orders and order_items tables.
Updates the inventory table to reduce the stock based on the product and quantity purchased.
The procedure should ensure that the stock is adjusted immediately after recording the sale.

```sql
-- Stored Procedure

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

```sql
-- Trigger

--pgsql
CREATE OR REPLACE FUNCTION update_inventory_on_order()
RETURNS TRIGGER AS $$
BEGIN
    -- Check stock availability
    IF NEW.quantity > (SELECT stock FROM inventory WHERE order_id = NEW.order_id) THEN
	
        RAISE NOTICE 'Not enough stock for item_id %', NEW.item_id;
    END IF;

    -- Update inventory
    UPDATE inventory
    SET stock = stock - NEW.quantity
    WHERE orderid = NEW.order_id;
    RETURN NEW;
	
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_inventory
BEFORE INSERT ON order_items
FOR EACH ROW
EXECUTE FUNCTION update_inventory_on_order();


-- plsql

CREATE OR REPLACE TRIGGER trg_update_inventory
BEFORE INSERT ON order_items
FOR EACH ROW
DECLARE
    v_stock inventory.stock%TYPE;
BEGIN
    -- Fetch current stock
    SELECT stock INTO v_stock
    FROM inventory
    WHERE item_id = :NEW.item_id;

    -- Check if enough stock is available
    IF :NEW.quantity > v_stock THEN
        RAISE_APPLICATION_ERROR(-20001, 'Not enough stock for item_id ' || :NEW.item_id);
    END IF;

    -- Update inventory stock
    UPDATE inventory
    SET stock = stock - :NEW.quantity
    WHERE item_id = :NEW.item_id;
END;
/
```
---

## **54 Basic-Intermediate Level challenges**

### Basic Queries, Joins and Set Operations

Task 1. **Basic Select**: Retrieve the names of all unique products in the `products` table.

```sql
SELECT 
DISTINCT product_name 
FROM products;
```

Task 2. **Simple Join**: Write a query to find the full name of customers (`first_name` + `last_name`) and the names of the products they ordered. Use a JOIN between `customers`, `orders`, `order_items`, and `products`.

```sql
SELECT 
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    p.product_name
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
JOIN
products as p
ON oi.product_id = p.product_id;
```

Task 3. **Conditional Select**: List all products with a price greater than 100. Display the product name and price.

```sql
SELECT 
     product_name,
     price
FROM products
WHERE 
     price > 100;
```

Task 4. **Inner Join**: List all orders details along with customer names and product names. Use INNER JOIN between `orders`, `customers`, and `products`.

```sql
SELECT 
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    p.product_name,
    o.*
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
JOIN
products as p
ON oi.product_id = p.product_id;
```

Task 5. **Left Join**: Retrieve all customers and their corresponding orders. Include customers who haven't placed any orders.

```sql
SELECT *
FROM customers as c
LEFT JOIN
orders as o
ON c.customer_id = o.customer_id;
```

Task 6. **Right Join**: Retrieve all orders and their corresponding customers. Include orders without customer information.

```sql
SELECT *
FROM orders as o
RIGHT JOIN
customers as c
ON o.customer_id = c.customer_id;
```

Task 7. **Join with Filtering**: List all products sold by sellers originating from "USA." Include product names and seller names.

```sql
SELECT 
     p.product_name,
     seller_name,
     origin
FROM products as p
JOIN 
order_items as oi
ON p.product_id = oi.product_id
JOIN
orders as o
ON oi.order_id = o.order_id
JOIN 
sellers as s
ON o.seller_id = s.seller_id
WHERE
     origin = 'USA';
```

Task 8. **Multi-table Join**: Write a query to find the total amount paid for each order. Include the `orders`, `order_items`, and `payments` tables.

```sql
SELECT 
     o.order_id,
     oi.total_sale
FROM 
order_items as oi
JOIN
orders as o
ON oi.order_id = o.order_id
JOIN
payments as p
ON o.order_id = p.order_id;
```

Task 9. **Join with Subquery**: List the customers who have ordered products in the "electronics" category. Use a subquery to find the category ID.

```sql
SELECT 
    DISTINCT c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
JOIN
products as p
ON oi.product_id = p.product_id
WHERE 
    p.category_id = (SELECT category_id FROM category WHERE category_name = 'electronics');
```

Task 10. **Cross Join**: Write a query to list all combinations of `category` and `sellers`.

```sql
SELECT * FROM
category 
CROSS JOIN 
sellers;
```

Task 11. **Intersect**: Write a query to fetch common rows between tables order_items and products.

```sql
SELECT product_id 
FROM order_items
INTERSECT
SELECT product_id 
FROM products;
```

Task 12. **Union**: Write a query to fetch all rows from tables order_items and products. Remove duplicates.

```sql
SELECT order_id
FROM orders
UNION
SELECT order_id
FROM payments;
```

Task 13. **Union All**: Write a query to fetch all rows from tables order_items and products. Include duplicates.

```sql
SELECT order_id
FROM orders
UNION ALL
SELECT order_id
FROM payments;
```

Task 14. **Except / Minus**: Write a query to rows that are present in customers table but not in orders table.

```sql
SELECT customer_id
FROM customers
EXCEPT
SELECT customer_id
FROM orders;

--`MINUS` is used instead of `EXCEPT` in Oracle
```

---

### Aggregation and Grouping

Task 15. **Count Function**: Count the total number of unique customers in the `customers` table.

```sql
SELECT 
DISTINCT COUNT(customer_id) 
FROM customers;
```

Task 16. **Sum and Group By**: Find the total revenue generated by each seller. Display the seller name and total revenue.

```sql
SELECT 
     s.seller_name,
     SUM(oi.total_sale) as revenue
FROM orders as o
JOIN 
order_items as oi
ON o.order_id = oi.order_id
JOIN
sellers as s
ON o.seller_id = s.seller_id
GROUP BY 1;
```

Task 17. **Average Function**: Calculate the average price of products in the `products` table.

```sql
SELECT AVG(price)
FROM products;
```

Task 18. **Group By with Having**: List all sellers who have sold more than 500 products. Display seller names and total products sold.

```sql
SELECT 
     s.seller_name,
     COUNT(o.order_id) as total_products_sold
FROM sellers as s
JOIN
orders as o
ON s.seller_id = o.seller_id
GROUP BY 1
HAVING COUNT(o.order_id) > 500;
```

Task 19. **Group By Multiple Columns**: Find the total revenue generated by each seller for each category. Display seller names, category names, and total revenue.

```sql
SELECT 
     s.seller_name,
     c.category_name,
     SUM(oi.total_sale) as total_revenue
FROM sellers as s
JOIN 
orders as o
ON s.seller_id = o.seller_id
JOIN 
order_items as oi
ON o.order_id = oi.order_id
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN 
category as c 
ON p.category_id = c.category_id
GROUP BY 1, 2
ORDER BY 1, 2;
```

Task 20. **Count and Distinct**: Find the total number of distinct products sold in each category.

```sql
SELECT
     c.category_name,
     COUNT(DISTINCT p.product_id) as products_sold
FROM products as p
JOIN 
category as c
ON p.category_id = c.category_id
GROUP BY 1;
```

Task 21. **Join with Aggregation**: Write a query to find the total number of orders and the total revenue generated for each customer.

```sql
SELECT 
     CONCAT(c.first_name, ' ',  c.last_name) as customer_name,
     COUNT(o.order_id) as total_orders,
     SUM(total_sale) as total_revenue
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1;
```

Task 22. **Aggregate Functions and CASE**: Find the number of orders for each order status ("Inprogress," "Completed," etc.). Use CASE to categorize the statuses.

```sql
SELECT 
    COUNT(CASE WHEN order_status = 'Inprogress' THEN 1 END) as total_orders_inprogress,
    COUNT(CASE WHEN order_status = 'Completed' THEN 1 END) as total_orders_completed,
    COUNT(CASE WHEN order_status = 'Returned' THEN 1 END) as total_orders_returned,
    COUNT(CASE WHEN order_status = 'Cancelled' THEN 1 END) as total_orders_cancelled
FROM orders;
```

Task 23. **Nested Aggregation**: Find the category with the highest total revenue.

```sql
SELECT 
    c.category_name,
    SUM(oi.total_sale) as highest_total_revenue
FROM order_items as oi
JOIN
products as p
ON oi.product_id = p.product_id
JOIN
category as c
ON p.category_id = c.category_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

Task 24. **Conditional Aggregation**: Count the number of successful and failed payments for each customer.

```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    COUNT(CASE WHEN payment_status = 'Payment Successed' THEN 1 END) as successful_payments,
    COUNT(CASE WHEN payment_status = 'Payment Failed' THEN 1 END) as failed_payments
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
JOIN
payments as p
ON o.order_id = p.order_id
GROUP BY 1;
```

---

### Subqueries and Nested Queries

Task 25. **Simple Subquery**: Find the product with the highest price. Use a subquery to get the maximum price.

```sql
SELECT 
     product_name,
     price
FROM products 
WHERE
    price = (SELECT MAX(price) FROM products);
```

Task 26. **Correlated Subquery**: Find all products whose price is above the average price in their category.

```sql
SELECT 
     p1.product_name,
     p1.price
FROM products as p1
WHERE
     p1.price > (SELECT AVG(p2.price) FROM products as p2);
```

Task 27. **Subquery in WHERE Clause**: Retrieve the names of customers who have ordered at least one product in the "Pet Supplies" category.

```sql
SELECT 
    DISTINCT c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name
FROM customers as c
JOIN 
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
JOIN 
products as p
ON oi.product_id = p.product_id
WHERE 
    p.category_id = (SELECT category_id FROM category WHERE category_name = 'Pet Supplies');
```

Task 28. **Subquery in SELECT Clause**: For each product, display its name and the total number of times it has been ordered.

```sql
SELECT 
     DISTINCT sub.product_name,
     COUNT(*) as times_ordered
FROM
(
  SELECT *
  FROM
  products as p
  JOIN 
  order_items as oi
  ON p.product_id = oi.product_id
) as sub
GROUP BY 1;
```

Task 29. **Subquery with EXISTS**: List all customers who have made at least one order.

```sql
SELECT * FROM
customers as c
WHERE EXISTS (
               SELECT 1 
               FROM orders as o
               WHERE 
               c.customer_id = o.customer_id
             );
```

Task 30. **IN Clause with Subquery**: Find the names of sellers who have sold "Apple" products.

```sql
SELECT seller_name
FROM sellers
WHERE seller_id IN 
                  ( SELECT seller_id 
                    FROM orders 
                    WHERE order_id IN
                                     ( SELECT order_id 
                                       FROM order_items
                                       WHERE product_id IN 
                                                           ( SELECT product_id
                                                             FROM products
                                                             WHERE product_name ILIKE '%Apple%'
                                                           )
                                     )
				 
                  );
```

Task 31. **NOT IN Clause**: List all customers who have not placed any orders.

```sql
SELECT *
FROM customers 
WHERE
customer_id NOT IN (SELECT customer_id FROM orders);
```

Task 32. **Subquery with JOIN**: Find the names of products that are out of stock. Use a subquery to get product IDs with stock = 0 in the `inventory` table.

```sql
SELECT p.product_name
FROM products as p
JOIN 
inventory as i
ON p.product_id = i.product_id
WHERE
p.product_id = (SELECT product_id FROM inventory WHERE stock = 0);
```

Task 33. **Subquery with HAVING**: Retrieve sellers who have an average selling price of their products greater than 300.

```sql
SELECT 
     s.seller_name
FROM sellers as s
WHERE 
s.seller_id IN (SELECT o.seller_id 
                FROM orders as o 
                JOIN order_items as oi 
                ON o.order_id = oi.order_id 
                JOIN products as p
                ON oi.product_id = p.product_id
                GROUP BY o.seller_id
                HAVING AVG(p.price) > 100
               );
```

Task 34. **Multi-level Subqueries**: Find the product that has generated the highest revenue. Use nested subqueries to calculate revenue.

```sql
SELECT 
    product_id,
    product_name
FROM products
WHERE 
product_id = ( SELECT product_id
               FROM 
                    ( SELECT 
                      oi.product_id,
                      SUM(oi.total_sale) as total_revenue
                      FROM products as p
                      JOIN
                      order_items as oi
                      ON p.product_id = oi.product_id
                      GROUP BY oi.product_id
                      ORDER BY total_revenue DESC
                      LIMIT 1
                    )
              );
```

---

### Window Functions

Task 35. **RANK() Function**: For each category, rank the products based on their total sales amount.

```sql
SELECT 
    c.category_name,
    p.product_name,
    ROUND(SUM(oi.total_sale)::numeric, 2) as total_sales,
    RANK() OVER (PARTITION BY c.category_name ORDER BY SUM(oi.total_sale) DESC) as rank
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN 
category as c 
ON p.category_id = c.category_id
GROUP BY 1, 2;
```

Task 36. **DENSE_RANK() Function**: List the top 5 customers based on the total amount spent. Use `DENSE_RANK()`.

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    ROUND(SUM(oi.total_sale)::numeric, 2) as total_sales,
    DENSE_RANK() OVER (ORDER BY SUM(oi.total_sale) DESC) as rank
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
GROUP BY 1
LIMIT 5;
```

Task 37. **ROW_NUMBER() Function**: Assign a row number to each product in the `products` table, ordered by price descending.

```sql
SELECT 
       *,
       ROW_NUMBER() OVER (ORDER BY price DESC) as row_number
FROM products;
```

Task 38. **NTILE() Function**: Divide all customers into 4 quartiles based on the total amount they have spent.

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    ROUND(SUM(oi.total_sale)::numeric, 2) as total_amount_spent,
    NTILE(4) OVER (ORDER BY SUM(oi.total_sale)) as quartile
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id
GROUP BY 1;
```

Task 39. **OVER Clause**: For each order, calculate the running total of sales for the corresponding customer.

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    o.order_date,
    oi.total_sale,
    SUM(total_sale) OVER (PARTITION BY c.customer_id ORDER BY o.order_date) as running_total_sales
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id;
```

Task 40. **PARTITION BY Clause**: Find the total revenue generated by each seller in each year.

```sql
SELECT
     s.seller_id,
     s.seller_name,
     EXTRACT(YEAR FROM o.order_date) as year,
     SUM(oi.total_sale) OVER (PARTITION BY s.seller_id, EXTRACT(YEAR FROM o.order_date)) as total_revenue
FROM sellers as s
JOIN
orders as o
ON s.seller_id = o.seller_id
JOIN 
order_items as oi
ON o.order_id = oi.order_id;
```

Task 41. **LEAD() Function**: For each product, find the next higher-priced product in the same category.

```sql
SELECT 
     c.category_name,
     p.product_name,
     p.price,
     LEAD(p.product_name) OVER (PARTITION BY c.category_name ORDER BY p.price) as next_higher_priced_product
FROM category as c
JOIN
products as p
ON c.category_id = p.category_id;
```

Task 42. **LAG() Function**: For each product, find the previous lower-priced product in the same category.

```sql
SELECT 
     c.category_name,
     p.product_name,
     p.price,
     LAG(p.product_name) OVER (PARTITION BY c.category_name ORDER BY p.price) as previous_lower_priced_product
FROM category as c
JOIN
products as p
ON c.category_id = p.category_id;
```

Task 43. **Cumulative Sum**: Calculate the cumulative sum of sales for each seller.

```sql
SELECT
     s.seller_id,
     s.seller_name,
     SUM(oi.total_sale) OVER (PARTITION BY s.seller_id) as cumulative_sales
FROM sellers as s
JOIN
orders as o
ON s.seller_id = o.seller_id
JOIN 
order_items as oi
ON o.order_id = oi.order_id;
```

Task 44. **Window Function with Aggregation**: Find the average order amount for each customer and compare it with their individual orders.

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    oi.total_sale,
    AVG(oi.total_sale) OVER (PARTITION BY c.customer_id) as avg_order_amount,
    oi.total_sale - AVG(oi.total_sale) OVER (PARTITION BY c.customer_id) as difference_from_average
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON o.order_id = oi.order_id;
```

---

### Date Functions

Task 45. **Date Filtering**: List all orders placed in the current month. Include order ID, order date, and customer name.

```sql
SELECT 
     o.order_id,
     order_date,
     CONCAT(c.first_name, ' ', c.last_name) as customer_name
FROM customers as c
JOIN
orders as o
ON c.customer_id = o.customer_id
WHERE 
EXTRACT(YEAR FROM o.order_date) = EXTRACT(YEAR FROM CURRENT_DATE)
AND
EXTRACT(MONTH FROM o.order_date) = EXTRACT(MONTH FROM CURRENT_DATE);

-- WHERE condition using SQL server
YEAR(o.order_date) = YEAR(GETDATE())
AND
MONTH(o.order_date) = MONTH(GETDATE());

-- WHERE condition using MYSQL
YEAR(o.order_date) = YEAR(CURDATE())
AND
MONTH(o.order_date) = MONTH(CURDATE());
```

Task 46. **Extract and Group By**: Find the number of orders placed in each year. Use the `EXTRACT()` function to group by year. 

```sql
SELECT
     EXTRACT(YEAR FROM order_date) as year,
     COUNT(*) as total_orders_placed
FROM orders
GROUP BY 1
ORDER BY 1;
```

Task 47. **DATEDIFF Function**: Calculate the average delivery time for all delivered orders. 

```sql
--SQL server
SELECT 
     AVG(DATEDIFF(day, o.order_date, s.return_date)) as avg_delivery_days
FROM orders as o
JOIN
shippings as s
ON o.order_id = s.order_id;

-- Date arithmetic operation for MySQL will be as follows
AVG(DATEDIFF(s.return_date, o.order_date)) as avg_delivery_days

-- Date arithmetic operation for Postgre SQL will be as follows
AVG(s.return_date - o.order_date) as avg_delivery_days
```

Task 48. **DATE_TRUNC Function**: Find the total sales amount for each month in the current year.

```sql
SELECT 
     DATE_TRUNC('month', o.order_date) as month_start,
     SUM(oi.total_sale) as total_sales_amount
FROM orders as o
JOIN
order_items as oi
ON o.order_id = oi.order_id
WHERE
EXTRACT(YEAR FROM o.order_date) = EXTRACT(YEAR FROM CURRENT_DATE)
GROUP BY 1;
```

Task 49. **Age Function**: Find customers who have not placed any orders in the last 6 months.

```sql
SELECT
     c.customer_id,
     CONCAT(c.first_name, ' ', c.last_name) as customer_name
FROM customers as c
WHERE 
NOT EXISTS ( SELECT 1 FROM orders as o
             WHERE c.customer_id = o.customer_id
             AND
             AGE(CURRENT_DATE, o.order_date) <= INTERVAL '6 months'
           );
```

Task 50. **Date Conversion**: Convert the `order_date` to a different format (e.g., 'YYYY-MM-DD') and display it with the order ID.

```sql
--PostgreSQL
SELECT
     order_id,
     TO_CHAR(order_date, 'YYYY-MM-DD') as formatted_order_date
FROM orders
```
```sql
--SQL server
SELECT
     order_id,
     CONVERT(VARCHAR(10), order_date, 120) as formatted_order_date
FROM orders
```
```sql
--MySQL
SELECT
     order_id,
     DATE_FORMAT(order_date, '%Y-%m-%d') as formatted_order_date
FROM orders
```

Task 51. **Date Arithmetic**: Calculate the total number of days between the order date and shipping date for each order.

```sql
--PostgreSQL
SELECT
     o.order_id,
     o.order_date,
     s.return_date as delivery_date,
     s.return_date - o.order_date as days_for_delivery
FROM orders as o
JOIN
shippings as s
ON o.order_id = s.order_id
WHERE
    s.return_date IS NOT NULL
GROUP BY
    o.order_id, s.return_date, o.order_date

-- Date arithmetic operation for SQL server will be as follows
DATEDIFF(day, o.order_date, s.return_date)) as avg_delivery_days

-- Date arithmetic operation for MySQL will be as follows
DATEDIFF(s.return_date, o.order_date)) as avg_delivery_days
```

Task 52. **Current Date Usage**: Find all orders that are overdue for payment. Assume payment is due within 30 days of the order date.

```sql
SELECT
     o.order_id,
     p.payment_id
FROM payments as p
JOIN 
orders as o
ON p.order_id = o.order_id
WHERE 
     CURRENT_DATE - o.order_date > 30
     AND
     p.payment_status = 'Pending';
 ```

Task 53. **Weekend Orders**: Retrieve all orders that were placed on weekends.

```sql
--Postgre SQL
SELECT *
FROM orders
WHERE EXTRACT(DOW FROM order_date) IN (0, 6);
```
```sql
--MySQL
SELECT *
FROM orders
WHERE DAYOFWEEK(order_date) IN (1, 7);
```
```sql
--SQL server
SELECT *
FROM orders
WHERE DATEPART(WEEKDAY, order_date) IN (1, 7);
```

Task 54. **Next Day Delivery**: List all orders that were delivered the next day after shipping.

```sql
--Postgre SQL
SELECT *
FROM shippings
WHERE return_date = shipping_date + INTERVAL '1 day';
```
```sql
--MySQL
SELECT *
FROM shippings
WHERE DATEDIFF(shipping_date, return_date) = 1;
```
```sql
--SQL server
SELECT *
FROM shippings
WHERE DATEDIFF(day, shipping_date, return_date) = 1;
```

## **Learning Outcomes**

This project enabled me to:
- Design and implement a normalized database schema.
- Clean and preprocess real-world datasets for analysis.
- Use advanced SQL techniques, including window functions, subqueries, and joins.
- Conduct in-depth business analysis using SQL.
- Optimize query performance and handle large datasets efficiently.

---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (used) (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## Conclusion

This advanced SQL project successfully demonstrates my ability to solve real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.

## Author - Prayag Dave

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/prayag-dave-56b3681b3)
- **Mail**: prayagdavework@gmail.com

Thank you for your support, and I look forward to connecting with you!
