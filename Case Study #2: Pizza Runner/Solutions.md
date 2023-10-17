# 🍕 [Case Study #2: Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

## A. Pizza Metrics
**Schema (PostgreSQL v13)**

    CREATE SCHEMA pizza_runner;
    SET search_path = pizza_runner;
    
    DROP TABLE IF EXISTS runners;
    CREATE TABLE runners (
      "runner_id" INTEGER,
      "registration_date" DATE
    );
    INSERT INTO runners
      ("runner_id", "registration_date")
    VALUES
      (1, '2021-01-01'),
      (2, '2021-01-03'),
      (3, '2021-01-08'),
      (4, '2021-01-15');
    
    
    DROP TABLE IF EXISTS customer_orders;
    CREATE TABLE customer_orders (
      "order_id" INTEGER,
      "customer_id" INTEGER,
      "pizza_id" INTEGER,
      "exclusions" VARCHAR(4),
      "extras" VARCHAR(4),
      "order_time" TIMESTAMP
    );
    
    INSERT INTO customer_orders
      ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
    VALUES
      ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
      ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
      ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
      ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
      ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
      ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
      ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
      ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
      ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
      ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
      ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');
    
    
    DROP TABLE IF EXISTS runner_orders;
    CREATE TABLE runner_orders (
      "order_id" INTEGER,
      "runner_id" INTEGER,
      "pickup_time" VARCHAR(19),
      "distance" VARCHAR(7),
      "duration" VARCHAR(10),
      "cancellation" VARCHAR(23)
    );
    
    INSERT INTO runner_orders
      ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
    VALUES
      ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
      ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
      ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
      ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
      ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
      ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
      ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
      ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
      ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
      ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');
    
    
    DROP TABLE IF EXISTS pizza_names;
    CREATE TABLE pizza_names (
      "pizza_id" INTEGER,
      "pizza_name" TEXT
    );
    INSERT INTO pizza_names
      ("pizza_id", "pizza_name")
    VALUES
      (1, 'Meatlovers'),
      (2, 'Vegetarian');
    
    
    DROP TABLE IF EXISTS pizza_recipes;
    CREATE TABLE pizza_recipes (
      "pizza_id" INTEGER,
      "toppings" TEXT
    );
    INSERT INTO pizza_recipes
      ("pizza_id", "toppings")
    VALUES
      (1, '1, 2, 3, 4, 5, 6, 8, 10'),
      (2, '4, 6, 7, 9, 11, 12');
    
    
    DROP TABLE IF EXISTS pizza_toppings;
    CREATE TABLE pizza_toppings (
      "topping_id" INTEGER,
      "topping_name" TEXT
    );
    INSERT INTO pizza_toppings
      ("topping_id", "topping_name")
    VALUES
      (1, 'Bacon'),
      (2, 'BBQ Sauce'),
      (3, 'Beef'),
      (4, 'Cheese'),
      (5, 'Chicken'),
      (6, 'Mushrooms'),
      (7, 'Onions'),
      (8, 'Pepperoni'),
      (9, 'Peppers'),
      (10, 'Salami'),
      (11, 'Tomatoes'),
      (12, 'Tomato Sauce');

---

**Query #1**

    SELECT COUNT(order_id) as total_orders
    FROM pizza_runner.customer_orders;

| total_orders |
| ------------ |
| 14           |

---
**Query #2**

    SELECT COUNT(DISTINCT(order_id)) as unique_orders
    FROM pizza_runner.customer_orders;

| unique_orders |
| ------------- |
| 10            |

---
**Query #3**

    SELECT runner_id,
        COUNT(order_id) as num_orders
    FROM pizza_runner.runner_orders
    WHERE pickup_time is not null
        AND pickup_time != 'null'
    GROUP BY runner_id
    ORDER BY runner_id;

| runner_id | num_orders |
| --------- | ---------- |
| 1         | 4          |
| 2         | 3          |
| 3         | 1          |

