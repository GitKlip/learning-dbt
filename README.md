# learning-dbt


starting with this course: https://courses.getdbt.com/courses/fundamentals

## Data Setup

I'm starting using snowflake as the backend (temporary free trial) with the option to switch to bigquery (always free) later if needed
* Snowflake
  ```sql
  create warehouse transforming;
  create database raw;
  create database analytics;
  create schema raw.jaffle_shop;

  -- create this one directly in the schema
  create table raw.jaffle_shop.customers
  (
      id integer,
      first_name varchar,
      last_name varchar
  );

  copy into raw.jaffle_shop.customers (id, first_name, last_name)
      from 's3://dbt-tutorial-public/jaffle_shop_customers.csv'
          file_format = (
              type = 'CSV'
              field_delimiter = ','
              skip_header = 1
          )
  ;

  create table raw.jaffle_shop.orders
  (
    id integer,
    user_id integer,
    order_date date,
    status varchar,
    _etl_loaded_at timestamp default current_timestamp
  );

  copy into raw.jaffle_shop.orders (id, user_id, order_date, status)
      from 's3://dbt-tutorial-public/jaffle_shop_orders.csv'
          file_format = (
              type = 'CSV'
              field_delimiter = ','
              skip_header = 1
          )
  ;

  create schema raw.stripe;

  create table raw.stripe.payment (
    id integer,
    orderid integer,
    paymentmethod varchar,
    status varchar,
    amount integer,
    created date,
    _batched_at timestamp default current_timestamp
  );

  copy into raw.stripe.payment (id, orderid, paymentmethod, status, amount, created)
  from 's3://dbt-tutorial-public/stripe_payments.csv'
      file_format = (
          type = 'CSV'
          field_delimiter = ','
          skip_header = 1
      )
  ;
  ```
* BigQuery 
  ```sql
  select * from `dbt-tutorial.jaffle_shop.customers`;
  select * from `dbt-tutorial.jaffle_shop.orders`;
  select * from `dbt-tutorial.stripe.payment`;
  ```


## Raw Source data
```txt
table raw.jaffle_shop.customers (
    id integer,
    first_name varchar,
    last_name varchar
)
```
```txt
table raw.jaffle_shop.orders (
  id integer,
  user_id integer,
  order_date date,
  status varchar,
  _etl_loaded_at timestamp default current_timestamp
)
```
```txt
table raw.stripe.payment (
  id integer,
  orderid integer,
  paymentmethod varchar,
  status varchar,
  amount integer,
  created date,
  _batched_at timestamp default current_timestamp
)
```


## Modeling History
There have been multiple modeling paradigms since the advent of database technology. Many of these are classified as normalized modeling.
Normalized modeling techniques were designed when storage was expensive and compute was not as affordable as it is today.
With a modern cloud-based data warehouse, we can approach analytics differently in an agile or ad hoc modeling technique. This is often referred to as denormalized modeling.
dbt can build your data warehouse into any of these schemas. dbt is a tool for how to build these rather than enforcing what to build.

## Naming Conventions 
In working on this project, we established some conventions for naming our models.

* Sources (src) refer to the raw table data that have been built in the warehouse through a loading process. (We will cover configuring Sources in the Sources module)
* Staging (stg) refers to models that are built directly on top of sources. These have a one-to-one relationship with sources tables. These are used for very light transformations that shape the data into what you want it to be. These models are used to clean and standardize the data before transforming data downstream. Note: These are typically materialized as views.
* Intermediate (int) refers to any models that exist between final fact and dimension tables. These should be built on staging models rather than directly on sources to leverage the data cleaning that was done in staging.
* Fact (fct) refers to any data that represents something that occurred or is occurring. Examples include sessions, transactions, orders, stories, votes. These are typically skinny, long tables.
* Dimension (dim) refers to data that represents a person, place or thing. Examples include customers, products, candidates, buildings, employees.
Note: The Fact and Dimension convention is based on previous normalized modeling techniques.

## Folder Structure
* Marts folder: All intermediate, fact, and dimension models can be stored here. Further subfolders can be used to separate data by business function (e.g. marketing, finance)
* Staging folder: All staging models and source configurations can be stored here. Further subfolders can be used to separate data by data source (e.g. Stripe, Segment, Salesforce). (We will cover configuring Sources in the Sources module)
