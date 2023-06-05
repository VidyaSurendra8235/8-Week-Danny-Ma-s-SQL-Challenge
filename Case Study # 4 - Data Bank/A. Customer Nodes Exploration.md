## Customer Nodes Exploration Challenge
1.How many unique nodes are there on the Data Bank system?
```sql
SELECT COUNT(DISTINCT node_id) as nodes
from data_bank.customer_nodes;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/0c52ad02-6300-440d-9715-48c96ea77528)

2.What is the number of nodes per region?

```sql
SELECT distinct c.region_id, r.region_name, count(c.node_id) as node_count
from data_bank.customer_nodes c join data_bank.regions r
on c.region_id=r.region_id
group by c.region_id,r.region_name
order by c.region_id,r.region_name asc ;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/50bfabe7-5f33-46cc-9a8e-62c44406e277)

3.How many customers are allocated to each region?

```sql
SELECT distinct region_id, count(customer_id) as customer_count
from data_bank.customer_nodes
group by region_id
order by region_id asc;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/9ada7e4b-fecf-4ff4-bb13-249c1ea02a0b)

4.How many days on average are customers reallocated to a different node?

```sql

with cte as (SELECT customer_id, region_id, node_id,start_date, end_date,
CAST(end_date as DATE)-CAST(start_date as DATE) AS avg_days
from data_bank.customer_nodes
where end_date != '9999-12-31'
group by 1,2,3,4,5
order by 1 asc),

cte2 as(SELECT customer_id, region_id, node_id, avg_days,
LAG(avg_days,1) OVER(PARTITION BY customer_id ) as lag_days
from cte),

cte3 as(select customer_id, region_id, node_id, avg_days, 
case when lag_days is null then 0 else lag_days end as lag_days
from cte2),

cte4 as(select customer_id, region_id, avg(abs(avg_days-lag_days)) as days_avg
from cte3
group by 1,2
order by 1,2 asc)
select avg(days_avg) as average_days
from cte4;

SELECT customer_id, txn_date, txn_type,txn_amount FROM data_bank.customer_transactions
group by 1,2,3,4
order by 1,2 asc;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/d5ab2688-ff53-4e00-b38e-f6d113c8e542)
