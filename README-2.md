
# Supply Chain Analysis

## Introduction

Supply chain had become a fundamental focus of today's business. Untill you have a supply and demand equilibrium exist, business will be sustainable and growth will be ensured there. From this perspective, the problem statement is here related to a company in the fmcg domain has been facing issues in their supply chain function and this leads to a disruption in their regular operations.

## Problem statement
Atliq is a growing FMCG manufacturing company, operations in 3 major cities and expected to expand their business in tier 1 cities in next 2 years. However, due to their operations failure and ineffeciency, their customers showing unwillingness to renew their annual contract for that.

## Objective 
The objective of the analysis is to provide insights around key important supply chain metrics to uncover the reason behind the customers not renweing their contract.

## Project Requests
In search for core challenges in the supply chain operations, the project tried to answer if the customers got delivered their orders in full quantity, on time and on time & full quantity. Also, to know the company's capacity to fulfull it's customer request, here line and volume fill rate metrics have been considered

Here is the questions i tried to answered through sql queries:

- What is the 'On time' , 'In Full' and 'OTIF' percentage for orders based on customers and    cities?
- What is the line fill rate and volume fill rate by customer and city?
- What is the gap between service level metrics and target level?
- What is the OTIF performance vs target over months?
- Show the change of LIFR and VOFR for products.

## Tools I used
For the deep dive into the data analyst of company's performance, I harnessed the power of several key tools:

- SQL: The backbone of my analysis, allowing me to query the database and unearth critical insights.
- MySQL: The chosen database management system, ideal for handling the job posting data.
- MySQL Workbench: My go-to for database management and executing SQL queries.
- Git & GitHub: Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.

## The Analysis

The queries are aimed to evaluate daily business performance of busiess through multiple key metries.

Here is how approach to each asked questios:

#### 1. Who are the biggest customers for the company?

To know the the customers contribution in the total orders, the query intended to know the biggest customers and their order count and quantity of products being ordered

```sql
select 
       customers.customer_name,
       count(distinct(fl.order_id)) as order_count,
       sum(fl.order_qty) as order_quantity
from fact_order_lines as fl
left join dim_customers as customers
using(customer_id)
group by customers.customer_name
order by order_count desc;

```
#### Here's a breakdown of the results for top customers:

- Lotus Mart, Accalaimed Stores, Vijay Stores are the top 3 customers 
- The top 5 contributes the 45% of total orders

#### 2. What are the top 3 products in each category by delived quantity

To get an understanding about top 3 selling products in each category

```sql
with stats as (
select p.category,
	   p.product_name,
       sum(fl.delivery_qty) as delivery_qty,
       rank() over(partition by p.category order by sum(fl.delivery_qty)) as rnk
from fact_order_lines as fl
left join dim_products as p
using(product_id)
group by p.category, p.product_name
)

select category,
       product_name,
       delivery_qty,
       rnk
from stats
where rnk <= 3;

```
Here's the breakdown of the customers order shows the OTIF variance from target:
- 


#### 3. What is  OT%, IF%, OTIF% of orders by existing customers

To understand the service, customers are getting from the company, i tried to evaluate the delivered KPI against the target KPI to get into the insight of difference.

```sql

with stats as (
select
       customers.customer_name,
       round((sum(case when orders.on_time = 1 then 1 else 0 end) * 100 / count(orders.order_id)),2) as actual_ot,
       round(avg(dt.ontime_target_percentage),2) as target_ot,
       round((sum(case when orders.in_full = 1 then 1 else 0 end) * 100 / count(orders.order_id)),2) as actual_if,
       round(avg(dt.infull_target_percentage),2) as target_if,
       round((sum(case when orders.otif = 1 then 1 else 0 end) * 100 / count(orders.order_id)),2) as actual_otif,
       round(avg(dt.otif_target_percentage),2) as target_otif
from fact_orders_aggregate as orders
left join dim_target_orders as dt
using(customer_id)
left join dim_customers as customers
using(customer_id)
group by customers.customer_name
)

select customer_name,
	   concat((target_ot - actual_ot), '%')  as ot_variance,
       concat((target_if - actual_if), '%') as if_variance,
       concat((target_otif - actual_otif), '%') as otif_variance
from stats
order by otif_variance desc:

```
Here's the breakdown of the customers order shows the OTIF variance from target:
- The higher the difference represts higher performance issue in the supply chain function
- In the higher OTIF score list, top 2-3 customers made their place those contributes most in the order count
- Customers are getting delayed in deliverying their orders as well as not getting their desired quantity

#### 4. What is  OT%, IF%, OTIF% of orders by cities?

To understand which KPI has the higest variance so the dominant KPI can be outlined.

```sql
with stats as (
select customers.city,
       (sum(case when orders.on_time = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as OT,
       avg(dt.ontime_target_percentage) as target_OT,
       (sum(case when orders.in_full = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as 'IF',
       avg(dt.infull_target_percentage) as target_IF,
       (sum(case when orders.otif = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as OTIF,
       avg(dt.otif_target_percentage) as target_OTIF
from fact_orders_aggregate as orders
left join dim_customers as customers
using(customer_id)
left join dim_target_orders as dt
using(customer_id)
group by customers.city
)

select city,
       concat((target_OT - OT), '%') as OT_variance,
       concat((target_IF -'IF'), '%') as IF_variance,
       concat((target_OTIF - OTIF), '%') as OTIF_variance
from stats
group by city;

```
Here's the breakdown of the customers order shows the OTIF variance from target:

