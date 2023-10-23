# 🍕 [Case Study #2: Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

## B. Runner and Customer Experience
```sql
-- How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
SELECT count(pizza_id) as total_pizza
FROM pizza_runner.customer_orders_temp;

-- What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
WITH avg_time as (
	SELECT DISTINCT(c.order_id), (pickup_time-order_time) as diff
	FROM pizza_runner.customer_orders_temp c
	LEFT JOIN pizza_runner.runner_orders_temp r
	ON c.order_id = r.order_id
	WHERE distance != 0
	)
SELECT Date_trunc('minute', avg(diff)+'30 second')
FROM avg_time;

-- Is there any relationship between the number of pizzas and how long the order takes to prepare?
WITH orders as (
	SELECT c.order_id, count(c.order_id) as num_pizza, (pickup_time-c.order_time) as diff
	FROM pizza_runner.customer_orders_temp c
	LEFT JOIN pizza_runner.runner_orders_temp r
	ON c.order_id = r.order_id
	WHERE distance != 0
	GROUP BY c.order_id, pickup_time, order_time
	ORDER BY c.order_id, num_pizza
)
SELECT num_pizza, avg(diff)
FROM orders
GROUP BY num_pizza;

-- What was the average distance travelled for each customer?
SELECT customer_id, ROUND(avg(distance),3) as average
FROM pizza_runner.customer_orders_temp c
JOIN pizza_runner.runner_orders_temp r
ON c.order_id = r.order_id
WHERE distance!= 0
GROUP BY customer_id;

-- What was the difference between the longest and shortest delivery times for all orders?
SELECT MAX(duration)-MIN(duration) as diff
FROM pizza_runner.runner_orders_temp;

-- What was the average speed for each runner for each delivery and do you notice any trend for these values?
SELECT 
  r.runner_id, 
  c.customer_id, 
  c.order_id,  
  r.distance, (r.duration / 60) AS duration_hr , 
  ROUND((r.distance/r.duration * 60), 2) AS avg_speed
FROM pizza_runner.runner_orders_temp AS r
JOIN pizza_runner.customer_orders_temp AS c
  ON r.order_id = c.order_id
WHERE distance != 0
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY c.order_id;

-- What is the successful delivery percentage for each runner?
SELECT runner_id, round(count(distance)::numeric/count(runner_id),2) *100 as percentage
FROM pizza_runner.runner_orders_temp
GROUP BY runner_id;
```
