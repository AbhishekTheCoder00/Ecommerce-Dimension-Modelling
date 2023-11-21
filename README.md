# Ecommerce-Dimension-Modelling
Ecommerce Dimension Modelling using snowflake and lucid.app

## Create My_stage
CREATE STAGE my_stage
URL = "S3_PATH"
CREDENTIALS = (AWS_KEY_ID = 'YOUR_KEY_ID_OF_AWS'
AWS_SECRET_KEY = 'YOUR_SECRET_KEY_OF_AWS');

## File Format
CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"';

## Create Table and Load Data using mystage+file format
CREATE TABLE aisles (
        aisle_id INTEGER PRIMARY KEY,
        aisle VARCHAR

    );

COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

SELECT * FROM aisles limit 10;

CREATE TABLE departments (
   department_id INTEGER PRIMARY KEY,
   department VARCHAR
);

COPY INTO departments (department_id, department)
FROM @my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

SELECT * FROM departments;

CREATE OR REPLACE TABLE products(
      product_id INTEGER PRIMARY KEY,
      product_name VARCHAR,
      aisle_id INTEGER,
      department_id INTEGER);

COPY INTO products (product_id, product_name, aisle_id, department_id)
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

CREATE OR REPLACE TABLE orders (
        order_id INTEGER PRIMARY KEY,
        user_id INTEGER,
        eval_set STRING,
        order_number INTEGER,
        order_dow INTEGER,
        order_hour_of_the_day INTEGER,
        days_since_prior_order INTEGER
  );

COPY INTO orders (order_id, user_id, eval_set, order_number, order_dow, order_hour_of_the_day, days_since_prior_order)
FROM @my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

SELECT * FROM orders limit 19;

CREATE OR REPLACE TABLE order_products (
         order_id INTEGER,
         product_id INTEGER,
         add_to_cart_order INTEGER,
         reordered INTEGER,
         PRIMARY KEY (order_id, product_id)
);

COPY INTO order_products (order_id, product_id, add_to_cart_order, reordered)
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

## Dimension Modeling using lucid.app
![Alt text](D:\ecom snowflake DW Fact Dimention Diagram.png)

## Dimension TABLES
CREATE OR REPLACE TABLE dim_user AS(
     SELECT user_id FROM orders
);

CREATE OR REPLACE TABLE dim_products AS (
    SELECT product_id, product_name FROM products
);

CREATE OR REPLACE TABLE dim_aisles AS(
    SELECT aisle_id, aisle FROM aisles
);

CREATE OR REPLACE TABLE dim_orders AS(
    SELECT
      order_id, 
      order_number,
      order_dow, 
      order_hour_of_the_day, 
      days_since_prior_order
    FROM
      orders 
);

CREATE OR REPLACE TABLE dim_department AS (
    SELECT department_id, department FROM departments
);

## Create Fact_table
CREATE OR REPLACE TABLE fact_order_products AS(
    SELECT 
      op.product_id,
      op.order_id,
      o.user_id,
      p.aisle_id,
      p.department_id,
      op.add_to_cart_order,
      op.reordered
    FROM 
      order_products op
    JOIN
      orders o ON op.order_id = o.order_id
    JOIN
      products p ON op.product_id = p.product_id
      
);

## Analytical Query 
-- Query to calculate the total number of products ordered per department:
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_departments d ON fop.department_id = d.department_id
GROUP BY
  d.department;

-- Query to find the top 5 aisles with the highest number of reordered products:
SELECT
  a.aisle,
  COUNT(*) AS total_reordered
FROM
  fact_order_products fop
JOIN
  dim_aisles a ON fop.aisle_id = a.aisle_id
WHERE
  fop.reordered = TRUE
GROUP BY
  a.aisle
ORDER BY
  total_reordered DESC
LIMIT 5;

-- Query to calculate the average number of products added to the cart per order by day of the week:
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;

-- Query to identify the top 10 users with the highest number of unique products ordered:
SELECT
  u.user_id,
  COUNT(DISTINCT fop.product_id) AS unique_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_users u ON fop.user_id = u.user_id
GROUP BY
  u.user_id
ORDER BY
  unique_products_ordered DESC
LIMIT 10;

