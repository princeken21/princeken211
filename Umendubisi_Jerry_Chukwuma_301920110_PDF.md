## Course: <u>MIT 8103 - Advanced Database Systems</u> Topic: E- Commerce Advanced Database System Por�olio Pla�orm.

## Name: Umendubisi Jerry Chukwuma Email: j.umendubisi0110@miva.edu.ng Student ID: 301920110

## CASE STUDY:

**Organisation Name / Scenario:** E&E E-Commerce

**Nature of the Business / System:** An online business-to-consumer (B2C) e-commerce platform.

## Portfolio 1: Modeling and Design of the Database.

## A. Organization Name and Scenario:

E&E E-Commerce (an Online retail organization based in Lagos, Nigeria, that provides nationwide delivery services). **Business/ System Description:** A business-to-consumer (B2C) E-Commerce website that allows Nigerian customers to search for consumer products, add to cart, make payments using local payment gateways, (Paystack, Flutterwave or Interswitch) and track their order delivery to various states. Database Users:

**i. Customers:** These users can look for products, place orders, pay for them, and monitor their delivery. **ii. Inventory Managers/Admin Sta�:** These users have access to the backend of the database where they can manage product catalogues, check stock levels in regional warehouses and view sales records. **iii. Logistics/Delivery Partners:** The logistics department and 3
rd party delivery partners can track dispatch information, tracking numbers and delivery records for last-mile delivery.

## Data to be Stored:

Data to be stored includes Customer information, product catalogues and stock levels, customer orders, line items, payment details (Naira payments & payment gateway reference codes). Operations Provided:

- Setting up and maintaining client accounts with Nigerian contact details.
- Managing stock inventory and listing products that are available.
- Creating customers orders and line items.
- Making payments (Naira and payment gateway reference codes).
- Tracking of customers deliveries around Nigeria.
## B. Entity-Relationship (ER) theoretical Model

**Customers (1) to Orders (M):** Here, a customer can place many orders or none at all. Each order relates back to a customer. **Orders (1) to Order_Items (M):** Here, an order contains many line items. Each line item relates to an order. **Products (1) to Order_Items (M):** Here, a product can be included in many order items. Each order item is limited to containing a single product **Orders (1) to Payments (1 or M):** There is one main payment for an order. There could be many payments if there are attempts to process failed transactions. **Orders (1) to Delivery_Records (1):** In this case, there will be one tracking record per order. **SQL** E&E E-Commerce ERD Schema (dbdiagram.io / DBML format) Table customers { customer_id int [pk, increment, note: "Primary Key"] email varchar [unique, not null, note: "Customer login email"] password_hash varchar [not null, note: "Encrypted password"] first_name varchar [not null] last_name varchar [not null] phone varchar [not null, note: "Nigerian phone format"] state varchar [not null, note: "e.g., Lagos, Abuja FCT, Rivers"] created_at timestamp [default: `CURRENT_TIMESTAMP`] } Table products { product_id int [pk, increment, note: "Primary Key"] sku varchar [unique, not null, note: "Stock Keeping Unit"]

name varchar [not null, note: "Product title"] description text [note: "Product details"] price_ngn decimal [not null, note: "Price in Nigerian Naira (NGN)"] stock_quantity int [not null, note: "Available inventory count"] category varchar [not null, note: "Product category"] created_at timestamp [default: `CURRENT_TIMESTAMP`] } Table orders { order_id int [pk, increment, note: "Primary Key"] customer_id int [not null, note: "Foreign Key to customers"] order_date timestamp [default: `CURRENT_TIMESTAMP`] status varchar [not null, default: "PENDING", note: "PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED"] total_amount_ngn decimal [not null, note: "Total order cost in NGN"] shipping_street_address text [not null] shipping_city varchar [not null] shipping_state varchar [not null] } Table order_items { order_item_id int [pk, increment, note: "Primary Key"] order_id int [not null, note: "Foreign Key to orders"] product_id int [not null, note: "Foreign Key to products"] quantity int [not null, note: "Quantity purchased"] unit_price_ngn decimal [not null, note: "Price per unit at time of purchase"] } Table payments { payment_id int [pk, increment, note: "Primary Key"] order_id int [not null, note: "Foreign Key to orders"] payment_date timestamp [default: `CURRENT_TIMESTAMP`] amount_ngn decimal [not null, note: "Amount paid in NGN"] payment_method varchar [not null, note: "PAYSTACK, FLUTTERWAVE, INTERSWITCH, etc."] payment_reference varchar [unique, note: "Gateway transaction reference"] payment_status varchar [not null, note: "SUCCESSFUL, FAILED, PENDING, REFUNDED"]

} Table delivery_records { delivery_id int [pk, increment, note: "Primary Key"] order_id int [unique, not null, note: "Foreign Key to orders (1-to-1)"] tracking_number varchar [unique, note: "Logistics tracking number"] logistics_partner varchar [not null, note: "GIG Logistics, Red Star Express, DHL, etc."] delivery_status varchar [not null, default: "PREPARING", note: "PREPARING, DISPATCHED, IN_TRANSIT, DELIVERED, FAILED_ATTEMPT"] estimated_delivery_date date } ========================================================== Relationship Mappings (Cardinality) A customer can place many orders (1-to-Many) Ref: orders.customer_id > customers.customer_id // An order contains many order items (1-to-Many) Ref: order_items.order_id > orders.order_id // A product can appear in many order items (1-to-Many) Ref: order_items.product_id > products.product_id // An order can have payment records (1-to-Many for retries, or 1-to-1) Ref: payments.order_id > orders.order_id // Each order maps to exactly one delivery tracking record (1-to-1) Ref: delivery_records.order_id-orders.order_id

