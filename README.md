# 🍜 Case Study : Danny's Diner

<img src="https://github.com/Dharshan68/images/blob/main/Screenshot%202024-05-18%20185342.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [SQL Skills Gained](#sql-skills-gained)
- [Case Study Description](#case-study-description)
- [Entity Relationship Diagram](#entity-relationship-diagram)
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

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

```sql
select
  s.customer_id,
  sum(price) as total_amount
from sales s
inner join menu m
on s.product_id = m.product_id
group by s.customer_id;
```
**Solution :**
| customer_id | spent |
|-------------|-------|
| A           | 76    |
| B           | 74    |
| C           | 36    |

***

**2. How many days has each customer visited the restaurant?**


