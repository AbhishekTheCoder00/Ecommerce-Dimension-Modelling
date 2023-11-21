# Ecommerce Dimension Modelling

Ecommerce Dimension Modelling using Snowflake and Lucid.app.

## DataSet Link
[Instacart Market Basket Analysis Datasets](https://www.kaggle.com/competitions/instacart-market-basket-analysis/rules)


## Dimension Modeling using lucid.app
![Ecommerce Snowflake DW Fact Dimension Diagram](https://github.com/AbhishekTheCoder00/Ecommerce-Dimension-Modelling/blob/main/ecom%20snowflake%20DW%20Fact%20Dimention%20Diagram.png)

# Ecommerce Dimension Modelling

## Create Stage
```markdown
```sql
CREATE STAGE my_stage
URL = "S3_PATH"
CREDENTIALS = (
  AWS_KEY_ID = 'YOUR_KEY_ID_OF_AWS',
  AWS_SECRET_KEY = 'YOUR_SECRET_KEY_OF_AWS'
);
```

## File Format

```sql
CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"';
```

## Create Table and Load Data

### Example for 'aisles' table

```sql
CREATE TABLE aisles (
  aisle_id INTEGER PRIMARY KEY,
  aisle VARCHAR
);

COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

#### Similar blocks for 'departments', 'products', 'orders', and 'order_products' tables

## Create Dimension Tables

### Example for 'dim_user' table

```sql
CREATE OR REPLACE TABLE dim_user AS (
  SELECT user_id FROM orders
);
```

#### Similar blocks for 'dim_products', 'dim_aisles', 'dim_orders', and 'dim_department' tables

## Create Fact Table

```sql
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
```

## Analytical Query

#### Query to calculate the total number of products ordered per department

```sql
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_department d ON fop.department_id = d.department_id
GROUP BY
  d.department;
```

#### Query to find the top 5 aisles with the highest number of reordered products:

```sql
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
```

#### Query to calculate the average number of products added to the cart per order by day of the week:

```sql
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;
```

#### Query to identify the top 10 users with the highest number of unique products ordered:

```sql
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
```