- OTIF and OT among Vadodara, Ahmedabad, Surat cities have been similiar which indicates the service level is consistant
- In full delivery has the higher variance for 3 cities and demonstrate that business in these cities ain't getting their ordered quantity in full


#### 5. What is  OT%, IF%, OTIF% of orders by cities?

To understand which KPI has the higest variance so the dominant KPI can be outlined.

```sql
with stats as (
select customers.city,
       (sum(case when orders.on_time = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as OT,
       avg(dt.ontime_target_percentage) as target_OT,
       (sum(case when orders.in_full = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as 'IF',
       avg(dt.infull_target_percentage) as target_IF,
       (sum(case when orders.otif = 1 then 1 else 0 end) * 100 / count(orders.order_id)) as OTIF,
       avg(dt.otif_target_percentage) as target_OTIF
from fact_orders_aggregate as orders
left join dim_customers as customers
using(customer_id)
left join dim_target_orders as dt
using(customer_id)
group by customers.city
)

select city,
       concat((target_OT - OT), '%') as OT_variance,
       concat((target_IF -'IF'), '%') as IF_variance,
       concat((target_OTIF - OTIF), '%') as OTIF_variance
from stats
group by city;

```
Here's the breakdown of the customers order shows the OTIF variance from target:

- OTIF and OT among Vadodara, Ahmedabad, Surat cities have been similiar which indicates the service level is consistant
- In full delivery has the higher variance for 3 cities and demonstrate that business in these cities ain't getting their ordered quantity in full

#### 6. What is  LIFR and VOFR for each customer ?

To understand the business ability for order fulfillment performance and how much volume delived in comparison to the order quantity.

```sql
select c.customer_name,
	   concat(round(sum(case when fl.in_full = 1 then 1 else 0 end) * 100 / count(fl.order_id),2),"%") as LIFR,
       concat(round(sum(fl.delivery_qty) * 100 / sum(fl.order_qty),2), "%") as VOFR
from fact_order_lines as fl
left join dim_customers as c
using(customer_id)
group by c.customer_name
order by LIFR desc;

```
Here's the breakdown of the LIFR and VOFR by each customers:
- max line fill rate is 75.62%, followed by min 51.53%
- Average vofr above 96% indicates they delived most of the order quantity of products
- Although, LIFR shows the vulrability that on average 53% order line has product shortage to fulfill the order line

#### 7. Analyze the monthly trend of on time delivery

To explore about the month on month consistency of on time delivery

```sql
select 
    month(order_placement_date) as month,
    count(order_id) AS Total_orders,
    sum(case when fact_order_lines.On_Time = 1 then 1   else 0 end) as on_time_orders,
    round((SUM(case when fact_order_lines.On_Time = 1 then 1 else 0 end) / count(fact_order_lines.order_id) *100 ),2) as on_time_pct 
from fact_order_lines 
group by month(order_placement_date)
order by 
month(order_placement_date);
```
Here's the breakdown of data understanding:

- There is continuous failture of on time delivery of orders
- Avg target across the customers are 86% and no improvemnts have found

#### 8. What is the average lead time and delay delivery days for each cusotmer? ?

To see the agility of delivery system of the company for their customer

```sql
select c.customer_name,
       round(avg(datediff(fl.agreed_delivery_date,fl.order_placement_date)),2) as lead_time,
       avg(datediff(fl.actual_delivery_date,fl.agreed_delivery_date)) as delay_days
from fact_order_lines as fl
left join dim_customers as c
using(customer_id)
group by c.customer_name
order by delay_days desc;

```
Here's the breakdown of the aveage lead time and delivery delay :

- The average lead time around 2 days for the customers
- Lotus, Coolblue, Accalaimed store being the top customers have dalays in delivery
- This makes on an average 1.69 days of product delivery delay 

#### 9. Product wise LIFR and OTIF Calculation

Explore the breakdown of product wise performance

```sql
select p.product_name,
       concat(round(sum(case when fl.in_full = 1 then 1 else 0 end) * 100 / count(fl.order_id),2),"%") as LIFR,
       concat(round(sum(case when fa.otif = 1 then 1 else 0 end) * 100 / count(fa.order_id),2),"%") as OTIF
from fact_order_lines as fl
left join fact_orders_aggregate as fa
using(customer_id)
left join dim_products as p
on p.product_id = fl.product_id
group by p.product_name
order by OTIF asc

```
Here's the breakdown:
- Each of product has lower line fill rate than expected 
- OTIF performance has been comprosed drastically

#### 10. 

## What I have learned 
Througout the whole project, I turbocharged my SQL querying skills with some serious firepower

✅ Mastered the art of writting complex sql queries which includes using CTE function
✅ Joining multiple table together with the use of case statement and calculate the avg, max, min from it
✅ Pulling out actionable insights from data to put value on the analysis
✅ Breaking down a large problem into small pieces to get alighment with the objective

## Insights

From the analysis, several general actionable insights have emerged: 

- Unfortunately, OTIF for top 3 customers in the 3 cities was way low than target level as well as for other customers. This puts questions into company's ability to keep the partership

- The average delay in delivery is 1.69 days and on time delivery succedd only 70%. Company need to work on with shipping carrier to reduce the continuous delays over the months.

- LIFR and VOFR Comparison: Volume fill rate has been in a acceptabel margin. But LIFR is inconsisent across the customers which shows company has having stock issue to fulfill orders

-- 







