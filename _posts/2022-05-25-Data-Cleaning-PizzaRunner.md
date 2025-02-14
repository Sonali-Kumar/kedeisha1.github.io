---
layout: post
title: "Case Study #2 - Pizza Runner"
image: "/posts/PizzaRunner/Pizza_Runner_Cover.png"
tags: [SQL, Danny Ma, Pizza Runner, 8 Week Challenge]
---


## Learning Objective 

The following post covers cleaning of the Pizza Runner dataset. I used *PostgreSQL* to tackle this case study, mainly focusing on following functions:

* Alter Table
* Alter Columns
* Rename Columns
* Change Column Type
* Timestamp
* Wildcards
* Substring 


## Introduction

Did you know that over **115 million kilograms** of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway...)

Danny was scrolling through his Instagram feed when something really caught his eye - "80s Retro Styling and Pizza Is The Future!"

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to *Uberize* it - and so Pizza Runner was launched!

Danny started by recruiting "runners" to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny's house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Available Data

All datasets exist within the `pizza_runner` database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.


### Table 1: runners

The `runners` table shows the `registration_date` for each new runner

| runner_id | registration_date |
| --------- | ----------------- |
|         1 | 2021-01-01        |
|         2 | 2021-01-03        |
|         3 | 2021-01-08        |
|         4 | 2021-01-15        |

### Table 2: customer_orders

Customer pizza orders are captured in the `customer_orders` table with 1 row for each individual pizza that is part of the order.

The `pizza_id` relates to the type of pizza which was ordered whilst the `exclusions` are the `ingredient_id` values which should be removed from the pizza and the `extras` are the `ingredient_id` values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying `exclusions` and `extras` values even if the pizza is the same type!

The `exclusions` and `extras` columns will need to be cleaned up before using them in your queries.

<div class="responsive-table" markdown="block">

| order_id | customer_id | pizza_id | exclusions | extras |     order_time      |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
|        1 |         101 |        1 |            |        | 2021-01-01 18:05:02 |
|        2 |         101 |        1 |            |        | 2021-01-01 19:00:52 |
|        3 |         102 |        1 |            |        | 2021-01-02 23:51:23 |
|        3 |         102 |        2 |            | NaN    | 2021-01-02 23:51:23 |
|        4 |         103 |        1 | 4          |        | 2021-01-04 13:23:46 |
|        4 |         103 |        1 | 4          |        | 2021-01-04 13:23:46 |
|        4 |         103 |        2 | 4          |        | 2021-01-04 13:23:46 |
|        5 |         104 |        1 | null       | 1      | 2021-01-08 21:00:29 |
|        6 |         101 |        2 | null       | null   | 2021-01-08 21:03:13 |
|        7 |         105 |        2 | null       | 1      | 2021-01-08 21:20:29 |
|        8 |         102 |        1 | null       | null   | 2021-01-09 23:54:33 |
|        9 |         103 |        1 | 4          | 1, 5   | 2021-01-10 11:22:59 |
|       10 |         104 |        1 | null       | null   | 2021-01-11 18:34:49 |
|       10 |         104 |        1 | 2, 6       | 1, 4   | 2021-01-11 18:34:49 |

</div>

### Table 3: runner_orders

After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The `pickup_time` is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The `distance` and `duration` fields are related to how far and long the runner had to travel to deliver the order to the respective customer.

There are some known data issues with this table so be careful when using this in your queries - make sure to check the data types for each column in the schema SQL!

<div class="responsive-table" markdown="block">

| order_id | runner_id |     pickup_time     | distance |  duration  |      cancellation       |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
|        1 |         1 | 2021-01-01 18:15:34 | 20km     | 32 minutes |                         |
|        2 |         1 | 2021-01-01 19:10:54 | 20km     | 27 minutes |                         |
|        3 |         1 | 2021-01-03 00:12:37 | 13.4km   | 20 mins    | NaN                     |
|        4 |         2 | 2021-01-04 13:53:03 | 23.4     | 40         | NaN                     |
|        5 |         3 | 2021-01-08 21:10:57 | 10       | 15         | NaN                     |
|        6 |         3 | null                | null     | null       | Restaurant Cancellation |
|        7 |         2 | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
|        8 |         2 | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
|        9 |         2 | null                | null     | null       | Customer Cancellation   |
|       10 |         1 | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

</div>

### Table 4: pizza_names

At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!

<div class="responsive-table" markdown="block">

| pizza_id | pizza_name  |
| -------- | ----------- |
|        1 | Meat Lovers |
|        2 | Vegetarian  |