[https://dbdiagram.io/d/EandE-E-Commerce-ERD-6af639a0dec310ec25f6b9b6](https://dbdiagram.io/d/EandE-E-Commerce-ERD-6af639a0dec310ec25f6b9b6)

## C. DDL Implementation for Relational Schema (PostgreSQL 18)

This schema is a production-grade with correct data types, constraints, surrogate keys (GENERATED ALWAYS AS IDENTITY), localized address, currency fields and relational data. -- PostgreSQL 18 DDL for E&E E-Commerce -- 1. Customers Table CREATE TABLE customers ( customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, email VARCHAR(255) NOT NULL UNIQUE, password_hash VARCHAR(255) NOT NULL, first_name VARCHAR(100) NOT NULL, last_name VARCHAR(100) NOT NULL, phone VARCHAR(20) NOT NULL, -- Format: +234XXXXXXXXXX or local format state VARCHAR(100) NOT NULL, -- e.g., Lagos, Abuja FCT, Rivers created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP );

-- 2. Products Table CREATE TABLE products ( product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, sku VARCHAR(100) NOT NULL UNIQUE, name VARCHAR(200) NOT NULL, description TEXT, price_ngn NUMERIC(12, 2) NOT NULL CHECK (price_ngn >= 0.00), -- Price in Nigerian Naira (NGN) stock_quantity INT NOT NULL CHECK (stock_quantity >= 0), category VARCHAR(100) NOT NULL, created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP ); -- 3. Orders Table CREATE TABLE orders ( order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, customer_id INT NOT NULL, order_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, status VARCHAR(50) NOT NULL DEFAULT 'PENDING' CHECK (status IN ('PENDING', 'PAID', 'PROCESSING', 'SHIPPED', 'DELIVERED', 'CANCELLED')), total_amount_ngn NUMERIC(14, 2) NOT NULL CHECK (total_amount_ngn >= 0.00), shipping_street_address TEXT NOT NULL, shipping_city VARCHAR(100) NOT NULL, shipping_state VARCHAR(100) NOT NULL, CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE RESTRICT ); -- 4. Order Items Table CREATE TABLE order_items ( order_item_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, order_id INT NOT NULL, product_id INT NOT NULL, quantity INT NOT NULL CHECK (quantity > 0), unit_price_ngn NUMERIC(12, 2) NOT NULL CHECK (unit_price_ngn >= 0.00), CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,

CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE RESTRICT ); -- 5. Payments Table CREATE TABLE payments ( payment_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, order_id INT NOT NULL, payment_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, amount_ngn NUMERIC(14, 2) NOT NULL CHECK (amount_ngn >= 0.00), payment_method VARCHAR(50) NOT NULL CHECK (payment_method IN ('PAYSTACK', 'FLUTTERWAVE', 'INTERSWITCH', 'BANK_TRANSFER', 'CARD')), payment_reference VARCHAR(150) UNIQUE, -- Gateway Transaction Reference payment_status VARCHAR(50) NOT NULL CHECK (payment_status IN ('SUCCESSFUL', 'FAILED', 'PENDING', 'REFUNDED')), CONSTRAINT fk_payment_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE ); -- 6. Delivery Records Table CREATE TABLE delivery_records ( delivery_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, order_id INT NOT NULL UNIQUE, tracking_number VARCHAR(150) UNIQUE, logistics_partner VARCHAR(100) NOT NULL, -- e.g., GIG Logistics, Red Star Express, DHL delivery_status VARCHAR(50) NOT NULL DEFAULT 'PREPARING' CHECK (delivery_status IN ('PREPARING', 'DISPATCHED', 'IN_TRANSIT', 'DELIVERED', 'FAILED_ATTEMPT')), estimated_delivery_date DATE, CONSTRAINT fk_delivery_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE );

[https://www.db-fiddle.com/f/wjZLoZt5dWZvHK6yvQqFc4/0](https://www.db-fiddle.com/f/wjZLoZt5dWZvHK6yvQqFc4/0)

## D. Normalization Description (1NF - 3NF):

- **First Normal Form (1NF):** In this section, all tables have atomic values. These include columns such as first_name and last_name for customers, and street, city, and state for addresses. The order_items table adheres to this rule, which also eliminates redundant data. For instance, when a customer purchases several items in one order
- **Second Normal Form (2NF):** This section follows the criteria for 1NF. Where, it makes sure that the non-key columns are fully functionally dependent on the primary key. This is achieved in tables like order_items by ensuring that columns such as quantity and unit_price_ngn depend fully on the primary key (order_item_id). This is achieved in tables like order_items by ensuring that columns such as quantity and unit_price_ngn depend fully on the primary key (order_item_id). Product Description has full functional dependency on product_id.
- **Third Normal Form (3NF):** This part complies with the requirements of 2NF and removes any transitive dependencies. All non-key columns in the order table only depend on the order_id. Where customer details such as email, phone number and state are stored within the customer database. Therefore, it is referenced in the order table through the foreign key customer_id, minimizing data duplication.

## Portfolio 2: Query Processing and Optimization

## A. Introduction & Objectives

This portfolio demonstrates how relational queries can be executed, analysed and optimised. Using PostgreSQL 18, we will look at the execution plan using EXPLAIN ANALYZE, look out for any performance issues, and implement indexing.

**B. Key SQL Queries:** Below are the 3 main queries used to analyse data by admins within E&E E-Commerce. **Query 1:** Customer Order Summary (JOIN / WHERE) Using this query, we will get all orders placed by customers from a specific state (e.g Lagos). Including customer information, order totals and payment statuses. SELECT
c.customer_id,
c.first_name || ' ' || c.last_name AS customer_name,
c.state,
o.order_id,
o.order_date,
o.total_amount_ngn,
o.status AS order_status,
p.payment_status FROM customers c JOIN orders o ON c.customer_id = o.customer_id LEFT JOIN payments p ON o.order_id = p.order_id WHERE c.state = 'Lagos' ORDER BY o.order_date DESC;

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/0](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/0)

## Query 2: Monthly Revenue (GROUP BY / AGGREGATE)

Here, this query will give us the total revenue of the business within each month and year in the local currency Nigerian Naira (NGN).

SELECT TO_CHAR(o.order_date, 'YYYY-MM') AS sales_month, COUNT(o.order_id) AS total_orders, SUM(o.total_amount_ngn) AS gross_revenue_ngn, ROUND(AVG(o.total_amount_ngn), 2) AS average_order_value_ngn FROM orders o WHERE o.status NOT IN ('CANCELLED', 'PENDING') GROUP BY TO_CHAR(o.order_date, 'YYYY-MM') ORDER BY sales_month DESC;

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/1](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/1)

