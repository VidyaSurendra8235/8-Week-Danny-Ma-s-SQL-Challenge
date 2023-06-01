## D. Pricing and Ratings

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
SELECT SUM(CASE WHEN pizza_id=1 then 12
                WHEN pizza_id=2 then 10 END) as revenue
```
2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra
```sql
SELECT 1::INTEGER AS pizza_id, 12::INTEGER AS amount
  UNION ALL
SELECT 2::INTEGER AS pizza_id, 10::INTEGER AS amount;


 SELECT order_id,
  	order_time,
    customer_id,
    pizza_id,
    CASE WHEN extras IN ('null', '') THEN NULL  
    ELSE extras END AS extras
  FROM pizza_runner.customer_orders;
```
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
SELECT SETSEED(1);

DROP TABLE IF EXISTS pizza_runner.ratings;
CREATE TABLE pizza_runner.ratings (
  "order_id" INTEGER,
  "rating" INTEGER
);

INSERT INTO pizza_runner.ratings
SELECT
  order_id,
  FLOOR(1 + 5 * RANDOM()) AS rating
FROM pizza_runner.runner_orders
WHERE pickup_time IS NOT NULL;
```

4. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
with temp_amt as(SELECT c.order_id, 
                 c.customer_id, 
                 c.pizza_id, 
                 c.exclusions, 
                 c.extras, 
                 c.order_time, 
                 TRIM(SPLIT_PART(o.distance,'km',1))::FLOAT as new_distance,
CASE WHEN c.pizza_id = 1 then 12 else 10 end as amount
FROM pizza_runner.customer_orders c join pizza_runner.runner_orders o
on c.order_id=o.order_id
where o.distance !='null')
select sum(amount - new_distance * 0.3) as revenue 
from temp_amt;

```
***
