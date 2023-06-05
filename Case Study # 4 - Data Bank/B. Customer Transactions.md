## B Customer Transactions
1. What is the unique count and total amount for each transaction type?
```sql
SELECT * FROM data_bank.customer_transactions;*/
SELECT txn_type,count(txn_type) as count, sum(txn_amount) as txn_amnt
FROM data_bank.customer_transactions
GROUP BY txn_type
order by sum(txn_amount) desc;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/864113d9-149f-4226-b21e-7a99e5543e27)


2.What is the average total historical deposit counts and amounts for all customers?
```sql
SELECT * FROM data_bank.customer_transaction*/
with cte as(
  		SELECT 
  		distinct customer_id, 
  		count(txn_date),
  		avg(txn_amount) as amnt 
from data_bank.customer_transactions
where txn_type = 'deposit'
group by 1)
select floor(avg(count)) as avg_deposit ,
	   ceil(avg(amnt)) as avg_amount from cte;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/9337caa6-eedc-4e28-8e18-88d7ab3c38a3)

3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
with ct as(SELECT customer_id, DATE_TRUNC( 'mon',txn_date) AS new_txn_date,
SUM(CASE WHEN txn_type= 'deposit' then 1 else 0 end) as dep_count,
SUM(CASE WHEN txn_type='withdrawal' then 1 else 0 end) as with_count,
SUM (CASE WHEN txn_type='purchase' then 1 else 0 end) as pur_count 
     from data_bank.customer_transactions
group by customer_id,2)

SELECT new_txn_date,COUNT(customer_id), 
	from ct
    where dep_count > 1 and (with_count >=1 or pur_count>=1)
    group by new_txn_date
    order by new_txn_date asc;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/a88cbdc5-e747-454a-bcf5-65f06a05d656)

4.  What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.
```sql
 with ending_month as
	(SELECT customer_id, 
     (DATE_TRUNC('months', txn_date) + INTERVAL '1 month - 1 day' )::date as ending_month_date, 	txn_amount,
	CASE WHEN txn_type in ('withdrawal','purchase') then (-txn_amount) else txn_amount end as transaction_balance
from data_bank.customer_transactions
group by customer_id,	
     	(DATE_TRUNC('months', txn_date) + INTERVAL '1 month - 1 day' ) ,3, 4
order by customer_id,2 asc),

txn_month as(
  SELECT distinct customer_id, 
  ( '2020-01-31T00:00:00.000Z'::DATE + GENERATE_SERIES(0,3) * INTERVAL ' 1 MONTH')  as end_month_date
  from data_bank.customer_transactions
  group by 1,2
  order by 1,2 asc),

temp as(SELECT b.customer_id, 
		b.end_month_date,
        COALESCE(a.transaction_balance,0) as monthly_balance, 
        SUM(a.transaction_balance) over(partition by b.customer_id order by b.end_month_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as closing_balance
FROM txn_month b left JOIN ending_month a
on  b.end_month_date =  a.ending_month_date  
and b.customer_id= a.customer_id),

row_number_dates as(
  SELECT customer_id, 
  		end_month_date, 
  		monthly_balance, 
  		closing_balance,
row_number() over(partition by customer_id, end_month_date order by end_month_date asc) as row_num 
from temp),

lead_row_num as(
  		SELECT customer_id, 
  				end_month_date, 
  				monthly_balance, 
  				closing_balance, 
  				row_num,
				LEAD(row_num) OVER(PARTITION BY customer_id, end_month_date order by end_month_date) as lead_row
from row_number_dates)

SELECT customer_id, 
		end_month_date, 
        monthly_balance, 
        closing_balance
FROM lead_row_num
where lead_row is null
GROUP BY customer_id, end_month_date, monthly_balance, closing_balance
ORDER BY 1,2,3 asc;
```
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/dccfc45c-1eb0-45cb-a7cc-57837303779f)



 