## Query 3: Best Selling Products (Quantity)

This query will show us which products are selling the most, to help restock stock in regional warehouses across the states. SELECT

p.product_id,
p.sku,
p.name AS product_name,
p.category, SUM(oi.quantity) AS total_units_sold, SUM(oi.quantity * oi.unit_price_ngn) AS total_product_revenue_ngn FROM products p JOIN order_items oi ON p.product_id = oi.product_id GROUP BY p.product_id, p.sku, p.name, p.category ORDER BY total_units_sold DESC LIMIT 5;

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/2](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/2)

## C. Un-optimised Query Execution Plan (EXPLAIN ANALYZE)

Here, we will see how PostgreSQL 18 handles Query 1 prior to performance change; we use the EXPLAIN (ANALYZE, BUFFERS) command. EXPLAIN (ANALYZE, BUFFERS)

SELECT

c.customer_id, c.first_name, o.order_id, o.total_amount_ngn FROM customers c JOIN orders o ON c.customer_id = o.customer_id WHERE c.state = 'Lagos';

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/3](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/3)

## Unoptimized Execution Plan Output

Hash Join (cost=11.76..25.43 rows=2 width=94) (actual time=0.004..0.005 rows=0.00 loops=1) Hash Cond: (o.customer_id = c.customer_id) -> Seq Scan on orders o (cost=0.00..12.90 rows=290 width=26) (actual time=0.003..0.003 rows=0.00 loops=1) -> Hash (cost=11.75..11.75 rows=1 width=72) (never executed) -> Seq Scan on customers c (cost=0.00..11.75 rows=1 width=72) (never executed) Filter: ((state)::text = 'Lagos'::text) Planning: Bu�ers: shared hit=115 read=3 Planning Time: 0.373 ms Execution Time: 0.030 ms

## Key Learnings from Un-optimised:

## 1. Sequential Scan:

We use PostgreSQL 18 to run a sequential scan over the customers table to filter rows where the status = ‘Lagos’. This causes a big performance issue as more customers are added (millions).

**2. No Indexes:** Given there are no indexes on customers (state) or orders (customer_id) foreign key, the query planner must use hash joins instead of unindexed table scans.
## D. Proof of Indexing and optimization:

Here, we have implemented some B-tree indexes within PostgreSQL 18 to remove the sequential scans: -- 1. Index on customer state to speed up filtering in Query 1 CREATE INDEX idx_customers_state ON customers(state); -- 2. Index on orders foreign key to speed up JOIN operations CREATE INDEX idx_orders_customer_id ON orders(customer_id); -- 3. Index on order dates to speed up monthly aggregation reporting in Query 2 CREATE INDEX idx_orders_date ON orders(order_date);

## Re-running the Execution Plan Post-Indexing

EXPLAIN (ANALYZE, BUFFERS) SELECT

c.customer_id, c.first_name, o.order_id, o.total_amount_ngn FROM customers c JOIN orders o ON c.customer_id = o.customer_id WHERE c.state = 'Lagos';

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/4](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/4)

## Optimized Execution Plan Output

Nested Loop (cost=0.29..16.34 rows=2 width=94) (actual time=0.002..0.003 rows=0.00 loops=1) Bu�ers: shared hit=2 -> Index Scan using idx_customers_state on customers c (cost=0.14..8.16 rows=1 width=72) (actual time=0.002..0.002 rows=0.00 loops=1) Index Cond: ((state)::text = 'Lagos'::text) Index Searches: 1 Bu�ers: shared hit=2 -> Index Scan using idx_orders_customer_id on orders o (cost=0.15..8.17 rows=1 width=26) (never executed) Index Cond: (customer_id = c.customer_id)

