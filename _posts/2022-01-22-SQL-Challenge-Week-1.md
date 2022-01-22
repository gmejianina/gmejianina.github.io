---
layout: post
title: SQL Challenge - Week 1
categories: [SQL]
excerpt: Danny has contacted us to help him answer some questions about his customers. He wants to know about their visiting and spending patterns, as well as their favorite items. He will use these insights to decide whether or not to expand his customer loyalty program.
---

![Danny's Diner Image](https://8weeksqlchallenge.com/images/case-study-designs/1.png)

Danny has contacted us to help him answer some questions about his customers. He wants to know about their visiting and spending patterns, as well as their favorite items. He will use these insights to decide whether or not to expand his customer loyalty program. 

If you would like to see the detailed premise of this case study, [click here](https://8weeksqlchallenge.com/case-study-1/)

**1. What is the total amount each customer spent at the restaurant?**

Customer A has spent the most, with a total of $76.

Customer B is close behind with a total of $74.

Customer C has spent the least, with a total of $36.


    SELECT
      	customer_id
        ,SUM(m.price) as TOTAL --We need to bring the price from the menu table
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m ON s.product_id = m.product_id
    GROUP BY customer_id
    
    ORDER BY SUM(m.price) DESC; -- Let's have the customer with the highest spending total at the top!

| customer_id | total |
| ----------- | ----- |
| A           | 76    |
| B           | 74    |
| C           | 36    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**2. How many days has each customer visited the restaurant?**

Customer B has visited our restaurant the most, coming 6 days. 

Customer A comes in second place, coming 4 days.

Customer C has only visited the restaurant twice.

    SELECT
      	customer_id
        ,COUNT(DISTINCT order_date) Days_Visited
    FROM dannys_diner.sales
    
    GROUP BY customer_id
    ORDER BY COUNT(DISTINCT order_date) DESC;

| customer_id | days_visited |
| ----------- | ------------ |
| B           | 6            |
| A           | 4            |
| C           | 2            |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**3. What was the first item from the menu purchased by each customer?**

Customer A had sushi has their first item.

Customer B had curry, while Customer C had ramen.

Each of our 3 customers had a distinct first dish at Danny's restaurant.


    WITH CTE AS (
    SELECT *,
      ROW_NUMBER () OVER (PARTITION BY customer_id ORDER BY order_date) as Occurrence
    
    FROM dannys_diner.sales
                 )
    
    SELECT
      	s.customer_id
        ,m.product_name as "First Purchase"
    FROM CTE s
    LEFT JOIN dannys_diner.menu m ON s.product_id = m.product_id
    WHERE s.Occurrence = 1;

| customer_id | First Purchase |
| ----------- | -------------- |
| A           | sushi          |
| B           | curry          |
| C           | ramen          |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

The most purchased item was ramen, with 8 purchases in total, doubling the amount of orders of other items.


    SELECT
    COUNT(SALES.PRODUCT_ID),
    MENU.PRODUCT_NAME
    
    FROM dannys_diner.SALES 
    LEFT JOIN dannys_diner.MENU ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
    
    GROUP BY MENU.PRODUCT_NAME
    
    ORDER BY COUNT(SALES.PRODUCT_ID) DESC;

| count | product_name |
| ----- | ------------ |
| 8     | ramen        |
| 4     | curry        |
| 3     | sushi        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


**5. Which item was the most popular for each customer?**

Ramen is the most popular item for customers A and C. However, B has ordered all items equally. 


    WITH CTE AS (
      SELECT CUSTOMER_ID,
      PRODUCT_ID,
      COUNT(PRODUCT_ID) AS AMOUNT,
      DENSE_RANK () OVER (PARTITION BY CUSTOMER_ID ORDER BY COUNT(PRODUCT_ID) DESC) AS MOST_POPULAR
      
      FROM dannys_diner.SALES
      GROUP BY CUSTOMER_ID, PRODUCT_ID
     )
    
    SELECT
    CTE.CUSTOMER_ID,
    CTE.AMOUNT,
    MENU.PRODUCT_NAME
    
    FROM CTE 
    LEFT JOIN dannys_diner.MENU
    ON CTE.PRODUCT_ID = MENU.PRODUCT_ID
    
    WHERE CTE.MOST_POPULAR = 1
    
    ORDER BY CTE.CUSTOMER_ID;

| customer_id | amount | product_name |
| ----------- | ------ | ------------ |
| A           | 3      | ramen        |
| B           | 2      | sushi        |
| B           | 2      | curry        |
| B           | 2      | ramen        |
| C           | 3      | ramen        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


**6. Which item was purchased first by the customer after they became a member?**

We only have 2 members: A and B.

Customer A's first purchase after becomming a member was ramen, while Customer B's was sushi.


    WITH CTE AS (SELECT sales.customer_id,
     		sales.product_id,
      sales.order_date,
      ROW_NUMBER () OVER (PARTITION BY sales.CUSTOMER_ID ORDER BY sales.ORDER_DATE) Members_First
      FROM
      dannys_diner.sales
      left join dannys_diner.members 
      ON sales.customer_id = members.customer_id
      where order_date > join_date ) 
      
    SELECT  
    CTE.customer_id,
    menu.product_name
    FROM CTE
    LEFT JOIN dannys_diner.menu
    ON CTE.product_id = menu.product_id
    WHERE members_first = 1;

| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | ramen        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


**7. Which item was purchased just before the customer became a member?**

Right before becomming a member, customer B ordered sushi, while customer A ordered both sushi and curry.


    WITH CTE AS (
      
      SELECT sales.customer_id,
     		sales.product_id,
      sales.order_date,
      RANK () OVER (PARTITION BY sales.CUSTOMER_ID ORDER BY sales.ORDER_DATE DESC) Before_Member
      FROM
      dannys_diner.sales
      left join dannys_diner.members 
      ON sales.customer_id = members.customer_id
      where order_date < join_date
    
    ) 
      
    SELECT  
    CTE.customer_id,
    menu.product_name
    FROM CTE
    LEFT JOIN dannys_diner.menu
    ON CTE.product_id = menu.product_id
    WHERE before_member = 1;

| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | sushi        |
| A           | curry        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**8. What is the total items and amount spent for each member before they became a member?**

Before becomming a member:

Customer B spent $40 and purchased 3 items

Customer A spent $25 and purchased 2 items


    SELECT sales.customer_id,
     		count(sales.product_id),
      SUM(menu.price)
      FROM
      dannys_diner.sales
      left join dannys_diner.members 
      ON sales.customer_id = members.customer_id
      left join dannys_diner.menu
      on sales.product_id = menu.product_id
      where order_date < join_date
    
    group by sales.customer_id;

| customer_id | count | sum |
| ----------- | ----- | --- |
| B           | 3     | 40  |
| A           | 2     | 25  |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

Customer B would have 940 points

Customer A would have 860 points

Customer C would have 360 points


    SELECT sales.customer_id,
    		SUM (CASE WHEN sales.product_id = 1 then (menu.price*20)
                 else (menu.price*10) END) as points
      FROM
      dannys_diner.sales
    
      left join dannys_diner.menu
      on sales.product_id = menu.product_id
    
    group by sales.customer_id;

| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| C           | 360    |
| A           | 860    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

At the end of January

Customer A has 1370 points

Customer B has 940 points

    SELECT sales.customer_id,
    		SUM (CASE WHEN sales.order_date between members.join_date and (members.join_date + 7) then (menu.price*20) 
                 when sales.order_date not between members.join_date and (members.join_date + 7) and sales.product_id = 1 then (menu.price*20)
                 else (menu.price*10) END) as points
      FROM
      dannys_diner.members
      left join dannys_diner.sales
      on sales.customer_id = members.customer_id
      left join dannys_diner.menu
      on sales.product_id = menu.product_id
      
      where sales.order_date < '2021-02-01'
    
    group by sales.customer_id;

| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 940    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**Bonus**

Let's give Danny and his team a ready-made table to get quick insights! 

The table below will allow his team to answer questions without the need for complex queries.



    CREATE TABLE insights AS
      SELECT 
      sales.customer_id,
      sales.order_date,
      menu.product_name,
      menu.price,
      
     CASE WHEN ranking.rank is not null then 'Y' else 'N' END AS "member",
     ranking.rank 
      
      FROM dannys_diner.sales
      LEFT JOIN dannys_diner.menu 
      ON sales.product_id = menu.product_id
      LEFT JOIN (SELECT 
      sales.customer_id,
      sales.order_date,
      sales.product_id,
      DENSE_RANK () OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS "rank"
      
      FROM dannys_diner.sales
      LEFT JOIN dannys_diner.members
      ON sales.customer_id = members.customer_id
      
      WHERE sales.order_date >= members.join_date
      GROUP BY sales.customer_id, sales.order_date, sales.product_id 
    ) ranking
      ON sales.customer_id = ranking.customer_id and sales.order_date = ranking.order_date
      
      ORDER BY sales.customer_id, sales.order_date
      ; 
    
      
      

---



    SELECT * FROM dannys_diner.insights;

| customer_id | order_date               | product_name | price | member | rank |
| ----------- | ------------------------ | ------------ | ----- | ------ | ---- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1    |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2    |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1    |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2    |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3    |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
