# Case Study - Danny's Diner

Dear readers,

Thank you for visiting my GitHub page.  
In this document, I showcase a shorter case study from 'data with Danny' where I leverage MySQL 5.7 Workbench to handle business queries, demonstrating my skills in 


Because of the outdated version, some functions such as WITH is not available, so I used CREATE VIEW instead.


The further down the questions, the more complicated the queries are.

To view the complete case study, please visit [this page](https://8weeksqlchallenge.com/case-study-1/).

---
## Part 1. Tables Creation

```sql
CREATE SCHEMA dannys_diner;

CREATE TABLE sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INTEGER
);

INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

CREATE TABLE menu (
  product_id INTEGER,
  product_name VARCHAR(5),
  price INTEGER
);

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE dannys_diner.members (
  customer_id VARCHAR(1),
  join_date DATE
);

INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
  ```

  
<br>


---


## Part 2. Challenges

### Question 1: What is the total amount each customer spent at the restaurant?

```sql
SELECT 
    S.customer_id, 
    SUM(M.price) AS total_spent
FROM sales S
LEFT JOIN Menu M ON S.product_id = M.product_id
GROUP BY customer_id;
```

<br>

| customer_id | total_spent |
|-------------|-------------|
| A           | 76          |
| B           | 74          |
| C           | 36          |

<br>

### Question 2: How many days has each customer visited the restaurant?

```sql
  SELECT    
    customer_id, 
    COUNT(DISTINCT order_date) AS days
  FROM Sales
  GROUP BY customer_id;

```

<br>

| customer_id | days |
|-------------|------|
| A           | 4    |
| B           | 6    |
| C           | 2    |

<br>

---

### Question 3: What was the first item from the menu purchased by each customer?

```sql
CREATE VIEW combined AS 
SELECT 
    s.customer_id, 
    s.order_date, 
    m.product_name
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id;

CREATE VIEW min_val AS
SELECT 
    customer_id, 
    MIN(order_date) AS min_OD /*finding the first date of purchase */
FROM sales
GROUP BY customer_id;

SELECT DISTINCT 
    c.customer_id, 
    c.order_date, 
    c.product_name 
FROM combined c 
JOIN min_val mv ON c.customer_id = mv.customer_id 
                AND c.order_date = mv.min_OD;

```

<br>

| customer_id | order_date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |

<br>

---

### Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
CREATE VIEW Number_of_purchase AS
SELECT 
    product_name, 
    COUNT(product_name) as No_purchase
FROM combined
GROUP BY product_name; /* counting number of purchase */

SELECT 
    product_name, 
    No_purchase
FROM Number_of_purchase
WHERE No_purchase = (SELECT MAX(No_purchase) FROM Number_of_purchase); /*finding the biggest number of purchase */

```

<br>

| product_name | No_purchase |
|--------------|-------------|
| ramen        | 8           |

<br>

---

### Question 5: Which item was the most popular for each customer?

```sql
CREATE VIEW Tcount_purchase AS
SELECT 
    customer_id, 
    product_name, 
    COUNT(product_name) AS count_purchase
FROM combined
GROUP BY 
    customer_id, 
    product_name;

CREATE VIEW TMax_purchase AS /* finding max purchase */
SELECT 
    customer_id, 
    MAX(count_purchase) AS Max_purchase
FROM Tcount_purchase
GROUP BY customer_id;

SELECT 
    C.customer_id, 
    C.product_name, 
    M.Max_purchase
FROM Tcount_purchase C
JOIN TMax_purchase M ON C.customer_id = M.customer_id 
                     AND C.count_purchase = M.Max_purchase;

```

<br>

| customer_id | product_name | Max_purchase |
|-------------|--------------|--------------|
| A           | ramen        | 3            |
| B           | curry        | 2            |
| B           | sushi        | 2            |
| B           | ramen        | 2            |
| C           | ramen        | 3            |

<br>

---

### Question 6: Which item was purchased first by the customer after they became a member?

```sql
CREATE VIEW temp AS /*finding orders purchased after join date */
SELECT DISTINCT 
    c.customer_id, 
    c.product_name, 
    c.order_date, 
    m.join_date
FROM combined c 
JOIN members m ON c.customer_id = m.customer_id
WHERE order_date >= join_date
ORDER BY 
    order_date, 
    customer_id ASC;

CREATE VIEW temp2 AS /*finding the first purchase date after becoming a member */
SELECT 
    customer_id, 
    MIN(order_date) AS first_OD_member
FROM temp
GROUP BY customer_id;

SELECT 
    t.customer_id, 
    t.product_name, 
    t2.first_OD_member, 
    t.join_date
FROM temp t 
JOIN temp2 t2 ON t.customer_id = t2.customer_id 
              AND t.order_date = t2.first_OD_member;

```

<br>

| customer_id | product_name | first_OD_member | join_date  |
|-------------|--------------|-----------------|------------|
| A           | curry        | 2021-01-07      | 2021-01-07 |
| B           | sushi        | 2021-01-11      | 2021-01-09 |

<br>

---

### Question 7: Which item was purchased just before the customer became a member?

```sql
CREATE VIEW before_member AS /*finding orders purchased before join date */
SELECT DISTINCT 
    c.customer_id, 
    c.product_name, 
    c.order_date, 
    m.join_date
FROM combined c 
JOIN members m ON c.customer_id = m.customer_id
WHERE order_date < join_date
ORDER BY order_date, customer_id ASC;

CREATE VIEW last_order AS /*finding last order purchased before join date */
SELECT 
    customer_id, 
    MAX(order_date) AS last_OD_b4_member
FROM before_member
GROUP BY customer_id;

SELECT 
    t.customer_id, 
    t.product_name, 
    t2.last_OD_b4_member, 
    t.join_date
FROM before_member t 
JOIN last_order t2 ON t.customer_id = t2.customer_id 
                   AND t.order_date = t2.last_OD_b4_member;

```

<br>

| customer_id | product_name | last_OD_b4_member | join_date  |
|-------------|--------------|-------------------|------------|
| A           | sushi        | 2021-01-01        | 2021-01-07 |
| A           | curry        | 2021-01-01        | 2021-01-07 |
| B           | sushi        | 2021-01-04        | 2021-01-09 |

<br>

---

### Question 8: What is the total items and amount spent for each member before they became a member?

```sql

SELECT 
    B.customer_id, 
    COUNT(M.product_name) AS quantity, 
    SUM(M.price) AS spending
FROM before_member B 
JOIN menu M ON B.product_name = M.product_name
GROUP BY customer_id;

```

<br>

| customer_id | quantity | spending |
|-------------|----------|----------|
| A           | 2        | 25       |
| B           | 3        | 40       |

<br>

---

### Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
CREATE VIEW customer_purchase AS
SELECT 
    s.customer_id, 
    m.product_name, 
    m.price
FROM sales s
JOIN menu m ON s.product_id = m.product_id;

SELECT 
    customer_id, 
    SUM(IF(product_name = 'sushi', price*20, price*10)) AS points
FROM customer_purchase
GROUP BY customer_id;

```

<br>

| customer_id | points |
|-------------|--------|
| A           | 860    |
| B           | 940    |
| C           | 360    |

<br>

---
### Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
CREATE VIEW All_combined AS
SELECT 
    S.customer_id, 
    order_date, 
    join_date, 
    product_name, 
    price,
    IF (order_date - join_date <= 7 AND order_date - join_date >= 0, 'Bonus', 'Normal') AS Bonus_week /* tagging whether it is a normal or bonus week */
FROM sales S 
JOIN members E on S.customer_id = E.customer_id 
JOIN menu M on S.product_id = M.product_id;


SELECT 
    customer_id,
    SUM(
        CASE WHEN (product_name = 'sushi' AND Bonus_week = 'Normal') THEN price* 20 /* calculating points incorporating bonus week extra points */
             WHEN (product_name <> 'sushi' AND Bonus_week = 'Normal') THEN price* 10
             ELSE price* 20
             END
        ) AS Total_points
FROM All_combined
GROUP BY customer_id;
```

<br>

| customer_id | Total_points |
|-------------|--------------|
| B           | 1060         |
| A           | 1370         |

<br>