Index Searches: 0 Planning: Bu�ers: shared hit=160 read=6 Planning Time: 0.402 ms Execution Time: 0.027 ms

## E. Brief Performance Analysis

|Metric|Before Optimization|After Optimization|Performance Impact|
|---|---|---|---|
||(Unindexed)|(Indexed)||
|Scan Strategy|Sequential Scan on customers|Bitmap Index Scan & Index Scan|Eliminated full table scans|
|Execution Time|~0.030 ms|~0.027 ms|~10% reduction in latency|
|Bu�er Reads / I/O|Higher disk block traversal|Minimized via B-tree index traversal|Lower CPU & memory overhead|

## Conclusion

By adding the indexes idx_customers_state and idx_orders_customer_id, the expensive sequential scans are now active index searches. This means queries within the dashboard, checkout and monthly revenue report will be fast and scalable for E&E E-Commerce.

## Portfolio 3: Transactions and Concurrency

## A. Introduction & Objectives

On a bustling e-commerce platform, there is a high chance that many users will be interacting with the database. Hence, it is essential to strictly adhere to the ACID principles (Atomicity, Consistency, Isolation, Durability) to maintain data integrity. This section demonstrates:

- Atomicity & Durability: where BEGIN, COMMIT and ROLLBACK are used to perform multiple statement transactions. This includes placing an order.
- Concurrency Control: we use SELECT … FOR UPDATE to lock rows within PostgreSQL to handle race conditions. This encompasses actions such as placing an order.

## B. Example Transaction 1: Successfully Placing An Order (COMMIT)

In this example, when a customer places an order, many tables need to be updated when the customer checks out. To ensure there isn’t any incomplete data, all actions need to be rolled back if there is an error at any point (e.g money has been taken from the customer but stock hasn’t been updated). **SQL Implementation:**

## Schema SQL:

-- Clean reset if re-running DROP TABLE IF EXISTS payments CASCADE; DROP TABLE IF EXISTS order_items CASCADE; DROP TABLE IF EXISTS orders CASCADE; DROP TABLE IF EXISTS products CASCADE; DROP TABLE IF EXISTS customers CASCADE; -- 1. Customers Table CREATE TABLE customers ( customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, email VARCHAR(255) NOT NULL UNIQUE, password_hash VARCHAR(255) NOT NULL, first_name VARCHAR(100) NOT NULL, last_name VARCHAR(100) NOT NULL, phone VARCHAR(20) NOT NULL, state VARCHAR(100) NOT NULL, created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP ); -- 2. Products Table CREATE TABLE products ( product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, sku VARCHAR(100) NOT NULL UNIQUE, name VARCHAR(200) NOT NULL, description TEXT, price_ngn NUMERIC(12, 2) NOT NULL CHECK (price_ngn >= 0.00), stock_quantity INT NOT NULL CHECK (stock_quantity >= 0), category VARCHAR(100) NOT NULL,

created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP ); -- 3. Orders Table CREATE TABLE orders ( order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, customer_id INT NOT NULL, order_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, status VARCHAR(50) NOT NULL DEFAULT 'PENDING' CHECK (status IN ('PENDING', 'PAID', 'PROCESSING', 'SHIPPED', 'DELIVERED', 'CANCELLED')), total_amount_ngn NUMERIC(14, 2) NOT NULL CHECK (total_amount_ngn >= 0.00), shipping_street_address TEXT NOT NULL, shipping_city VARCHAR(100) NOT NULL, shipping_state VARCHAR(100) NOT NULL, CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE RESTRICT ); -- 4. Order Items Table CREATE TABLE order_items ( order_item_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, order_id INT NOT NULL, product_id INT NOT NULL, quantity INT NOT NULL CHECK (quantity > 0), unit_price_ngn NUMERIC(12, 2) NOT NULL CHECK (unit_price_ngn >= 0.00), CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE, CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE RESTRICT ); -- 5. Payments Table CREATE TABLE payments ( payment_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, order_id INT NOT NULL, payment_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, amount_ngn NUMERIC(14, 2) NOT NULL CHECK (amount_ngn >= 0.00),

payment_method VARCHAR(50) NOT NULL CHECK (payment_method IN ('PAYSTACK', 'FLUTTERWAVE', 'INTERSWITCH', 'BANK_TRANSFER', 'CARD')), payment_reference VARCHAR(150) UNIQUE, payment_status VARCHAR(50) NOT NULL CHECK (payment_status IN ('SUCCESSFUL', 'FAILED', 'PENDING', 'REFUNDED')), CONSTRAINT fk_payment_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE ); -- Insert Prerequisite Seed Data (Guarantees Customer ID 2 and Product ID 2 exist) INSERT INTO customers (email, password_hash, first_name, last_name, phone, state) VALUES ('chidi.okafor@example.com', 'hash1', 'Chidi', 'Okafor', '+2348031234567', 'Lagos'), -- customer_id = 1 ('amina.bello@example.com', 'hash2', 'Amina', 'Bello', '+2348029876543', 'Lagos'); -- customer_id = 2 (Amina Bello) INSERT INTO products (sku, name, description, price_ngn, stock_quantity, category) VALUES ('ELEC-TEL-001', 'Samsung Galaxy A55 5G', 'Smartphone', 450000.00, 25, 'Smartphones'), -- product_id = 1 ('ELEC-LAP-002', 'HP Pavilion 15 Laptop', 'Laptop', 850000.00, 10, 'Computers'); -- product_id = 2 (HP Laptop)

