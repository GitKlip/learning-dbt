# learning-dbt


starting with this course: https://courses.getdbt.com/courses/fundamentals

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
