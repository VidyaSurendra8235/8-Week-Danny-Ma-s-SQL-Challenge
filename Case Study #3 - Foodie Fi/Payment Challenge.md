## C. Payment Challenge
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

1.monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
2.upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
3.upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
4.once a customer churns they will no longer make payments

```sql
ALTER TABLE foodie_fi.subscriptions 
ADD COLUMN end_date 
DATE DEFAULT '2020-12-31';


with cte as(
  	SELECT customer_id, 
  		   plan_id,
           start_date, 
           end_date,
           LEAD(start_date, 1) OVER(PARTITION BY customer_id 
           ORDER BY start_date, plan_id) cutoff_date
    FROM foodie_fi.subscriptions
           where extract ('year' from start_date) = 2020),

temp as(
  	SELECT c.customer_id, 
  			c.plan_id, 
  			p.plan_name, 
  			c.start_date,
			(CASE WHEN cutoff_date is null then end_date else cutoff_date end) as plan_end_date, 
  			p.price
	FROM cte c join 
  		 foodie_fi.plans p
	ON c.plan_id = p.plan_id 
	WHERE p.plan_id not in (0,4)
	GROUP BY c.customer_id,
  			 c.plan_id,
             p.plan_name,
             c.start_date, 5, 
             p.price
   ORDER BY c.customer_id,
            c.plan_id,
            p.plan_name asc),

 temp_table as( 
  SELECT    customer_id, 
  			plan_id, 
  			plan_name, 
  			start_date, 
  			price
  from temp
            WHERE plan_id = 3),
              
 
temp1 as(
  SELECT customer_id, 
  		plan_id, 
        plan_name, 
        start_date, 
        DATE_PART('mon', age(plan_end_date-1, start_date))::INTEGER AS months, price
        FROM temp
        WHERE plan_id != 3),

payment as(
  	SELECT customer_id, 
           plan_id, plan_name, price,
           (start_date + GENERATE_SERIES(0, months) * INTERVAL '1 month')::DATE start_date
           from temp1),

join_table as
	(SELECT 
     	customer_id, 
     	plan_id, 
     	plan_name, 
     	start_date, price FROM temp_table
UNION
   SELECT 
     	customer_id, 
     	plan_id, 
     	plan_name, 
     	start_date, 
     price FROM payment
	GROUP BY 1,2,3,4,5
	ORDER BY 1,2,3 asc),

table1 as(
  SELECT customer_id, plan_id, 
	LAG(plan_id,1) OVER(PARTITION BY customer_id ORDER BY customer_id, plan_id) lag_plan_id , 
   plan_name, price, start_date
       FROM join_table),
     
    
 table2 as(
   SELECT customer_id, plan_id, plan_name, start_date,
   CASE WHEN plan_id IN(2,3) and lag_plan_id = 1 then price - 9.90
           else price 
           end as amount, 
   ROW_NUMBER() OVER(PARTITION BY customer_id order by customer_id, plan_id,start_date asc) as payment_order
    from table1)
 SELECT customer_id, 
 		plan_id, 
        plan_name, 
        start_date::VARCHAR as payment_date, 
        amount, 
        payment_order 
     FROM table2;
   
```
## OUTPUT
![image](https://github.com/VidyaSurendra8235/6-Week-SQL-Challenge/assets/107226432/77c75549-6fad-4e50-846b-e91489911635)