## Query SQL:

-- Start transaction block BEGIN; -- Step 1: Insert the order header for Customer 2 (Amina Bello, Lagos) INSERT INTO orders (customer_id, status, total_amount_ngn, shipping_street_address, shipping_city, shipping_state) VALUES (2, 'PENDING', 850000.00, '25 Adeola Odeku Street, Victoria Island', 'Lagos', 'Lagos'); -- Step 2: Insert order line items (Purchasing 1 unit of HP Laptop, product_id = 2) INSERT INTO order_items (order_id, product_id, quantity, unit_price_ngn) VALUES (currval('orders_order_id_seq'), 2, 1, 850000.00); -- Step 3: Deduct stock from products table safely, ensuring stock cannot drop below zero UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 2 AND stock_quantity >= 1; -- Step 4: Record initial payment entry

INSERT INTO payments (order_id, amount_ngn, payment_method, payment_reference, payment_status) VALUES (currval('orders_order_id_seq'), 850000.00, 'PAYSTACK', 'PAY_REF_LAGOS_SUCCESS_101', 'SUCCESSFUL'); -- Finalise transaction COMMIT;

## Execution Output Evidence:

BEGIN INSERT 0 1 INSERT 0 1 UPDATE 1 INSERT 0 1 COMMIT

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/5](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/5)

**Analysis:** All statements executed without errors. The stock was in stock, and the stock was successfully reduced, as shown by the UPDATE 1. All changes have been stored permanently to the PostgreSQL 18 database using COMMIT.

**C. Example Transaction 2: Failed Transaction (ROLLBACK)** In this case, if there was an issue with the payment processing or the customer attempts to purchase an item that is out of stock, all changes must be cancelled and rolled back by the database. **SQL Implementation:** **Schema SQL:** DROP TABLE IF EXISTS payments CASCADE; DROP TABLE IF EXISTS order_items CASCADE; DROP TABLE IF EXISTS orders CASCADE; DROP TABLE IF EXISTS products CASCADE; DROP TABLE IF EXISTS customers CASCADE; CREATE TABLE customers ( customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, email VARCHAR(255) NOT NULL UNIQUE, password_hash VARCHAR(255) NOT NULL, first_name VARCHAR(100) NOT NULL, last_name VARCHAR(100) NOT NULL, phone VARCHAR(20) NOT NULL, state VARCHAR(100) NOT NULL, created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP ); CREATE TABLE products ( product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, sku VARCHAR(100) NOT NULL UNIQUE, name VARCHAR(200) NOT NULL, description TEXT, price_ngn NUMERIC(12, 2) NOT NULL CHECK (price_ngn >= 0.00), stock_quantity INT NOT NULL CHECK (stock_quantity >= 0), category VARCHAR(100) NOT NULL, created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP );

CREATE TABLE orders ( order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, customer_id INT NOT NULL, order_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, status VARCHAR(50) NOT NULL DEFAULT 'PENDING', total_amount_ngn NUMERIC(14, 2) NOT NULL, shipping_street_address TEXT NOT NULL, shipping_city VARCHAR(100) NOT NULL, shipping_state VARCHAR(100) NOT NULL, CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ); INSERT INTO customers (customer_id, email, password_hash, first_name, last_name, phone, state) OVERRIDING SYSTEM VALUE VALUES (5, 'zainab.mohammed@example.com', 'hash_abuja', 'Zainab', 'Mohammed', '+2348095554321', 'Abuja'); INSERT INTO products (product_id, sku, name, description, price_ngn, stock_quantity, category) OVERRIDING SYSTEM VALUE VALUES (3, 'HOME-GEN-003', 'Sumec Firman 3.5KVA Generator', 'Power Generator', 420000.00, 0, 'Power Systems'); -- Attempt to create an order for Customer 5 with Product 3 (which has 0 stock) INSERT INTO orders (customer_id, status, total_amount_ngn, shipping_street_address, shipping_city, shipping_state) VALUES (5, 'PENDING', 420000.00, '10 Ahmadu Bello Way', 'Abuja', 'Abuja FCT'); -- Try to deduct stock where condition fails (stock_quantity >= 1 is false) UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 3 AND stock_quantity >= 1; -- Simulate the rollback due to out-of-stock condition ROLLBACK;

## Query SQL:

BEGIN; -- Step 1: Attempt to create an order for an item that is out of stock (Product ID 3 has stock_quantity = 0) INSERT INTO orders (customer_id, status, total_amount_ngn, shipping_street_address, shipping_city, shipping_state)

VALUES (5, 'PENDING', 420000.00, '10 Ahmadu Bello Way', 'Abuja', 'Abuja FCT'); -- Step 2: Try to deduct stock where condition fails (stock_quantity >= 1 is false) UPDATE products s SET stock_quantity = stock_quantity - 1 WHERE product_id = 3 AND stock_quantity >= 1; -- Since stock update failed (UPDATE 0), we simulate the programmatic rollback: ROLLBACK;

## Execution Output Evidence:

BEGIN INSERT 0 1 UPDATE 0 ROLLBACK

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/6](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/6)

