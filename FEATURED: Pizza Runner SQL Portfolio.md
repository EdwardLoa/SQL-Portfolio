# Case Study - Pizza Runner Store

Greetings!

Thank you for visiting my GitHub page.  
In this repository, I showcase a case study from 'data with Danny' where I leverage MySQL 8.0 Workbench to handle business queries, demonstrating my skills in table editing, nested queries, and data transformation. The further down the questions, the more complicated the queries are.

To view the complete case study, please visit [this page](https://8weeksqlchallenge.com/case-study-2/).

---
## Part 0. Tables Creation

```sql
CREATE SCHEMA pizza_runner;
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
```

<br>


---
## Part 1. Data Cleaning

### 1.1. Cleaning Pizza Recipes Table

<br>
Converting these tables;


**Pizza_recipes**

| pizza_id | toppings |
| --- | --- |
|'1'| '1, 2, 3, 4, 5, 6, 8, 10'|
|'2'| '4, 6, 7, 9, 11, 12'|



**Pizza_names** 

| pizza_id | pizza_name |
| --- | --- |
| 1 | Meatlovers |
| 2	| Vegetarian | 

**Pizza_toppings**

| topping_id | topping_name |
| --- | --- |
| 1 |	Bacon |
| 2 |	BBQ Sauce |
| 3 |	Beef |
| 4 |	Cheese |
| 5 |	Chicken |
| 6 |	Mushrooms |
| 7 |	Onions |
| 8 |	Pepperoni |
| 9 |	Peppers |
| 10 |	Salami |
| 11 |	Tomatoes |
| 12 |	Tomato Sauce |

```sql
CREATE VIEW pizza_ing_cleaned AS
 
	WITH RECURSIVE split_pizza_recipe AS (
		SELECT
			pizza_id,
			SUBSTRING_INDEX(toppings, ',', 1) AS ingredient_id, /* Fetching the first topping ID */
			SUBSTRING(toppings, LOCATE(',' , toppings) + 2) AS remaining_text /* Fetching the remaining topping IDs */
		FROM
			pizza_recipes
		
		UNION ALL /*union with the remainder text below */
		SELECT
			pizza_id,
			SUBSTRING_INDEX(remaining_text, ',', 1), /* Fetching the first topping ID of the remaining text */
			SUBSTRING(remaining_text, LOCATE(',' , remaining_text) + 2)
		FROM
			split_pizza_recipe
		WHERE
			remaining_text != '' /* recursive or looping this process until there is nothing left */
        )

		
	SELECT 
		S.pizza_id, 
        	CAST(ingredient_id AS SIGNED) AS ingredient_id, /* changing datatype to number; signed. INT is not available */
        	N.pizza_name, 
        	T.topping_name
	FROM 
		split_pizza_recipe S
		JOIN pizza_toppings T ON S.ingredient_id = T.topping_id
		JOIN pizza_names N ON N.pizza_id = S.pizza_id
	WHERE 
		ingredient_id != 0
	ORDER BY 
		pizza_id ASC, 
        	CAST(ingredient_id AS SIGNED) ;

SELECT * FROM pizza_ing_cleaned;
```

Output view: **pizza_ing_cleaned**
| pizza_id | ingredient_id | pizza_name | topping_name |
| --- | --- | --- | --- |
|1|	1 |	Meatlovers |	Bacon |
|1|	2 |	Meatlovers |	BBQ Sauce |
|1|	3 |	Meatlovers |	Beef |
|1|	4 |	Meatlovers |	Cheese |
|1|	5 |	Meatlovers |	Chicken |
|1|	6 |	Meatlovers |	Mushrooms |
|1|	8 |	Meatlovers |	Pepperoni |
|1|	10 |	Meatlovers |	Salami |
|2|	2 |	Vegetarian |	BBQ Sauce |
|2|	4 |	Vegetarian |	Cheese |
|2|	6 |	Vegetarian |	Mushrooms |
|2|	7 |	Vegetarian |	Onions |
|2|	9 |	Vegetarian |	Peppers |
|2|	11 |	Vegetarian |	Tomatoes |
|2|	12 |	Vegetarian |	Tomato Sauce |

<br>

### 1.2. Cleaning Runner Orders Table


**Converting this table : Runner_orders**
| order_id | runner_id | pickup_time         | distance | duration   | cancellation          |
| -------- | --------- | ------------------- | -------- | ---------- | --------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                       |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                       |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                       |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4km   | 40         |                       |
| 5        | 3         | 2020-01-08 21:10:57 | 10km     | 15         |                       |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25 mins    |                       |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minutes |                       |
| 9        | 2         | null                | null     | null       | Customer Cancellation  |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10 minutes |                       |

```sql

CREATE VIEW cleaned_runner_orders AS
SELECT 
	order_id,
	CAST(IF (pickup_time = 'null', NULL, pickup_time) AS DATETIME) AS pickup_time, /*cleaning the column, some nulls are in actual string form */
    	CAST(
		CASE 
			WHEN distance LIKE '%km' THEN TRIM(SUBSTRING_INDEX(distance, 'km', 1)) /*removing additional string of km */
			WHEN distance = 'null' THEN NULL
			ELSE distance
			END
		AS float)
		AS 'distance(km)', /* `` is required for string value with ( ) */
    	CAST(
		CASE
			WHEN duration = 'null' THEN NULL
			WHEN duration REGEXP '[0-9]+' THEN REGEXP_SUBSTR(duration, '[0-9]+') /* just experimenting substring with regex */
			ELSE TRIM(duration)
			END
		AS SIGNED)
		AS 'duration(mins)',
    	CASE
		WHEN cancellation = 'null' THEN NULL
        	WHEN cancellation = '' THEN NULL
        	ELSE cancellation
        	END
		AS cancellation,
    	O.runner_id,
    	R.registration_date,
    	O.rating
FROM runner_orders O
JOIN runners R ON O.runner_id = R.runner_id;

SELECT * FROM cleaned_runner_orders;
```

Output view: **cleaned_runner_orders**

| order_id | pickup_time         | distance(km) | duration(mins) | cancellation          | runner_id | registration_date |
| -------- | ------------------- | ------------ | -------------- | --------------------- | --------- | ----------------- |
| 1        | 2020-01-01 18:15:34 | 20           | 32             |                       | 1         | 2021-01-01        |
| 2        | 2020-01-01 19:10:54 | 20           | 27             |                       | 1         | 2021-01-01        |
| 3        | 2020-01-03 00:12:37 | 13.4         | 20             |                       | 1         | 2021-01-01        |
| 4        | 2020-01-04 13:53:03 | 23.4         | 40             |                       | 2         | 2021-01-03        |
| 5        | 2020-01-08 21:10:57 | 10           | 15             |                       | 3         | 2021-01-08        |
| 6        |                     |              |                | Restaurant Cancellation | 3         | 2021-01-08        |
| 7        | 2020-01-08 21:30:45 | 25           | 25             |                       | 2         | 2021-01-03        |
| 8        | 2020-01-10 00:15:02 | 23.4         | 15             |                       | 2         | 2021-01-03        |
| 9        |                     |              |                | Customer Cancellation  | 2         | 2021-01-03        |
| 10       | 2020-01-11 18:50:20 | 10           | 10             |                       | 1         | 2021-01-01        |


<br>


### 1.3. Cleaning Customer Orders Table

**Converting this table : customer_orders**

| order_id | customer_id | pizza_id | exclusions | extras | order_time          | 
| -------- | ----------- | -------- | ---------- | ------ | ------------------- | 
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 | 
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 | 
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 | 
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 | 
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 | 
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 | 
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 | 
| 5        | 104         | 1        | null           | 1      | 2020-01-08 21:00:29 | 
| 6        | 101         | 2        | null           | null       | 2020-01-08 21:03:13 | 
| 7        | 105         | 2        | null           | 1      | 2020-01-08 21:20:29 | 
| 8        | 102         | 1        | null           | null       | 2020-01-09 23:54:33 | 
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 | 
| 10       | 104         | 1        | null           | null        | 2020-01-11 18:34:49 | 
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 | 

```sql
ALTER TABLE customer_orders
ADD COLUMN record_id INT AUTO_INCREMENT PRIMARY KEY;

ALTER VIEW cleaned_customer_orders AS
SELECT 	record_id,
	order_id,
	customer_id,
	pizza_id,
	order_time,
	CASE 
		WHEN exclusions = 'null' THEN NULL
		WHEN exclusions = '' THEN NULL
        	ELSE exclusions
        	END
		AS exclusions,
	CASE 
		WHEN extras = 'null' THEN NULL
		WHEN extras = '' THEN NULL
        	ELSE extras
        	END
	AS extras
FROM customer_orders;
SELECT * FROM cleaned_customer_orders;
```

Output view: **cleaned_customer_orders**
| record_id | order_id | customer_id | pizza_id | order_time          | exclusions | extras |
| --------- | -------- | ----------- | -------- | ------------------- | ---------- | ------ |
| 1         | 1        | 101         | 1        | 2020-01-01 18:05:02 |            |        |
| 2         | 2        | 101         | 1        | 2020-01-01 19:00:52 |            |        |
| 3         | 3        | 102         | 1        | 2020-01-02 23:51:23 |            |        |
| 4         | 3        | 102         | 2        | 2020-01-02 23:51:23 |            |        |
| 5         | 4        | 103         | 1        | 2020-01-04 13:23:46 | 4          |        |
| 6         | 4        | 103         | 1        | 2020-01-04 13:23:46 | 4          |        |
| 7         | 4        | 103         | 2        | 2020-01-04 13:23:46 | 4          |        |
| 8         | 5        | 104         | 1        | 2020-01-08 21:00:29 |            | 1      |
| 9         | 6        | 101         | 2        | 2020-01-08 21:03:13 |            |        |
| 10        | 7        | 105         | 2        | 2020-01-08 21:20:29 |            | 1      |
| 11        | 8        | 102         | 1        | 2020-01-09 23:54:33 |            |        |
| 12        | 9        | 103         | 1        | 2020-01-10 11:22:59 | 4          | 1, 5   |
| 13        | 10       | 104         | 1        | 2020-01-11 18:34:49 |            |        |
| 14        | 10       | 104         | 1        | 2020-01-11 18:34:49 | 2, 6       | 1, 4   |

<br>


### 1.4. Creating exclusions Table

```sql
CREATE VIEW cleaned_exclusions AS

WITH recursive split_exclusions AS (
	SELECT 
		record_id,
 		order_id,
        	pizza_id,
		SUBSTRING_INDEX(exclusions, ',', 1) AS exclusions2, /* similar recursive form as previous codes */
        	SUBSTRING(exclusions, LOCATE(',', exclusions) +2) AS remainders
	FROM cleaned_customer_orders
    	UNION ALL
    	SELECT
		record_id,
        	order_id,
        	pizza_id,
        	SUBSTRING_INDEX(remainders, ',', 1),
        	SUBSTRING(remainders, LOCATE(',',remainders) + 2)
	FROM split_exclusions
    	WHERE remainders != ''
    )
    
SELECT 
    record_id, 
    order_id, 
    pizza_name, 
    T.topping_id AS excluded_topping_id, 
    T.topping_name AS excluded_topping
FROM split_exclusions S
JOIN pizza_toppings T ON S.exclusions2 = T.topping_id
JOIN pizza_names N ON N.pizza_id = S.pizza_id
WHERE exclusions2 IS NOT NULL
ORDER BY record_id ASC;
    
SELECT * FROM cleaned_exclusions;

```

| record_id | order_id | pizza_name | excluded_topping_id | excluded_topping |
| --------- | -------- | ---------- | ------------------- | ---------------- |
| 5         | 4        | Meatlovers | 4                   | Cheese           |
| 6         | 4        | Meatlovers | 4                   | Cheese           |
| 7         | 4        | Vegetarian | 4                   | Cheese           |
| 12        | 9        | Meatlovers | 4                   | Cheese           |
| 14        | 10       | Meatlovers | 2                   | BBQ Sauce        |
| 14        | 10       | Meatlovers | 6                   | Mushrooms        |

<br>


### 1.5. Creating extras Table

```sql
CREATE VIEW cleaned_extras AS

WITH recursive split_extras AS (
	SELECT 
		record_id,
        	order_id,
        	pizza_id,
		SUBSTRING_INDEX(extras, ',', 1) AS extras2,
        	SUBSTRING(extras, LOCATE(',', extras) +2) AS remainders
	FROM cleaned_customer_orders
    	UNION ALL
    	SELECT
		record_id,
        	order_id,
        	pizza_id,
        	SUBSTRING_INDEX(remainders, ',', 1),
        	SUBSTRING(remainders, LOCATE(',',remainders) + 2)
	FROM split_extras
    	WHERE remainders != ''
)
    
SELECT
	record_id,
	order_id,
	pizza_name,
	T.topping_id AS extra_topping_id,
	T.topping_name AS extra_topping
FROM split_extras S
JOIN pizza_toppings T ON S.extras2 = T.topping_id
JOIN pizza_names N ON N.pizza_id = S.pizza_id
WHERE extras2 IS NOT NULL
ORDER BY record_id;
    
SELECT * FROM cleaned_extras;

```

| record_id | order_id | pizza_name | extra_topping_id | extra_topping |
| --------- | -------- | ---------- | ---------------- | ------------- |
| 8         | 5        | Meatlovers | 1                | Bacon         |
| 10        | 7        | Vegetarian | 1                | Bacon         |
| 12        | 9        | Meatlovers | 1                | Bacon         |
| 12        | 9        | Meatlovers | 5                | Chicken       |
| 14        | 10       | Meatlovers | 1                | Bacon         |
| 14        | 10       | Meatlovers | 4                | Cheese        |

<br>


---

## Part 2. Pizza Metrics

### Question 1: How many pizzas were ordered?

```sql
SELECT COUNT(order_id) as quantity
FROM cleaned_customer_orders;
```

**14 Pizzas were ordered.**

<br>


### Question 2: How many unique customer orders were made?

```sql
SELECT  customer_id, 
        COUNT(DISTINCT order_id) AS no_orders
FROM cleaned_customer_orders
GROUP BY customer_id;
```

| customer_id | no_orders |
| ----------- | --------- |
| 101         | 3         |
| 102         | 2         |
| 103         | 2         |
| 104         | 2         |
| 105         | 1         |

<br>


### Question 3: How many successful orders were delivered by each runner? 

```sql
SELECT COUNT(order_id) AS no_successful_order
FROM cleaned_runner_orders
WHERE cancellation IS NULL;
```

**8 successful orders**

<br>


### Question 4: How many of each type of pizza was delivered? 

```sql
WITH successful_order AS (
	SELECT 	COUNT(order_id) AS no_successful_order, /* fetching up the orders with no cancellation from runners table */
		order_id
	FROM 	cleaned_runner_orders
	WHERE 	cancellation IS NULL
	GROUP BY order_id
)
SELECT  N.pizza_name, 
        COUNT(O.pizza_id) AS no_delivered_pizza /* counting the number of delivered pizzas from successful delivery order */
FROM cleaned_customer_orders O
JOIN successful_order S ON O.order_id = S.order_id
JOIN pizza_names N ON N.pizza_id = O.pizza_id
GROUP BY N.pizza_name;
```

| pizza_name | no_delivered_pizza |
| ---------- | ------------------ |
| Meatlovers | 9                  |
| Vegetarian | 3                  |

<br>


### Question 5: How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT  O.customer_id, 
        N.pizza_name, 
        COUNT(order_id) AS no_order
FROM cleaned_customer_orders O
JOIN pizza_names N ON N.pizza_id = O.pizza_id
GROUP BY O.customer_id, N.pizza_name
ORDER BY customer_id;
```

| customer_id | pizza_name  | no_order |
| ----------- | ----------- | -------- |
| 101         | Meatlovers  | 2        |
| 101         | Vegetarian  | 1        |
| 102         | Meatlovers  | 2        |
| 102         | Vegetarian  | 1        |
| 103         | Meatlovers  | 3        |
| 103         | Vegetarian  | 1        |
| 104         | Meatlovers  | 3        |
| 105         | Vegetarian  | 1        |

<br>


### Question 6: What was the maximum number of pizzas delivered in a single order?

```sql
SELECT  COUNT(order_id) AS no_pizza_ordered, 
        order_id
FROM cleaned_customer_orders
GROUP BY order_id
ORDER BY no_pizza_ordered DESC
LIMIT 1;
```

| no_pizza_ordered | order_id |
| ---------------- | -------- |
| 3                | 4        |

<br>


### Question 7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
WITH temp_table AS
	(WITH successful_order AS 
	    (   SELECT order_id /*fetching list of successful orders */
	        FROM cleaned_runner_orders
	        WHERE cancellation IS NULL
	    )
	SELECT /*creating a new column to tag exclusions, extras, or no change */
		customer_id, 
		CASE 
			WHEN exclusions IS NOT NULL THEN 'exclusions'
			WHEN extras IS NOT NULL THEN 'extras'
			ELSE 'No change'
		END
		AS changes_made
	FROM cleaned_customer_orders O
	JOIN successful_order S ON O.order_id = S.order_id
	)


SELECT
	customer_id,
    	COUNT(
		CASE 	WHEN changes_made = 'exclusions' THEN 1 /* tags the changes to 1 then sum them up to count the number of changes */
			WHEN changes_made = 'extras' THEN 1
		END) As Num_changes_made,
	COUNT(
		CASE WHEN changes_made = 'No change' THEN 1
		END) As Num_no_changes
FROM temp_table
GROUP BY customer_id;
```

| customer_id | Num_changes_made | Num_no_changes |
| ----------- | ---------------- | -------------- |
| 101         | 0                | 2              |
| 102         | 0                | 3              |
| 103         | 3                | 0              |
| 104         | 2                | 1              |
| 105         | 1                | 0              |

<br>


### Question 8: How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT COUNT(C.order_id)
FROM cleaned_customer_orders C
JOIN cleaned_runner_orders R ON C.order_id = R.order_id
WHERE
	C.exclusions IS NOT NULL 
    	AND C.extras IS NOT NULL 
    	AND R.cancellation IS NULL;
```
**1 Pizza**

<br>


### Question 9: What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
    CAST(
        REGEXP_SUBSTR(order_time, '[0-9]{4}-[0-9]{2}-[0-9]{2} ') /* fetching dates string with regex */
        AS DATE) 
    	AS date_order,
    CAST(
        TRIM(REGEXP_SUBSTR(order_time, ' [0-9]{2}')) /* fetching the hours only with regex (I'm actually a regex fan) */
        AS UNSIGNED) 
    	AS hour_order,
    COUNT(order_id) AS no_pizza_ordered
FROM cleaned_customer_orders
GROUP BY date_order, hour_order;
```
| date_order | hour_order | no_pizza_ordered |
| ---------- | ---------- | ---------------- |
| 2020-01-01 | 18         | 1                |
| 2020-01-01 | 19         | 1                |
| 2020-01-02 | 23         | 2                |
| 2020-01-04 | 13         | 3                |
| 2020-01-08 | 21         | 3                |
| 2020-01-09 | 23         | 1                |
| 2020-01-10 | 11         | 1                |
| 2020-01-11 | 18         | 2                |
| 2024-04-28 | 7          | 4                |

<br>


### Question 10: What was the volume of orders for each day of the week? 

```sql
SELECT
	DAYNAME(STR_TO_DATE(order_time, '%Y-%m-%d %H:%i:%s')) AS day_of_week, 
    	COUNT(order_id) AS no_orders
FROM cleaned_customer_orders
GROUP BY day_of_week;
```

| day_of_week | no_orders |
| ----------- | --------- |
| Wednesday   | 5         |
| Thursday    | 3         |
| Saturday    | 5         |
| Friday      | 1         |
| Sunday      | 4         |

---

<br>


## Part 3.  Runner and Customer Experience

### Question 1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
WITH RECURSIVE WeekNumbers AS (
    SELECT
	1 AS WeekNumber,
    	CAST('2021-01-01' AS DATE) AS StartDate	/*creating loop of weeknumbers based on the start date + 7 */
    
    UNION ALL
    SELECT 
        WeekNumber + 1,
        DATE_ADD(StartDate, INTERVAL 7 DAY)
    FROM 
        WeekNumbers
    WHERE 
        DATE_ADD(StartDate, INTERVAL 7 DAY) <= '2021-02-01' /* Limit recursion */
)

SELECT 
    COUNT(r.runner_id) AS no_runners_joined,
    wn.StartDate AS week_starting,
    wn.WeekNumber
FROM 
    runners r
JOIN 
    WeekNumbers wn ON 
    r.registration_date >= wn.StartDate AND 
    r.registration_date < DATE_ADD(wn.StartDate, INTERVAL 7 DAY)
GROUP BY 
    wn.StartDate, wn.WeekNumber;
```
| no_runners_joined | week_starting | WeekNumber |
| ----------------- | ------------- | ---------- |
| 2                 | 2021-01-01    | 1          |
| 1                 | 2021-01-08    | 2          |
| 1                 | 2021-01-15    | 3          |

<br>


### Question 2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql

WITH tempoTable AS ( 
    SELECT /* joining the runner order and customer order tables to get pickup and order time */
        RO.order_id,
        RO.runner_id,
        CAST(RO.pickup_time AS DATETIME) AS pickupTime,
        CAST(CO.order_time AS DATETIME) AS orderTime
    FROM 
        cleaned_runner_orders RO
	JOIN cleaned_customer_orders CO ON CO.order_id = RO.order_id
    WHERE RO.pickup_time IS NOT NULL
)

SELECT
	AVG(TIME_TO_SEC
            (TIMEDIFF(pickupTime, orderTime) ) /* finding the difference and then average them for each runner */
	    )/ 60 AS avg_mins_to_arrive,
    	runner_id
FROM tempoTable
GROUP BY runner_id;
```

| avg_mins_to_arrive | runner_id |
| ------------------- | --------- |
| 15.67777778        | 1         |
| 23.72000000        | 2         |
| 10.46666667        | 3         |

<br>


### Question 3: Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
CREATE VIEW mins_prep_per_orderID AS
		SELECT 
			RO.order_id,
			TIME_TO_SEC (
                        	TIMEDIFF (CAST(RO.pickup_time AS DATETIME),  CAST(CO.order_time AS DATETIME) ) 
                        	) /60 AS mins_prep_time
		FROM 
			cleaned_runner_orders RO
			JOIN cleaned_customer_orders CO ON CO.order_id = RO.order_id
		WHERE RO.pickup_time IS NOT NULL;

CREATE VIEW No_pizza_per_orderID AS
    SELECT
        order_id,
        COUNT(order_id) AS no_pizza_ordered
    FROM mins_prep_per_orderID
    GROUP BY order_id;
    
SELECT 
	AVG(mins_prep_time),
	no_pizza_ordered
FROM mins_prep_per_orderid M
JOIN no_pizza_per_orderid P ON P.order_id = M.order_id
GROUP BY no_pizza_ordered;
```
**Yes, it seems that each pizza requires around 10 mins to prepare, and 1 pizza only requires longer (12mins)**

| AVG(mins_prep_time) | no_pizza_ordered |
| -------------------- | ---------------- |
| 12.35666000         | 1                |
| 18.37500000         | 2                |
| 29.28330000         | 3                |

<br>


### Question 4: What was the average distance travelled for each customer?

```sql
SELECT  DISTINCT 
	ROUND(AVG(`distance(km)`)) AS Avg_distance_in_km, 
	customer_id
FROM cleaned_customer_orders CO
JOIN cleaned_runner_orders RO ON RO.order_id = CO.order_id
WHERE `distance(km)` IS NOT NULL
GROUP BY customer_id;
```

| Avg_distance_in_km | customer_id |
| ------------------ | ----------- |
| 20                 | 101         |
| 17                 | 102         |
| 23                 | 103         |
| 10                 | 104         |
| 25                 | 105         |

<br>


### Question 5: What was the difference between the longest and shortest delivery times for all orders?

```sql

SELECT
	MAX(`duration(mins)`) - MIN(`duration(mins)`) AS answer
FROM cleaned_runner_orders;
```

**30 minutes difference**

<br>


### Question 6: What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT
	runner_id,
	order_id,
	ROUND( `distance(km)`/ (`duration(mins)`/60) ) AS speed	/* assuming speed in this question is km per hour */
FROM cleaned_runner_orders
WHERE `duration(mins)` iS NOT NULL
ORDER BY runner_id, order_id;
```
**it seems that the more order they delivered, the faster they become**

| runner_id | order_id | speed |
| --------- | -------- | ----- |
| 1         | 1        | 38    |
| 1         | 2        | 44    |
| 1         | 3        | 40    |
| 1         | 10       | 60    |
| 2         | 4        | 35    |
| 2         | 7        | 60    |
| 2         | 8        | 94    |
| 3         | 5        | 40    |

<br>


### Question 7: What is the successful delivery percentage for each runner?

```sql
SELECT 
	runner_id, 
    	COUNT(order_id) AS No_orders,
    	SUM(CASE WHEN cancellation IS NULL THEN 1 /*quantifying successful order */
		 ELSE 0
		 END)
		 AS successful_order,
	ROUND(
        	(SUM(CASE   WHEN cancellation IS NULL THEN 1 /*finding the percentage */
		            ELSE 0
			    END)
        	/
		COUNT(order_id)
        	)*100 
    		, 2)
		AS success_rate
FROM cleaned_runner_orders
GROUP BY runner_id;
```
| runner_id | No_orders | successful_order | success_rate |
| --------- | --------- | ---------------- | ------------ |
| 1         | 4         | 4                | 100.00       |
| 2         | 4         | 3                | 75.00        |
| 3         | 2         | 1                | 50.00        |

<br>


## Part 4. Ingredient Optimisation

### Question 1: What are the standard ingredients for each pizza? 

```sql
SELECT topping_name AS std_ingredient
FROM pizza_ing_cleaned
GROUP BY topping_name
HAVING count(pizza_id) = 2;
```
| std_ingredient |
| ------------ |
| BBQ Sauce    |
| Cheese       |
| Mushrooms    |

<br>


### Question 2: What was the most commonly added extra?

```sql
WITH recursive split_extras AS (
	SELECT 
		order_id,
		SUBSTRING_INDEX(extras, ',', 1) AS extras, /*creating list of order_id: extra topping with recursive function like before */
        	SUBSTRING(extras, LOCATE(',', extras) +2) AS remainders2
	FROM cleaned_customer_orders
    	WHERE extras IS NOT NULL
    
    	UNION ALL
    
    	SELECT
		order_id,
        	SUBSTRING_INDEX(remainders2, ',', 1),
        	SUBSTRING(remainders2, LOCATE(remainders2, ',') +2)
	FROM split_extras
 	WHERE remainders2 != ''
)
    
SELECT
	COUNT(S.order_id),
	T.topping_name
FROM split_extras S
JOIN pizza_toppings T ON S.extras = T.topping_id
GROUP BY T.topping_name
ORDER BY COUNT(S.order_id) DESC
LIMIT 1;
```
**Bacon is the most commonly added extra**

| COUNT(S.order_id) | topping_name |
| ----------------- | ------------ |
| 5                 | Bacon        |

<br>


### Question 3: What was the most common exclusion?

```sql
WITH RECURSIVE split_exclusions AS (
    SELECT 
        order_id,
        SUBSTRING_INDEX(exclusions, ',', 1) AS exclusions,
        CASE
            WHEN LOCATE(',', exclusions) > 0 THEN SUBSTRING(exclusions, LOCATE(',', exclusions) + 2)
            ELSE ''
	    END
	    AS remainders
    FROM cleaned_customer_orders
    WHERE exclusions IS NOT NULL
    UNION ALL
    SELECT
        order_id,
        SUBSTRING_INDEX(remainders, ',', 1),
        CASE
            WHEN LOCATE(',', remainders) > 0 THEN SUBSTRING(remainders, LOCATE(',', remainders) + 2)
            ELSE ''
            END
	    AS remainders
    FROM split_exclusions
    WHERE remainders != ''
)

SELECT
	T.topping_name,
	COUNT(S.order_id)
FROM split_exclusions S
JOIN pizza_toppings T ON S.exclusions = T.topping_id
GROUP BY T.topping_name
ORDER BY COUNT(S.order_id) DESC
LIMIT 1;
```
**Cheese is the most commonly excluded**
| topping_name | COUNT(S.order_id) |
| ------------ | ----------------- |
| Cheese       | 5                 |

<br>


### Question 4: Generate an order item for each record in the customers_orders table in the format of one of the following:
### Meat Lovers - Exclude Beef
### Meat Lovers - Extra Bacon
### Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```sql
INSERT INTO customer_orders(order_id, customer_id, pizza_id, exclusions, extras, order_time)
VALUES 
    (11, NULL, 1, 3, NULL, CURRENT_TIME()),
    (12, NULL, 1, NULL, 1, CURRENT_TIME()),
    (13, NULL, 1, '1, 4', '6, 9', CURRENT_TIME());
```

<br>


### Question 5: Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```sql
WITH tempTable4 AS (
		WITH tempTable3 AS (
				        SELECT DISTINCT /* there is no outer join in mysql as far as i know, so i have to combine left join and right join */
						CO.order_id,
						CO.record_id,
						PI.pizza_name,
						PI.topping_name,
						CE.extra_topping
					FROM cleaned_customer_orders CO
					JOIN pizza_ing_cleaned PI ON CO.pizza_id = PI.pizza_id
					LEFT JOIN cleaned_extras CE ON CE.record_id = CO.record_id AND topping_name = extra_topping

					/*the section above is fetching the extras that are already in the ingredient list, */
					/* below is for extras that are not in the original ingredient list */

					UNION ALL

				        SELECT DISTINCT
						CE.order_id,
						CE.record_id,
						CE.pizza_name,
						PI.topping_name,
						CE.extra_topping
					FROM cleaned_customer_orders CO
					JOIN pizza_ing_cleaned PI ON CO.pizza_id = PI.pizza_id
					RIGHT JOIN cleaned_extras CE ON CE.record_id = CO.record_id AND topping_name = extra_topping
					WHERE topping_name IS NULL
        				)

		SELECT T.order_id, T.record_id, T.pizza_name,
				CASE	WHEN topping_name = extra_topping THEN CONCAT('2x ', topping_name) /* updating the ingredient list */
					WHEN topping_name = X.excluded_topping THEN null
					WHEN topping_name IS NULL THEN extra_topping
					ELSE topping_name
					END
					AS updated_ingredient
		FROM tempTable3 T
		LEFT JOIN cleaned_exclusions X ON T.record_id = X.record_id AND T.topping_name = X.excluded_topping
)

SELECT  record_id, 
        order_id,
	CONCAT(pizza_name, ': ', GROUP_CONCAT(updated_ingredient ORDER BY updated_ingredient SEPARATOR ', ' )) AS pizza_ingredient
	/* finally concat all the strings together as required */
FROM tempTable4
GROUP BY order_id, record_id, pizza_name;
```
| record_id | order_id | pizza_ingredient                                                                           |
| --------- | -------- | ------------------------------------------------------------------------------------------- |
| 1         | 1        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 2         | 2        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 3         | 3        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 4         | 3        | Vegetarian: BBQ Sauce, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes          |
| 5         | 4        | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami                  |
| 6         | 4        | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami                  |
| 7         | 4        | Vegetarian: BBQ Sauce, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                   |
| 8         | 5        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami      |
| 9         | 6        | Vegetarian: BBQ Sauce, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes          |
| 10        | 7        | Vegetarian: Bacon, BBQ Sauce, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes  |
| 11        | 8        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 12        | 9        | Meatlovers: 2x Bacon, 2x Chicken, BBQ Sauce, Beef, Mushrooms, Pepperoni, Salami            |
| 13        | 10       | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 14        | 10       | Meatlovers: 2x Bacon, 2x Cheese, Beef, Chicken, Pepperoni, Salami                           |
| 15        | 11       | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami         |
| 16        | 11       | Meatlovers: Bacon, BBQ Sauce, Cheese, Chicken, Mushrooms, Pepperoni, Salami                |
| 17        | 12       | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami       |
| 18        | 13       | Meatlovers: 2x Mushrooms, BBQ Sauce, Beef, Chicken, Pepperoni, Peppers, Salami             |

<br>


### Question 6: What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```sql
WITH tempTable6 AS(
                SELECT /* removing the excluded ingredient from the list */
			CO.record_id,
			CO.order_id,
			PI.pizza_name,
			CASE	WHEN topping_name = excluded_topping THEN null
				ELSE topping_name
                                END
				AS updated_ingredient
		FROM cleaned_customer_orders CO
		JOIN pizza_ing_cleaned PI ON CO.pizza_id = PI.pizza_id
		LEFT JOIN cleaned_exclusions X ON CO.record_id = X.record_id AND PI.topping_name = X.excluded_topping

		UNION ALL
		SELECT /* including the extras on the list of ingredients */
			record_id,
			order_id,
			pizza_name,
			extra_topping AS updated_ingredient
                FROM cleaned_extras
)
SELECT  COUNT(updated_ingredient) AS quantity, /* counting the updated ingredient list */
        updated_ingredient AS ingredient
FROM tempTable6
WHERE updated_ingredient IS NOT NULL
GROUP BY updated_ingredient
ORDER BY COUNT(updated_ingredient) DESC,
	 updated_ingredient ASC
;
```
| quantity | ingredient   |
| -------- | ------------ |
| 18       | Bacon        |
| 18       | Mushrooms    |
| 17       | BBQ Sauce    |
| 15       | Chicken      |
| 14       | Cheese       |
| 14       | Pepperoni    |
| 14       | Salami       |
| 13       | Beef         |
| 5        | Peppers      |
| 4        | Onions       |
| 4        | Tomato Sauce |
| 4        | Tomatoes     |

<br>


## Part 5. Pricing and Ratings

### Question 1: If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```sql
WITH tempTable7 AS(
SELECT  pizza_id, 
        COUNT(record_id) AS quantity, /* creating a price column */
		CASE WHEN pizza_id = 1 THEN 12
		     ELSE 10
                     END
		     AS price
FROM cleaned_customer_orders
GROUP BY pizza_id
)
SELECT SUM(quantity*price) AS Revenue FROM tempTable7;
```
**$208 Revenue**

<br>


### Question 2: What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra

```sql
WITH tempTable7 AS( /*creating CTE of cleaned_customer_orders with price and quantity sold */
	SELECT 	pizza_id,
		COUNT(record_id) AS quantity,
		CASE 	WHEN pizza_id = 1 THEN 12
			ELSE 10
			END
			AS price
	FROM cleaned_customer_orders
	GROUP BY pizza_id
)
SELECT  SUM(quantity*price) + /* finding the revenue */
		    (SELECT 
		    	SUM(CASE WHEN extra_topping = 'Cheese' THEN 2 /*finding the revenue from extras */
			     ELSE 1
			     END) 
			AS extra_cost
		     FROM cleaned_extras)
	AS Revenue
FROM tempTable7;
```
**Revenue of $218**

<br>


### Question 3: The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, 
### how would you design an additional table for this new dataset? 
### insert your own data for ratings for each successful customer order between 1 to 5.

```sql
ALTER TABLE runner_orders
ADD rating ENUM('1', '2', '3', '4', '5');

UPDATE runner_orders
SET rating = 
	CASE	WHEN order_id = 1 THEN 1
		WHEN order_id = 2 THEN 2
            	WHEN order_id = 3 THEN 3
            	WHEN order_id = 4 THEN 4
            	WHEN order_id = 5 THEN 5
            	WHEN order_id = 6 THEN 1
            	WHEN order_id = 7 THEN 2
            	WHEN order_id = 8 THEN 3
            	WHEN order_id = 9 THEN 4
            	ELSE 5
            	END;
```

<br>


### Question 4: Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
### customer_id, order_id, runner_id, rating, order_time, pickup_time, Time between order and pickup, Delivery duration, Average speed, Total number of pizzas

```sql
WITH tempTable8 AS(
        SELECT 	COUNT(record_id) AS pizza_quantity, /* creating CTE with order_time and pizza quantity */
		order_id,
                customer_id,
                order_time
        FROM cleaned_customer_orders
        WHERE customer_id IS NOT NULL
        GROUP BY order_id, customer_id, order_time
)

SELECT  customer_id, 
        R.order_id, 
        runner_id, 
        rating, 
        order_time, 
        pickup_time, 
	TIMEDIFF(pickup_time, order_time) AS Time_between_order_and_pickup,
	SEC_TO_TIME(`duration(mins)` * 60) AS delivery_duration,
        ROUND(AVG(`distance(km)`/ (`duration(mins)` / 60)) 
		OVER (PARTITION BY runner_id) , 2) AS runner_avg_speed, /* window function, averaging the speed for each runner */
	pizza_quantity  AS total_number_of_pizza
FROM cleaned_runner_orders R
JOIN tempTable8 CO ON CO.order_id = R.order_id
WHERE cancellation IS NULL;
```
| customer_id | order_id | runner_id | rating | order_time         | pickup_time        | Time_between_order_and_pickup | delivery_duration | runner_avg_speed | total_number_of_pizza |
|-------------|----------|-----------|--------|--------------------|--------------------|--------------------------------|-------------------|------------------|-----------------------|
| 101         | 1        | 1         | 1      | 2020-01-01 18:05:02| 2020-01-01 18:15:34| 00:10:32                      | 00:32:00          | 45.54            | 1                     |
| 101         | 2        | 1         | 2      | 2020-01-01 19:00:52| 2020-01-01 19:10:54| 00:10:02                      | 00:27:00          | 45.54            | 1                     |
| 102         | 3        | 1         | 3      | 2020-01-02 23:51:23| 2020-01-03 00:12:37| 00:21:14                      | 00:20:00          | 45.54            | 2                     |
| 104         | 10       | 1         | 5      | 2020-01-11 18:34:49| 2020-01-11 18:50:20| 00:15:31                      | 00:10:00          | 45.54            | 2                     |
| 103         | 4        | 2         | 4      | 2020-01-04 13:23:46| 2020-01-04 13:53:03| 00:29:17                      | 00:40:00          | 62.9             | 3                     |
| 105         | 7        | 2         | 2      | 2020-01-08 21:20:29| 2020-01-08 21:30:45| 00:10:16                      | 00:25:00          | 62.9             | 1                     |
| 102         | 8        | 2         | 3      | 2020-01-09 23:54:33| 2020-01-10 00:15:02| 00:20:29                      | 00:15:00          | 62.9             | 1                     |
| 104         | 5        | 3         | 5      | 2020-01-08 21:00:29| 2020-01-08 21:10:57| 00:10:28                      | 00:15:00          | 40               | 1                     |

<br>


### Question 5: If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
WITH tempTable10 AS(
                    SELECT order_id, /*creating price column */
		            SUM(CASE WHEN pizza_id = 1 THEN 12
				     ELSE 10
				     END) AS Revenue_per_order
                    FROM cleaned_customer_orders
                    GROUP BY order_id
)
SELECT 	CO.order_id, 
	ROUND (`distance(km)` * 0.3 , 2) AS delivery_cost,
	Revenue_per_order,
	ROUND (Revenue_per_order - (`distance(km)` * 0.3) ,2) AS Profit_or_loss
FROM cleaned_runner_orders CO
JOIN tempTable10 T ON T.order_id = CO.order_id
WHERE `distance(km)` IS NOT NULL;
```
**Total Profit is $94.44** 
| order_id | delivery_cost | Revenue_per_order | Profit_or_loss |
|----------|---------------|-------------------|----------------|
| 1        | 6             | 12                | 6              |
| 2        | 6             | 12                | 6              |
| 3        | 4.02          | 22                | 17.98          |
| 4        | 7.02          | 34                | 26.98          |
| 5        | 3             | 12                | 9              |
| 7        | 7.5           | 10                | 2.5            |
| 8        | 7.02          | 12                | 4.98           |
| 10       | 3             | 24                | 21             |

