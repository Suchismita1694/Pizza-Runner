# Pizza-Runner
Danny Ma's SQL Challenge #2

```
DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  "runner_id" INTEGER,
  "registration_date" DATE
);
INSERT INTO runners
  ("runner_id", "registration_date")
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');


DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  "order_id" INTEGER,
  "customer_id" INTEGER,
  "pizza_id" INTEGER,
  "exclusions" VARCHAR(4),
  "extras" VARCHAR(4),
  "order_time" DATEtime
);

INSERT INTO customer_orders
  ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');


DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "pickup_time" VARCHAR(19),
  "distance" VARCHAR(7),
  "duration" VARCHAR(10),
  "cancellation" VARCHAR(23)
);

INSERT INTO runner_orders
  ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');


DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" TEXT
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');


DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  "pizza_id" INTEGER,
  "toppings" TEXT
);
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');


DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  "topping_id" INTEGER,
  "topping_name" TEXT
);
INSERT INTO pizza_toppings
  ("topping_id", "topping_name")
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```


-- PIZZA MATRICS

--1.How many pizzas were ordered?

```
select Count(*) as pizza_ordered from customer_orders;
```

--2.How many unique customer orders were made?

```
select count(distinct order_id)
from customer_orders
```

-- updating the coloum with null

```
update runner_orders
set pickup_time = NULL
where pickup_time = 'null'

update runner_orders
set distance = NULL
where distance = 'null'

update runner_orders
set duration = NULL
where duration = 'null'

update runner_orders
set cancellation = NULL
where cancellation = 'null'
```

```
select * from runner_orders;
```

--3.How many successful orders were delivered by each runner?

```
select count(pickup_time) as orders_delivered,runner_id from runner_orders
group by runner_id;
```

--4. How many of each type of pizza was delivered?

```
select count(co.pizza_id) as pizza_ordered, cast(pizza_name as nvarchar(100)) -- cast is used to change the datatype from text to nvarchar
from customer_orders co
join pizza_names pn
on pn.pizza_id = co.pizza_id
join runner_orders r
on r.order_id= co.order_id
where r.pickup_time is not null
group by cast(pizza_name as nvarchar(100))
```

--5.How many Vegetarian and Meatlovers were ordered by each customer?

```
select count(co.pizza_id) as pizza_ordered, cast(pizza_name as nvarchar(100)), co.customer_id
from customer_orders co
join pizza_names pn
on pn.pizza_id = co.pizza_id
join runner_orders r
on r.order_id= co.order_id
where r.pickup_time is not null
group by cast(pizza_name as nvarchar(100)),co.customer_id
order by co.customer_id
```

--6.What was the maximum number of pizzas delivered in a single order?

```
with cte as 
(select Count(pizza_id) as pizza_count,ro.order_id 
from runner_orders ro
join customer_orders co
on ro.order_id = co.order_id
group by ro.order_id)
select max(cte.pizza_count) as max_pizza_delivered
from cte
```

--7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```
select *
from customer_orders

update customer_orders
set exclusions = ' ' 
where exclusions is null


update customer_orders
set extras = ' ' 
where extras is null

with cte as
(select *,
(case when exclusions <> ' ' or extras <> ' ' then 1 
else 0 end) as  with_changes,
(case when exclusions = ' ' and extras = ' ' then 1 
else 0 end) as without_change
from customer_orders)
select customer_id,sum(with_changes) as with_changes, sum(without_change) as no_change
from cte
join runner_orders r
on r.order_id = cte.order_id
where pickup_time is not null
group by customer_id
```

--8.How many pizzas were delivered that had both exclusions and extras?

```
with cte as 
(select *,
case when exclusions <> ' ' and extras <> ' ' then 1
else 0 end as points
from customer_orders)
select sum(points) as pizza_with_extra_n_exclusion
from cte 
join runner_orders r
on r.order_id = cte.order_id
where pickup_time is not null
```

--9.What was the total volume of pizzas ordered for each hour of the day?

```
select count(order_id) as pizza_count,
datepart (Hour, [order_time]) as hr_day
from customer_orders
group by datepart (Hour, [order_time])
```

--10.What was the volume of orders for each day of the week?

```
select count(order_id) as pizza_count,
datename (WEEKDAY, [order_time])
from customer_orders
group by datename (WEEKDAY, [order_time])
order by pizza_count
```

--B. Runner and Customer Experience
-- Cleaning two columns distance and duration from runner_orders table

```
update runner_orders
set duration = case WHEN duration LIKE '%mins' THEN TRIM('mins' from duration) 
    WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)        
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)       
    ELSE duration END

alter table runner_orders
alter COLUMN duration float;

update runner_orders
set distance = case WHEN distance LIKE '%km' THEN TRIM('km' from distance)        
    ELSE distance END

alter table runner_orders
alter column distance float
```

--1.How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```
select count(*), 
DATEPART(week, [registration_date])
from runners
group by DATEPART(week, [registration_date]);
```

--2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```
with cte as
(select r.order_id, pickup_time, c.order_time,
DATEDIFF(minute,c.order_time,r.pickup_time) as timediff
from runner_orders r
join customer_orders c
on r.order_id=c.order_id
where pickup_time is not null
group by r.order_id, pickup_time, c.order_time)
select AVG(timediff), runner_id
from cte
join runner_orders r
on r.order_id = cte.order_id
group by runner_id;
```

--3.Is there any relationship between the number of pizzas and how long the order takes to prepare?

```
with cte as
(select count(r.order_id) as p_cnt, r.order_id,
DATEDIFF(minute,c.order_time,r.pickup_time)as timediff
from runner_orders r
join customer_orders c
on r.order_id=c.order_id
where pickup_time is not null
group by DATEDIFF(minute,c.order_time,r.pickup_time), r.order_id)
select p_cnt, AVG(timediff)
from cte
group by p_cnt;
```

--4.What was the average distance travelled for each customer?

```
select c.customer_id, AVG (distance)
from runner_orders r
join customer_orders c
on r.order_id = c.order_id
group by c.customer_id
```

--5.What was the difference between the longest and shortest delivery times for all orders?

```
select Max(duration) - Min(duration)
from runner_orders
```

--6.What was the average speed for each runner for each delivery and do you notice any trend for these values?

```
with cte as 
(select Round((distance * 60/duration),2) as speed, runner_id,order_id
from runner_orders)
select AVG(speed) as avg_speed, runner_id
from cte 
group by runner_id
```

--7.What is the successful delivery percentage for each runner?

```
with cte as
(select runner_id,
        count(order_id) as total_order,
        Sum(case when distance is null then 0 else 1 end) as total_delivered
  from runner_orders
 group by runner_id)

select runner_id, Round(((total_delivered*0.1)/(total_order*0.1))*100,2) as successfull_delivery_percentage
from cte
```


-- C. Ingredient Optimisation

--1.What are the standard ingredients for each pizza?

 Work in Progress