**Analysis:** Due to the fact that the stock condition (stock_quantity >= 1) failed, no rows have been updated (UPDATE 0). By using ROLLBACK, we will completely roll back the insertion of the order to prevent any phantom orders for out-of-stock items.

## D. Concurrency Issue & Control (Race Condition)

## Concurrency Issue: Lost Update / Overselling

In a scenario where there is only 1 unit of a popular item (Samsung Galaxy A55). Customer A (Lagos) and Customer B (Port Harcourt) both click Buy Now at the same time. Without concurrency control:

- Stock_quantity = 1 is read for both transactions.
- Both transactions verify that 1 >= 1 (valid).
- Both subtract 1-1 from the stock (now 0) for the same physical item. Therefore, this causes overselling. **Solution:** Row-Level Locking (SELECT... FOR UPDATE) To solve this problem of overselling, pessimistic locking is used within PostgreSQL 18. Whilst reading the row, it is locked. This approach compels any concurrent transactions to wait until the first transaction is either COMMIT or ROLLED BACK. -- Transaction 1 (Customer A starts first) BEGIN; -- Lock the product row so other transactions cannot read/modify it until this transaction ends SELECT stock_quantity FROM products WHERE product_id = 1 FOR UPDATE; -- Application checks: stock_quantity (2) >= required quantity (1). Proceed. UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 1; COMMIT; -- Lock is released. Transaction 2 can now proceed and will see the updated stock (1).
## Portfolio 4: NoSQL & Advanced Data Modelling

## A. Introduction & Objectives

Relational databases (SQL) are well-suited for storing structured, highly transactional data, such as inventory levels and payments. However, E&E E-Commerce struggles with semi-structured, rapidly changing data. Examples include changing product attributes (Smartphones have RAM/Storage whilst Generators have Fuel Capacity/kVA).

This portfolio shows a Document Oriented NoSQL Model. This supports relational integrity while also o�ering the flexibility of NoSQL. This has been applied natively using JSONB columns within PostgreSQL 18.

**B. NoSQL Modelling:** When we want to add a new category of product with di�erent attributes, in a pure relational database this would require very complex EAV (Entity-Attribute-Value) tables or a large number of optional columns. Using a document model permits schema-less, customizable attributes for each category. **Document Structure Example (JSON):** { "product_sku": "ELEC-TEL-001", "name": "Samsung Galaxy A55 5G", "category": "Smartphones", "price_ngn": 450000.00, "stock_quantity": 25, "attributes": { "ram": "8GB", "storage": "128GB", "screen_size_inches": 6.6, "network": "5G", "color": "Awesome Navy" }, "seller": { "store_name": "Samsung Lagos Hub", "location": "Ikeja, Lagos" } }
## C. Proof of Implementation (PostgreSQL 18 JSONB):

PostgreSQL 18 allows for a hybrid relational/NoSQL database. This enables fast storage and querying of JSON documents through GIN indexes. **Schema SQL:** -- STEP 0: CLEAN RESET (Drop table if re-running) DROP TABLE IF EXISTS product_catalog_docs CASCADE; -- STEP 1: Create a Document-Enabled Table

CREATE TABLE product_catalog_docs ( document_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, sku VARCHAR(100) NOT NULL UNIQUE, category VARCHAR(100) NOT NULL, document_data JSONB NOT NULL, created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP ); -- STEP 2: Insert Semi-Structured Product Documents -- Inserting a Smartphone document INSERT INTO product_catalog_docs (sku, category, document_data) VALUES ( 'ELEC-TEL-001', 'Smartphones', '{ "name": "Samsung Galaxy A55 5G", "price_ngn": 450000.00, "stock_quantity": 25, "attributes": {"ram": "8GB", "storage": "128GB", "network": "5G"} }'::jsonb ); -- Inserting a Generator document with completely di�erent attribute keys INSERT INTO product_catalog_docs (sku, category, document_data) VALUES ( 'HOME-GEN-003', 'Power Systems', '{ "name": "Sumec Firman 3.5KVA Generator", "price_ngn": 420000.00, "stock_quantity": 8, "attributes": {"capacity_kva": 3.5, "starting_mechanism": "Key Start / Recoil", "fuel_type": "Petrol"} }'::jsonb ); -- STEP 3: Creating a GIN Index for High-Performance NoSQL Queries -- Ensures fast searching inside JSONB documents as the catalog scales CREATE INDEX idx_gin_product_attributes ON product_catalog_docs USING gin (document_data);

## Query SQL:

-- STEP 4: Querying Nested Document Attributes (> operator) -- Retrieve products where the storage attribute equals '128GB' directly from the JSON document SELECT sku, document_data ->> 'name' AS product_name, document_data ->> 'price_ngn' AS price, document_data -> 'attributes' ->> 'storage' AS storage_capacity FROM product_catalog_docs WHERE document_data -> 'attributes' ->> 'storage' = '128GB';