</div>

### Table 5: pizza_recipes

Each `pizza_id` has a standard set of `toppings` which are used as part of the pizza recipe.

<div class="responsive-table" markdown="block">

| pizza_id |        toppings         |
| -------- | ----------------------- |
|        1 | 1, 2, 3, 4, 5, 6, 8, 10 |
|        2 | 4, 6, 7, 9, 11, 12      |

</div>

### Table 6: pizza_toppings

This table contains all of the `topping_name` values with their corresponding `topping_id` value

<div class="responsive-table" markdown="block">

| topping_id | topping_name |
| ---------- | ------------ |
|          1 | Bacon        |
|          2 | BBQ Sauce    |
|          3 | Beef         |
|          4 | Cheese       |
|          5 | Chicken      |
|          6 | Mushrooms    |
|          7 | Onions       |
|          8 | Pepperoni    |
|          9 | Peppers      |
|         10 | Salami       |
|         11 | Tomatoes     |
|         12 | Tomato Sauce |

</div>


## Case Study Questions

This case study has **LOTS** of questions - they are broken up by area of focus including:
* Pizza Metrics
* Runner and Customer Experience
* Ingredient Optimisation
* Pricing and Ratings
* Bonus DML Challenges (DML = Data Manipulation Language)


**Before you start writing your SQL queries however - you might want to investigate the data, you may want to do something with some of those `null` values and data types in the `customer_orders` and `runner_orders` tables!**


## Let's Get Started!!!

---

#### RENAMING COLUMNS

I am starting with renaming some columns in the Runner_Orders table. If you are wondering WHY, I want to remove the units (min, km, etc.) from the column and change the column type from *VARCHAR* to either *INTEGER* or *DECIMAL*. This will help later in the challenge when we are instructed to do calculations using these columns.

```ruby
ALTER TABLE PIZZA_RUNNER.runner_ORDERS
RENAME COLUMN distance TO distance_KM
```
```ruby
ALTER TABLE PIZZA_RUNNER.runner_ORDERS
RENAME COLUMN duration TO duration_mins 
```
--- 
#### CHANGING COLUMN VALUES

After changing the column name, I am removing text/string from the *duration_mins* column and *distance_km* column, using the substring function.

```ruby
UPDATE PIZZA_RUNNER.runner_ORDERS
SET duration_mins = SUBSTRING(duration_mins,0,3)
WHERE duration_mins like '%mi%'
```

```ruby
UPDATE PIZZA_RUNNER.runner_ORDERS
SET distance_km = SUBSTRING(distance_km,0,3)
WHERE distance_km like '%km%'
```
---
Below, I am updating the 'null' and/or BLANK values to **NULL** for consistency purposes. 

```ruby
UPDATE PIZZA_RUNNER.CUSTOMER_ORDERS
SET EXCLUSIONS = NULL 
WHERE exclusions = 'null' or exclusions = ''
```

```ruby

UPDATE PIZZA_RUNNER.CUSTOMER_ORDERS
SET extras = NULL 
WHERE extras = 'null' or extras = ''
```

```ruby

UPDATE PIZZA_RUNNER.runner_ORDERS
SET distance_km = NULL 
WHERE distance_km = 'null' or distance_km = ''
```

```ruby

UPDATE PIZZA_RUNNER.runner_ORDERS
SET duration_mins = NULL 
WHERE duration_mins = 'null' or duration_mins = ''
```

---
#### CHANGING COLUMN TYPE 

##### Changing the column type to *INTEGER* or *DECIMAL*

**NOTE**: *In order to change the column type to *INTEGER* or *DECIMAL*, make sure that the column data only contains numbers **NOT** strings.*

```ruby
ALTER TABLE pizza_runner.runner_orders
ALTER distance_km TYPE DECIMAL USING distance_km::DECIMAL
```
```ruby
ALTER TABLE pizza_runner.runner_orders
ALTER duration_mins TYPE INTEGER USING duration_mins::INTEGER
```

##### Changing the column type to *TIMESTAMP*

**NOTE:** *In order to change the column type to *TIMESTAMP*, data **CANNOT** be BLANK or strings in that given column.*

```ruby
UPDATE pizza_runner.runner_orders
SET pickup_time = NULL   
WHERE pickup_time = '' OR pickup_time = 'null'
```
```ruby
ALTER TABLE pizza_runner.runner_orders
ALTER pickup_time TYPE TIMESTAMP USING pickup_time::TIMESTAMP
```
