# 🍜 Case Study : Danny's Diner

<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-18%20185342.png" alt="Image" width="500" height="520">

## 📚 Table of Contents

- [SQL skills gained](#sql-skills-gained)
- [Case Study Description](#case-study-description)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Datasets](#datasets)
- [Question and Solution](#question-and-solution)

***

## SQL skills gained

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
**Steps:**

- **INNER JOIN** is used to merge `sales` and `menu` tables on `Product_id` column to combine the matching sales records with menu records.
- Result set of inner join is **GROUP BY** `sales.customer_id` and **SUM( )** is used to calculate the total amount spent by each customer.


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
	group by
                 customer_id,
		 order_date
   ) 
select 
      customer_id,
      count(order_date) as no_of_days_visited
from cte
group by customer_id;
```
**Steps:**

##### Common Table Expression (CTE):
- **CTE** is used to get unique visit days of each customer by grouping the records by `customer_id` and `order_date` columns.

##### Outer Query:
- Group the result set of CTE by `customer_id` column and **count( )** is used on the grouped records to calculate the total number of days each customer visited.

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

**Steps:**

##### Subquery: 
- **INNER JOIN** is used to Combine the `sales` and `menu` tables on `roduct_id` column to get product names for each sale.
- **PARTITION BY** `s.customer_id` creates a separate ranking for each customer and **ORDER BY** `s.order_date` ensures the orders are ranked chronologically.
- **ROW_NUMBER()** window function assigns a unique rank to each order per customer based on the order date.

##### Outer Query:
- **WHERE** Clause selects only the rows where the ranks is 1, which corresponds to the first order for each customer.


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
**Steps:**
- **INNER JOIN** is used to Combine the `sales` and `menu` tables on `product_id` column to get product names for each sale.
- **GROUP BY** Clause groups the results by product name and **COUNT( )** is used to Count the number of times each product was purchased. The result is given an alias `total_purchased` for clarity.
- **ORDER BY** Clause orders the results by the total number of purchases in descending order, so the most purchased product comes first.
- **TOP** 1, Limits the result to the first row, which will be the most frequently purchased product.


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

**Steps:**

##### Common Table Expression (CTE):
- **GROUP BY** Clause and **count(*)** helps in determining the order count of each product by each customer.
- **dense_rank( )** assign a rank to each product ordered by each customer based on the order count.

##### Outer Query:
- **INNER JOIN** is used to Combine the `cte` result set and `menu` tables on `product_id` column to get product names for each sale.
- **WHERE** Clause selects only the rows where the ranks is 1, indicating the most frequently purchased product for each customer.

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

```sql
with cte as
(
	select
               s.customer_id,
	       s.order_date,
	       s.product_id,
	       ROW_NUMBER() over(partition by s.customer_id
	       order by s.order_date) as ranks
	from sales s
	join members m
	on s.customer_id = m.customer_id
	where s.order_date >= m.join_date
)
select c.customer_id,
       c.order_date,
       m.product_name
from cte c
join menu m
on c.product_id = m.product_id
where ranks = 1;
```
**Steps:**

##### Common Table Expression (CTE):
- **INNER JOIN** matchs the `customer_id` from the `sales` table with the `customer_id` in the `members` table to identify customers who joined as members.
- **WHERE** Clause helps in considering only the orders which are made after a customer's join date as member.
- **ROW_NUMBER()** assigns a unique rank to each order per customer based on the order date.

##### Outer Query:
- **INNER JOIN** is used to Combine the `cte` result set and `menu` tables on `product_id` column to get product names for each sale.
- **WHERE** Clause selects only the rows where the ranks is 1, indicating the first product purchased by each customer after joining as a member.


**Solution :**
customer_id|order_date|product_name
---|---|---
A|2021-01-07|curry
B|2021-01-11|sushi

***

**7. Which item was purchased just before the customer became a member?**

```sql
with cte as
(
	select
	       s.customer_id,
	       s.order_date,
	       s.product_id,
	       ROW_NUMBER() over(partition by s.customer_id
	       order by s.order_date) as ranks
	from sales s
	join members m
	on s.customer_id = m.customer_id
	where s.order_date < m.join_date
),
cte1 as
(
	select
	       c.customer_id,
	       c.order_date,
	       m.product_name,
	       ROW_NUMBER() over(partition by c.customer_id
	       order by ranks desc) as ranks1
	from cte c
	join menu m
	on c.product_id = m.product_id
)
select
       ct.customer_id,
       ct.order_date,
       ct.product_name
from cte1 ct
where ranks1 = 1 ;
```

**Solution :**
customer_id|order_date|product_name
---|---|---
A|2021-01-01|curry
B|2021-01-04|sushi

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
with cte as 
(
	select
	       s.customer_id,
               s.order_date,
               s.product_id,
               me.join_date,
               m.product_name,
               m.price 
	from sales s
	join members me
	on s.customer_id = me.customer_id
	join menu m
	on s.product_id = m.product_id
	where s.order_date < me.join_date 
)
select
       customer_id,
       count(product_name) as total_products,
       sum(price) as total_amount_spent
from cte
group by customer_id;
```
**Solution :**
customer_id|total_products|total_amount_spent
---|---|---
A|2|25
B|3|40

***

**9. Join All The Things, Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

```sql
select
	s.customer_id,
	s.order_date,
	m.product_name,
	m.price,
	case when s.order_date >= ms.join_date
	then 'Y' else 'N' end as member 
from sales s                                                       
join menu m                                                         
on s.product_id = m.product_id                                      
left join members ms
on s.customer_id = ms.customer_id
```

**Solution :**
customer_id|order_date|product_name|price|member
---|---|---|---|---
A	|2021-01-01|	sushi|	10|	N
A	|2021-01-01|	curry|	15|	N
A	|2021-01-07|	curry|	15|	Y
A	|2021-01-10|	ramen|	12|	Y
A	|2021-01-11|	ramen|	12|	Y
A	|2021-01-11|	ramen|	12|	Y
B	|2021-01-01|	curry|	15|	N
B	|2021-01-02|	curry|	15|	N
B	|2021-01-04|	sushi|	10|	N
B	|2021-01-11|	sushi|	10|	Y
B	|2021-01-16|	ramen|	12|	Y
B	|2021-02-01|	ramen|	12|	Y
C	|2021-01-01|	ramen|	12|	N
C	|2021-01-01|	ramen|	12|	N
C	|2021-01-07|	ramen|	12|	N

***

**10. Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

```sql
with cte as
(
	select 
 	       s.customer_id,
	       s.order_date,
	       m.product_name,
	       m.price,
	       case when s.order_date >= ms.join_date 
	       then 'Y' else 'N' end as [members]
	from sales s                                                        
	join menu m                                                        
	on s.product_id = m.product_id                                     
	left join members ms
	on s.customer_id = ms.customer_id
)
select 
	customer_id,
	order_date,
	product_name,
	price,
	[members],
	case when [members] = 'N' then NULL 
	else DENSE_RANK() over(partition by customer_id,[members] 
	order by order_date) end as ranking
from cte;
```

**Solution :**

customer_id|order_date|product_name|price|members|ranking
---|---|---|---|---|---
A	|2021-01-01|	sushi|	10|	N|NULL
A	|2021-01-01|	curry|	15|	N|NULL
A	|2021-01-07|	curry|	15|	Y|1
A	|2021-01-10|	ramen|	12|	Y|2
A	|2021-01-11|	ramen|	12|	Y|3
A	|2021-01-11|	ramen|	12|	Y|3
B	|2021-01-01|	curry|	15|	N|NULL
B	|2021-01-02|	curry|	15|	N|NULL
B	|2021-01-04|	sushi|	10|	N|NULL
B	|2021-01-11|	sushi|	10|	Y|1
B	|2021-01-16|	ramen|	12|	Y|2
B	|2021-02-01|	ramen|	12|	Y|3
C	|2021-01-01|	ramen|	12|	N|NULL
C	|2021-01-01|	ramen|	12|	N|NULL
C	|2021-01-07|	ramen|	12|	N|NULL

***



