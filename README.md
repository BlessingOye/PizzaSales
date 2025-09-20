--A1: What is the total number of orders placed? 
SELECT COUNT (DISTINCT order_id) AS TotalOrders
FROM order_details

--A2: What is the total revenue generated from pizza sales? 
SELECT pizzas.pizza_id, pizzas.price, order_details.quantity
FROM pizzas
INNER JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id

SELECT SUM(pizzas.price * order_details.quantity) AS Revenue
FROM pizzas
INNER JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id

--A3: What is the most common pizza size ordered? 
SELECT DISTINCT size
FROM pizzas

UPDATE pizzas
SET size = CASE
	WHEN size = 'S' THEN 'Small'
	WHEN size = 'M' THEN 'Medium'
	WHEN size = 'L' THEN 'Large'
	WHEN size = 'XL' THEN 'ExtraLarge'
	WHEN size = 'XXL' THEN 'ExtraExtraLarge'
	ELSE size
END;

SELECT DISTINCT pizzas.size, SUM(order_details.quantity) AS TotalQnty
FROM pizzas
INNER JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY TotalQnty DESC;

SELECT DISTINCT pizzas.size, SUM(order_details.quantity) AS TotalQnty, COUNT(DISTINCT order_id) 
FROM pizzas
INNER JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY TotalQnty DESC;

--A4: What are the top 3 most ordered pizza types based on revenue 
SELECT TOP 3 pizza_type_id, SUM(pizzas.price * order_details.quantity) AS TotalRevenue
FROM pizzas
INNER JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_type_id
ORDER BY TotalRevenue DESC;

--A5: Group the orders by date and calculate the average number of pizzas ordered per day.
WITH daily_qnty_per_date AS
(
SELECT date, SUM(quantity) AS TotalQnty
FROM orders
JOIN order_details
ON order_details.order_id = orders.order_id
GROUP BY date
)
SELECT AVG(TotalQnty) AS Avg_orders_daily
FROM daily_qnty_per_date;

--B1: What is the distribution of orders by hour of the day (at which time the orders are maximum? 
SELECT DATEPART(HOUR, time) AS Hour_of_day,
	COUNT(DISTINCT order_id) AS no_of_orders
FROM orders
GROUP BY DATEPART(HOUR, time)
ORDER BY no_of_orders DESC;

--C2: What are the peak hours and days for orders?  
SELECT 
	DATENAME(WEEKDAY,date) AS 'Days of the week',
	DATEPART (Hour, time) AS Hour_of_day,
	COUNT( DISTINCT order_id) AS no_of_orders
FROM orders
GROUP BY 
	DATENAME(WEEKDAY,date),
	DATEPART (Hour, time)
ORDER BY no_of_orders DESC;

--C3: What are the top 5 most ordered pizza types along with their quantities. 
/* WE need NAME in PIZZA_TYPES table, We need QUANTITY in ORDER_DETAILS table.
JOIN PIZZA_ID in PIZZAS table to PIZZA_ID in ORDER_DETAILS TABLE 
JOIN PIZZA_TYPE_ID in PIZZAS table to PIZZA_TYPE_ID in PIZZA_TYPES table.
*/
SELECT TOP 5 pizza_types.name, SUM(order_details.quantity) AS TotalQnty
FROM order_details
JOIN pizzas
ON pizzas.pizza_id = order_details.pizza_id
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY TotalQnty DESC;

--C3: What is the total quantity of each pizza category ordered
SELECT DISTINCT Category
FROM pizza_types

SELECT pizza_types.category, SUM(order_details.quantity) AS TotalQnty
FROM pizza_types
JOIN pizzas
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details
ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.category
ORDER BY TotalQnty DESC;

--C4:What is the category-wise distribution of pizzas 
SELECT Category, COUNT(DISTINCT pizza_type_id) AS No_of_pizzas
FROM pizza_types
GROUP BY CATEGORY
ORDER BY No_of_pizzas DESC;

--D1: Percentage Contribution of Each Pizza Type to Total Revenue
WITH total_revenue AS
(
SELECT pizza_types.name,
	SUM(pizzas.price * order_details.quantity) AS revenue
FROM pizza_types
JOIN pizzas
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
)
SELECT name, CONCAT(CAST((revenue * 100)/SUM(revenue) OVER() AS decimal(5,2)),'%') AS '% Contribution'
FROM total_revenue;

--D2: Analyze the cummulative revenue generated over time.