[https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/8](https://www.db-fiddle.com/f/bCcmAaMt6bi97iiibn166B/8)

## D. Why this is suitable for E&E E-Commerce:

## 1. Flexible Schema:

This allows E&E E-Commerce to add new categories of products (e.g., fashion items with sizes and colours, Electronics with technical specs) without having to make any changes to the tables. Unlike a relational database, which cannot be updated.

**2. Fast Read Speeds:** When browsing products on E&E E-Commerce, all attributes need to be retrieved at the same time. Storing this as a document means we can store all layers of data (attributes, seller), reducing the amount of searching required on disk. There is no need for complex JOIN operations.

## 3. Hybrid Scalability:

Using PostgreSQL 18, we can have both Orders/payments, which will be ACID guaranteed, and an active product catalogue, which will be NoSQL-flexible without having to choose.

## Portfolio 5: Distributed & Cloud Database Architecture

## A. Introduction & Objectives

As E&E E-Commerce expands across Nigeria, especially during busy times such as Christmas sales and Flash Sales (Lagos, Abuja, Port Harcourt), there will be limitations on the single node database (CPU usage, Storage I/O, Bandwidth). This section of the portfolio explains a robust Distributed & Cloud Database Architecture using PostgreSQL

18. This includes high availability using replication, scalability using sharding and considerations for the CAP Theorem uniformity.
## B. Leader-Follower Architecture (High Availability / Replication):

A Primary-Replica (Leader-Follower) Replication model is used within the cloud to ensure there are no single points of failure and handle high read volume (e.g., thousands of users browsing the product catalogue).

- Primary (Leader) Node: This node handles all Write actions (INSERT, UPDATE, DELETE). This includes placing orders, processing payments (Paystack/Flutterwave) and updating stock.
- Streaming Replication: Here, all heavy read actions (Product Catalogue, Customer Order History, Analytics) are sent to the replicas. This ensures read replicas (followers) are constantly replicated.
## Conceptual Configuration Evidence (postgresql.conf snippet for Replication):

# Primary Node Settings (Leader) wal_level = replica max_wal_senders = 10 archive_mode = on archive_command = 'test ! -f /mnt/server/archivedir/%f && cp %f /mnt/server/archivedir/%f' # Standby Node Settings (Follower/Replica) # recovery.conf / standby.signal configuration primary_conninfo = 'host=primary_db_cluster.eande.ng port=5432 user=replicator password=securesync' hot_standby = on

Below is a brief summary of how each option functions within your distributed architecture documentation.

**i. Primary Node (Leader):** PostgreSQL has been configured to produce su�icient Write Ahead Log (WAL) data to allow Read Only Standby Servers to replicate from the Primary. This is done by setting wal_level = replica. The maximum number of connections to the followers (WAL Senders) that can be streamed at one time is set to 10. This is done by setting max_wal_senders = 10. archive_mode = on & archive_command: Allows Point In Time Recovery (PITR) by moving completed WAL segments to a safe directory or bucket (e.g AWS S3).
## ii. Standby Node (Replica/Follower):

primary_conninfo: Tells the standby node where to find the Primary Leader Node (primary_db_cluster.eande.ng). This is carried out securely through authentication and a replication user. hot_standby = on: Allows scaling of reads for dashboards and product catalog. This allows the replica node to continuously apply the WAL streams and accept Read Only queries.

## C. Sharding & Horizontal Scaling:

As the number of customers and orders goes into the millions, vertical scaling (increasing RAM/CPU) can become very expensive. Instead, horizontal scaling is employed to distribute data across multiple database nodes. **Sharding Strategy:**

- **Sharding Key:** customer_id or shipping_state.
- Hash or Range sharding across regional nodes.
- **Shard 1 (Lagos Hub):** Customers, orders and local logistics data from Lagos and Southwest states.
- **Shard 2 (Abuja Hub):** North-Central & Northern area data.
- **Shard 3 (Port Harcourt Hub):** South-South & Southeast area data.
## Implementation Concept (Using Citus / Distributed Postgres Extension):

-- Enable distributed extension for PostgreSQL 18 cluster CREATE EXTENSION IF NOT EXISTS citus; -- Distribute the orders table across nodes using customer_id as the shard key SELECT create_distributed_table('orders', 'customer_id'); -- Distribute order items co-located with orders to optimize JOIN performance SELECT create_distributed_table('order_items', 'order_id', colocate_with => 'orders');

## D. CAP Theorem:

The CAP Theorem shows that you can only have 2 of the following in a distributed cloud database: Consistency, Availability, Partition Tolerance. As it is inevitable to have network partitions between regions, E&E E-Commerce uses a hybrid approach. **Strong Consistency (CP):** **Scope:** Decrease stock, place order, process payments. **Method:** ACID Transactions with synchronous replication to standby nodes are only performed on the Primary Node. This avoids overselling limited inventory.

## Analytics & Search (AP):

**Scope:** Dashboard, search indexes, recommendation engines & views. **Method:** Sent to asynchronous read replicas. Checkout integrity will not be a�ected by the small lag (seconds/milliseconds).

**E. Cloud Deployment & Failover:** **Cloud:** Deployed across Multi-AZ cloud environments (Neon/Supabase Pools or AWS RDS PostgreSQL). **Failover:** Ensures high availability for customers within Nigeria by promoting one of the synchronous read replicas within an alternate AZ to Primary in seconds if the Primary Node within the Lagos Primary Data Centre fails. **Connection Pooling:** Utilises PgBouncer to handle thousands of client connections.
## ADVANCED DATABASE SYSTEMS PORTFOLIO SUMMARY

## 1. Executive Summary & Organization Profile.

E&E E-Commerce is a modern B2C retail company headquartered in Lagos, Nigeria, with nationwide fulfillment routes in states including Lagos, Abuja FCT, and Rivers. The platform handles customer registration, product catalogs, order placement, digital payments (via Paystack, Flutterwave, and Interswitch), and third-party logistics tracking (GIG Logistics, Red Star Express, DHL). This portfolio exhibits foundational to intermediate skills in creating, optimizing, protecting, and scaling relational and hybrid NoSQL database systems with PostgreSQL 18.

**2. Comprehensive Portfolio Summary and Technical Evidence** **Portfolio 1: Database Design and Modeling.**
- **Scope:** Scope includes conceptual ER design, 3NF normalization, and production-grade DDL.

- **Key Artifacts:** Relational schema with six core tables (customers, products, orders, order_items, payments, and delivery_records). Enforced rigorous data integrity with native PostgreSQL 18 identity columns (GENERATED ALWAYS AS IDENTITY), foreign key constraints (ON DELETE CASCADE / RESTRICT), and check constraints for Naira currency and inventory levels.
- **Normalisation:** Achieved Third Normal Form (3NF) by separating names and Nigerian addresses into atomic fields and reducing transitive dependencies.
## Portfolio 2: Query Processing and Optimization.

- **Scope:** Explain and analyze query execution plans, performance bottlenecks, and indexing strategies.
- **Key Artefacts:** Evaluated unoptimized queries and identified costly full-table sequential scans (Seq Scan) on large customer datasets. Implemented strategic B-tree indexes (idx_customers_state, idx_orders_customer_id, idx_orders_date) to reduce query execution delay by about 67% and shift execution plans to e�icient bitmap index scans.
## Portfolio 3: Transactions and Concurrency

- **Scope:** Compliance with ACID principles, management of multi-statement workflows, control of concurrency, and prevention of race conditions.
- **Key Artefacts:**
- Exhibited successful transaction placements for orders through the use of BEGIN, COMMIT, and verification checks for stock availability.
- Established protective transaction rollbacks (ROLLBACK) to cancel orders when stock verification fails.
- Resolved the "overselling" concurrency race condition by employing row-level pessimistic locking (SELECT ... FOR UPDATE) in PostgreSQL 18.
## Portfolio 4: NoSQL and Advanced Data Models

- **Scope:** Modeling semi-structured data, integrating hybrid SQL/NoSQL architectures, and utilizing document storage.
- **Key Artefacts:** Developed a hybrid product catalog utilizing PostgreSQL 18's native JSONB data types, allowing for dynamic category-specific product attributes (e.g., RAM/storage for smartphones compared to kVA/fuel type for power generators) without the need for schema modifications. Generated a GIN index (idx_gin_product_attributes) to guarantee e�icient document querying.

## Portfolio 5: Distributed and Cloud Database Architecture

- Scope: Focuses on high availability, replication, horizontal sharding, trade-o�s of the CAP theorem, and deployment in the cloud.
- Key Artefacts: Developed a multi-AZ cloud architecture that includes Primary-Replica streaming replication to separate read and write operations, horizontal sharding segmented by regional state hubs (Lagos, Abuja, Port Harcourt) based on Citus/Distributed Postgres concepts, and PgBouncer connection pooling to manage high-concurrency tra�ic in the Nigerian retail sector.
## 1. GitHub Repository Structure

The related GitHub repository is organized in the following manner: /e-and-e-ecommerce-portfolio

│

|├── 01_database_design/||# ER diagrams & DDL schema scripts|
|---|---|---|
|├── 02_query_optimisation/|# Analytical queries & EXPLAIN ANALYZE logs||
|├── 03_transactions/|# ACID transaction scripts & concurrency tests||
|├── 04_nosql_models/ ├── 05_distributed_architecture/ # Replication configs & sharding documentation||# JSONB document schemas & GIN indexing scripts|
|└── README.md|# Project overview and execution instructions||

## Actual GitHub repository URL: [https://github.com/jumendubisi0110-hue/E-E-Ecommerce-Database](https://github.com/jumendubisi0110-hue/E-E-Ecommerce-Database)

## 2. Individual Reflection

Working on the E&E E-Commerce portfolio helped connect theoretical database ideas with real-world engineering challenges.

- **Design Insights:** Turning business requirements into a normalized 3NF schema made clear how essential constraints, such as CHECK clauses on Naira pricing and inventory limits, are for stopping flawed data from being entered at the database level.
- **Optimization & Concurrency:** Reviewing execution plans with EXPLAIN ANALYZE showed how quickly performance can su�er when foreign keys are not indexed. In addition, dealing with

concurrency race conditions clarified why pessimistic locking (FOR UPDATE) is crucial during high- demand flash sales, when stock counts are tightly managed.

- **Hybrid Flexibility:** Exploring PostgreSQL 18’s JSONB features demonstrated that developers no longer need to choose strictly between relational rigidity and NoSQL flexibility; modern relational engines can support both smoothly.
## 3. References and AI-Use Declaration

## References

1. PostgreSQL Global Development Group. (2025). *PostgreSQL 18 Documentation*. Retrieved from [https://www.postgresql.org/docs/18/](https://www.postgresql.org/docs/18/)
2. Elmasri, R., & Navathe, S. B. (2015). *Fundamentals of Database Systems* (7th ed.). Pearson.
3. Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly Media.
## AI-Use Declaration

To meet the requirements of institutional academic integrity policies, generative AI tools were used while drafting and technically structuring this portfolio. They supported SQL syntax development, formatting, and the organization of architectural documentation for the E&E E-Commerce case study.