---
**Query #4**

    SELECT p.pizza_name as pizza_type,
        count(c.pizza_id) as quantity
    FROM pizza_runner.customer_orders c
        JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
        JOIN pizza_runner.pizza_names p ON p.pizza_id = c.pizza_id
    WHERE pickup_time is not null
        AND pickup_time != 'null'
    GROUP BY c.pizza_id,
        p.pizza_name;

| pizza_type | quantity |
| ---------- | -------- |
| Meatlovers | 9        |
| Vegetarian | 3        |

---
**Query #5**

    SELECT c.customer_id,
        p.pizza_name,
        count(c.pizza_id) as quantity
    FROM pizza_runner.customer_orders c
        JOIN pizza_runner.pizza_names p ON c.pizza_id = p.pizza_id
    GROUP BY c.customer_id,
        p.pizza_name
    ORDER BY c.customer_id,
        p.pizza_name;

| customer_id | pizza_name | quantity |
| ----------- | ---------- | -------- |
| 101         | Meatlovers | 2        |
| 101         | Vegetarian | 1        |
| 102         | Meatlovers | 2        |
| 102         | Vegetarian | 1        |
| 103         | Meatlovers | 3        |
| 103         | Vegetarian | 1        |
| 104         | Meatlovers | 3        |
| 105         | Vegetarian | 1        |

---
**Query #6**

    WITH num_pizza_perorder as (
        SELECT count(c.order_id) as quantity
        FROM pizza_runner.customer_orders c
            LEFT JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
        WHERE r.pickup_time is not null
            AND r.pickup_time != 'null'
        GROUP BY c.order_id
    )
    SELECT max(quantity)
    FROM num_pizza_perorder;

| max |
| --- |
| 3   |

---
**Query #7**

    SELECT c.customer_id,
        SUM(
            CASE
                WHEN c.exclusions <> ''
                OR c.extras <> '' THEN 1
                ELSE 0
            END
        ) AS at_least_1_change,
        SUM(
            CASE
                WHEN c.exclusions = ''
                AND c.extras = '' THEN 1
                ELSE 0
            END
        ) AS no_change
    FROM pizza_runner.customer_orders c
        LEFT JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
    WHERE r.pickup_time is not null
        AND r.pickup_time != 'null'
    GROUP BY c.customer_id
    ORDER BY c.customer_id;

| customer_id | at_least_1_change | no_change |
| ----------- | ----------------- | --------- |
| 101         | 0                 | 2         |
| 102         | 1                 | 1         |
| 103         | 3                 | 0         |
| 104         | 3                 | 0         |
| 105         | 1                 | 0         |

---
**Query #8**

    SELECT SUM(
            CASE
                WHEN exclusions is not null
                and extras is not null THEN 1
                ELSE 0
            END
        ) as exclusion_and_extra
    FROM pizza_runner.customer_orders c
        LEFT JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
    WHERE r.pickup_time is not null
        AND r.pickup_time != 'null'
        AND exclusions <> ''
        AND extras <> ''
        AND exclusions != 'null'
        AND extras != 'null';

| exclusion_and_extra |
| ------------------- |
| 1                   |

---
**Query #9**

    SELECT extract(
            hour
            from order_time
        ) as hour_,
        count(order_id) as pizza_count
    FROM pizza_runner.customer_orders
    GROUP BY hour_
    ORDER BY hour_;

| hour_ | pizza_count |
| ----- | ----------- |
| 11    | 1           |
| 13    | 3           |
| 18    | 3           |
| 19    | 1           |
| 21    | 3           |
| 23    | 3           |

---
**Query #10**

    SELECT TO_CHAR(order_time, 'DAY') AS day_of_week,
        COUNT(pizza_id) AS pizza_count
    FROM pizza_runner.customer_orders
    GROUP BY day_of_week
    ORDER BY day_of_week;

| day_of_week | pizza_count |
| ----------- | ----------- |
| FRIDAY      | 1           |
| SATURDAY    | 5           |
| THURSDAY    | 3           |
| WEDNESDAY   | 5           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
