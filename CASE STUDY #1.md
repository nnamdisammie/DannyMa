# DannyMa SQL Challenge
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500" height="500">

## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:
## Schema

**Schema (PostgreSQL v13)**
```sql
    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
  
  
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
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
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
 
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
  
  
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
   
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');
 ```

## Problem 1

    What is the total amount each customer spent at the restaurant?
### Solution
```sql
    SELECT sales.customer_id, SUM(menu.price) AS total_amount
    FROM dannys_diner.menu menu
    JOIN dannys_diner.sales sales
    ON menu.product_id = sales.product_id
    GROUP BY 1
    ORDER BY 1;
```
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |


## Problem 2
    How many days has each customer visited the restaurant?
### Solution
```sql    
    SELECT customer_id, COUNT( DISTINCT order_date)
    FROM dannys_diner.sales
    GROUP BY 1
    ORDER BY 1;
```
| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |


## Problem 3
    What was the first item from the menu purchased by each customer?
### Solution
```sql
    SELECT sales.customer_id, menu.product_name, sales.order_date
    FROM dannys_diner.menu menu
    JOIN dannys_diner.sales sales
    ON menu.product_id = sales.product_id
    WHERE order_date in 
    (SELECT MIN(sales.order_date)
    FROM dannys_diner.menu menu
    JOIN dannys_diner.sales sales
    ON menu.product_id = sales.product_id
    GROUP BY sales.customer_id);
```
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| B           | curry        | 2021-01-01T00:00:00.000Z |
| C           | ramen        | 2021-01-01T00:00:00.000Z |
| C           | ramen        | 2021-01-01T00:00:00.000Z |


## Problem 4
    What is the most purchased item on the menu and how many times was it purchased by all customers?
### Solution
```sql
    SELECT menu.product_name, COUNT(menu.product_name) AS total_count
    FROM dannys_diner.menu menu
    JOIN dannys_diner.sales sales
    ON menu.product_id = sales.product_id
    GROUP BY 1
    LIMIT 1;
```
| product_name | total_count |
| ------------ | ----------- |
| ramen        | 8           |

## Problem 5
    Which item was the most popular for each customer?
### Solution
```sql
    WITH first_CTE AS 
    (
      SELECT sales.customer_id, menu.product_name, DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS rank 
    FROM dannys_diner.menu menu
    JOIN dannys_diner.sales sales
    ON menu.product_id = sales.product_id
    GROUP BY sales.customer_id, menu.product_name
    )
    
    SELECT customer_id, product_name
    FROM first_CTE
    WHERE rank = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |

## Problem 6
    Which item was purchased first by the customer after they became a member?
### Solution
```sql
    WITH second_CTE AS 
    (
      SELECT sales.customer_id, menu.product_name, sales.order_date, DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS order_time 
     FROM dannys_diner.sales sales
    JOIN dannys_diner.menu menu
    ON menu.product_id = sales.product_id
    JOIN dannys_diner.members members
    ON sales.customer_id = members.customer_id
    WHERE sales.order_date >= members.join_date   
    )
    
    SELECT customer_id, product_name, order_date
    FROM second_CTE
    WHERE order_time = 1;
```
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | curry        | 2021-01-07T00:00:00.000Z |
| B           | sushi        | 2021-01-11T00:00:00.000Z |


## Problem 7
    Which item was purchased just before the customer became a member?
### Solution
```sql
    WITH third_cte AS 
    (
     SELECT sales.customer_id,
     sales.product_id,
     sales.order_date,
     DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS time_rank 
     FROM dannys_diner.sales AS sales
     JOIN dannys_diner.members AS members
      ON sales.customer_id = members.customer_id
     WHERE sales.order_date < members.join_date
    )
    
    SELECT customer_id,
    product_name,
    order_date 
    FROM third_cte sales
    JOIN dannys_diner.menu menu
    ON sales.product_id = menu.product_id
    WHERE time_rank = 1
    ORDER BY 1;
```
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| B           | sushi        | 2021-01-04T00:00:00.000Z |


## Problem 8
    What is the total items and amount spent for each member before they became a member?
### Solution
```sql
    SELECT sales.customer_id, COUNT(sales.product_id), SUM(menu.price) 
    FROM dannys_diner.sales sales
    JOIN dannys_diner.menu menu
    ON sales.product_id = menu.product_id
    JOIN dannys_diner.members members
    ON sales.customer_id = members.customer_id
    WHERE sales.order_date < members.join_date
    GROUP BY 1
    ORDER BY 1;
```
| customer_id | count | sum |
| ----------- | ----- | --- |
| A           | 2     | 25  |
| B           | 3     | 40  |

## Problem 9
    If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
### Solution
```sql
    SELECT sales.customer_id, 
    SUM(CASE WHEN menu.product_name = 'sushi' THEN 20 * menu.price ELSE 10 * menu.price END) AS total_points
    FROM dannys_diner.sales sales
    JOIN dannys_diner.menu menu
    ON sales.product_id = menu.product_id
    GROUP BY 1
    ORDER BY 1;
```
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

## Problem 10
    In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
### Solution
```sql
    SELECT sales.customer_id,
    sales.order_date, members.join_date,  menu.product_name, menu.price,
    SUM(CASE WHEN menu.product_name = 'sushi' THEN 20 * menu.price 
        WHEN sales.order_date BETWEEN  members.join_date AND members.join_date + INTERVAL '6 days' THEN 2 * 10 * menu.price ELSE 10 * menu.price END) AS total_points
                     
    FROM dannys_diner.sales sales
    JOIN dannys_diner.menu menu
    ON sales.product_id = menu.product_id
    JOIN dannys_diner.members members
    ON sales.customer_id = members.customer_id
    WHERE sales.order_date::date < '2021-01-31'
    GROUP BY 1,2,3,4,5;
```
| customer_id | order_date               | join_date                | product_name | price | total_points |
| ----------- | ------------------------ | ------------------------ | ------------ | ----- | ------------ |
| A           | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | curry        | 15    | 150          |
| A           | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | sushi        | 10    | 200          |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z | curry        | 15    | 300          |
| A           | 2021-01-10T00:00:00.000Z | 2021-01-07T00:00:00.000Z | ramen        | 12    | 240          |
| A           | 2021-01-11T00:00:00.000Z | 2021-01-07T00:00:00.000Z | ramen        | 12    | 480          |
| B           | 2021-01-01T00:00:00.000Z | 2021-01-09T00:00:00.000Z | curry        | 15    | 150          |
| B           | 2021-01-02T00:00:00.000Z | 2021-01-09T00:00:00.000Z | curry        | 15    | 150          |
| B           | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z | sushi        | 10    | 200          |
| B           | 2021-01-11T00:00:00.000Z | 2021-01-09T00:00:00.000Z | sushi        | 10    | 200          |
| B           | 2021-01-16T00:00:00.000Z | 2021-01-09T00:00:00.000Z | ramen        | 12    | 120          |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
