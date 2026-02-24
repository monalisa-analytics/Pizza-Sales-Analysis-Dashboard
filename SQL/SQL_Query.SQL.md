/* =====================================================
   DATABASE & TABLE CREATION
   ===================================================== */

CREATE DATABASE pizzadb;
USE pizzadb;

CREATE TABLE pizza_sales (
    order_details_id INT,
    order_id INT,
    pizza_id VARCHAR(50),
    quantity INT,
    order_date DATE,
    order_time TIME,
    unit_price DECIMAL(10,2),
    total_price DECIMAL(10,2),
    pizza_size VARCHAR(10),
    pizza_category VARCHAR(50),
    pizza_ingredients TEXT,
    pizza_name VARCHAR(100)
);

/* =====================================================
   KPI METRICS
   ===================================================== */

-- Total Revenue
SELECT 
    SUM(total_price) AS total_revenue
FROM pizza_sales;

-- Total Orders
SELECT 
    COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales;

-- Total Pizzas Sold
SELECT 
    SUM(quantity) AS total_pizzas_sold
FROM pizza_sales;

-- Average Order Value
SELECT 
    ROUND(SUM(total_price) / COUNT(DISTINCT order_id), 2) AS avg_order_value
FROM pizza_sales;

-- Average Pizzas per Order
SELECT 
    ROUND(SUM(quantity) / COUNT(DISTINCT order_id), 2) AS avg_pizzas_per_order
FROM pizza_sales;

/* =====================================================
   TIME-BASED ANALYSIS
   ===================================================== */

-- Orders by Hour
SELECT 
    HOUR(order_time) AS order_hour,
    COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales
GROUP BY order_hour
ORDER BY total_orders DESC;

-- Revenue by Hour
SELECT 
    HOUR(order_time) AS order_hour,
    SUM(total_price) AS total_revenue
FROM pizza_sales
GROUP BY order_hour
ORDER BY total_revenue DESC;

-- Orders by Day of Week
SELECT 
    DAYNAME(order_date) AS day_name,
    COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales
GROUP BY day_name
ORDER BY total_orders DESC;

-- Monthly Revenue Trend
SELECT 
    MONTH(order_date) AS month_no,
    MONTHNAME(order_date) AS month_name,
    ROUND(SUM(total_price), 2) AS monthly_revenue
FROM pizza_sales
GROUP BY month_no, month_name
ORDER BY month_no;

/* =====================================================
   PRODUCT PERFORMANCE
   ===================================================== */

-- Top 5 Best-Selling Pizzas
SELECT 
    pizza_name,
    SUM(quantity) AS total_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_sold DESC
LIMIT 5;

-- Bottom 5 Worst-Selling Pizzas
SELECT 
    pizza_name,
    SUM(quantity) AS total_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_sold ASC
LIMIT 5;

-- Top 5 Revenue Generating Pizzas
SELECT 
    pizza_name,
    ROUND(SUM(total_price), 2) AS revenue
FROM pizza_sales
GROUP BY pizza_name
ORDER BY revenue DESC
LIMIT 5;

-- Revenue Contribution by Pizza Size
SELECT 
    pizza_size,
    ROUND(
        SUM(total_price) * 100 / (SELECT SUM(total_price) FROM pizza_sales),
        2
    ) AS revenue_percentage
FROM pizza_sales
GROUP BY pizza_size
ORDER BY revenue_percentage DESC;

-- Pizzas Sold by Category
SELECT 
    pizza_category,
    SUM(quantity) AS total_pizzas_sold
FROM pizza_sales
GROUP BY pizza_category
ORDER BY total_pizzas_sold DESC;

/* =====================================================
   OPERATIONAL INSIGHTS
   ===================================================== */

-- Kitchen Load (Pizzas Made per Hour)
SELECT 
    HOUR(order_time) AS order_hour,
    SUM(quantity) AS pizzas_made
FROM pizza_sales
GROUP BY order_hour
ORDER BY pizzas_made DESC;

-- Seat Utilization Percentage
-- Assumptions: Seats = 60, Customers per Order = 2
SELECT 
    order_hour,
    ROUND((orders * 2 / 60) * 100, 2) AS seat_utilization_pct
FROM (
    SELECT 
        HOUR(order_time) AS order_hour,
        COUNT(DISTINCT order_id) AS orders
    FROM pizza_sales
    GROUP BY order_hour
) t
ORDER BY seat_utilization_pct DESC;

-- Busiest Days
SELECT 
    DAYNAME(order_date) AS day_name,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(SUM(total_price), 2) AS total_revenue
FROM pizza_sales
GROUP BY day_name
ORDER BY total_orders DESC;

-- Busiest Hours
SELECT 
    HOUR(order_time) AS order_hour,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(SUM(total_price), 2) AS total_revenue
FROM pizza_sales
GROUP BY order_hour
ORDER BY total_orders DESC;

-- Operational Peak (Max Pizzas in a Single Hour)
SELECT 
    order_date,
    HOUR(order_time) AS order_hour,
    SUM(quantity) AS pizzas_made
FROM pizza_sales
GROUP BY order_date, order_hour
ORDER BY pizzas_made DESC
LIMIT 5;
