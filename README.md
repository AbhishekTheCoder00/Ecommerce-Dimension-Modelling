# Ecommerce Dimension Modelling

Ecommerce Dimension Modelling using Snowflake and Lucid.app.

## Dimension Modeling using lucid.app
![Ecommerce Snowflake DW Fact Dimension Diagram](D:\ecom snowflake DW Fact Dimention Diagram.png)

## Create My_stage

```sql
CREATE STAGE my_stage
URL = "S3_PATH"
CREDENTIALS = (
  AWS_KEY_ID = 'YOUR_KEY_ID_OF_AWS',
  AWS_SECRET_KEY = 'YOUR_SECRET_KEY_OF_AWS'
);

## File Format
CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"';

## Create Table and Load Data using mystage+file format
-- Example for 'aisles' table
CREATE TABLE aisles (
  aisle_id INTEGER PRIMARY KEY,
  aisle VARCHAR
);

COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

-- Similar blocks for 'departments', 'products', 'orders', and 'order_products' tables


-- Example for 'dim_user' table
CREATE OR REPLACE TABLE dim_user AS (
  SELECT user_id FROM orders
);

-- Similar blocks for 'dim_products', 'dim_aisles', 'dim_orders', and 'dim_department' tables

## Create Fact_table
CREATE OR REPLACE TABLE fact_order_products AS (
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
-- Query to calculate the total number of products ordered per department
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_department d ON fop.department_id = d.department_id
GROUP BY
  d.department;

-- Similar blocks for other analytical queries


