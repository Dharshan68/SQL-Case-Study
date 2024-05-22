# 🍜 Case Study : Danny's Diner

<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-18%20185342.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [SQL Skills Gained](#sql-skills-gained)
- [Case Study Description](#case-study-description)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Datasets](#datasets)
- [Question and Solution](#question-and-solution)

***

## SQL Skills Gained

***

## Case Study Description
Danny's Diner is a small restaurant that has been collecting data on its customers, menu items, and sales. This case study involves analyzing the data to answer specific business questions and uncover trends that can help improve the diner's performance.

***

## Entity Relationship Diagram

![image](https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-18%20202858.png)

***

## Datasets

### Sales Table :

The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.

<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-22%20153555.png" alt="Image" width="500" height="520">

### Menu Table :

The menu table maps the product_id to the actual product_name and price of each menu item.
                                           
<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-22%20153625.png" alt="Image" width="400" height="150">

### Members Table :

The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.


<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-22%20153611.png" alt="Image" width="400" height="120">

***

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

```sql
select
        s.customer_id,
	sum(price) as total_amount_spent
from sales s
inner join menu m
on s.product_id = m.product_id
group by s.customer_id;
```
**Solution :**
| customer_id | total_amount_spent |
|-------------|-------|
| A           | 76    |
| B           | 74    |
| C           | 36    |

***

**2. How many days has each customer visited the restaurant?**

```sql
with cte as
   (
	select 
              customer_id,
              order_date
	from sales
	group by customer_id,order_date
   ) 
select 
      customer_id,
      count(order_date) as no_of_days_visited
from cte
group by customer_id;
```
**Solution :**
| customer_id | no_of_days_visited |
|-------------|--------|
| A           | 4      |
| B           | 6      |
| C           | 2      |

***

**3. What was the first item from the menu purchased by each customer?**
```sql
select 
      customer_id,
      order_date,
      product_name
from
(
	select
	       s.customer_id,
	       s.order_date,
	       m.product_name,
	       row_number() over(partition by s.customer_id
	       order by s.order_date) as ranks
	from sales s
	inner join menu m
	on s.product_id = m.product_id
) as x
where ranks = 1;
```
**Solution :**
| customer_id | order_date | product_name |           
|-------------|--------------|--------------|
| A           | 2021-01-01        |  sushi  |
| B           | 2021-01-01        |  curry  |
| C           | 2021-01-01        |  ramen  |

***

**4. What is the most purchased item on the menu?**
```sql
select
       top 1
       m.product_name,
       count(*) as total_purchased
from sales s
join menu m
on s.product_id = m.product_id
group by m.product_name
order by total desc;
```
**Solution :**
| product_name | total_purchased |
|--------------|-----------|
| ramen        | 8         |

***

**5. Which item was the most popular for each customer?**
```sql
with cte as
(
	select
	       customer_id,
               product_id,
               count(*) as order_count,
               dense_rank() over(partition by customer_id
               order by count(*) desc) as ranks
	from sales
	group by
	         customer_id,
                 product_id
)
select
       c.customer_id,
       c.order_count,
       m.product_name 
from cte c
join menu m
on c.product_id = m.product_id
where ranks = 1;
```
**Solution :**
customer_id|order_count|product_name
---|---|---
A|3|ramen
B|2|sushi
B|2|curry
B|2|ramen
C|3|ramen

***

**6. Which item was purchased first by the customer after they became a member?**



