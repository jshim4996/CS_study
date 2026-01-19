# Chapter 05. Sorting and Aggregation (정렬과 집계)

> Learn to organize query results with sorting and summarize data using aggregate functions, grouping, and filtering.

---

## ORDER BY - Sorting Results (결과 정렬)

### Concept (개념)
The ORDER BY clause sorts the result set based on one or more columns. Without ORDER BY, the order of rows in the result is undefined and may vary between executions.

### Basic Syntax
```sql
SELECT columns
FROM table_name
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...;
```

### Sort Directions
- **ASC (Ascending, 오름차순)**: Smallest to largest, A to Z (default)
- **DESC (Descending, 내림차순)**: Largest to smallest, Z to A

### Example
```sql
-- Sample data setup
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock_quantity INT,
    created_at DATE
);

INSERT INTO products VALUES
(1, 'Laptop Pro', 'Electronics', 1299.99, 50, '2023-01-15'),
(2, 'Wireless Mouse', 'Electronics', 29.99, 200, '2023-02-20'),
(3, 'Office Chair', 'Furniture', 249.99, 75, '2023-01-10'),
(4, 'USB Cable', 'Electronics', 9.99, 500, '2023-03-05'),
(5, 'Standing Desk', 'Furniture', 599.99, 30, '2023-02-28'),
(6, 'Monitor 27"', 'Electronics', 399.99, 100, '2023-01-25'),
(7, 'Keyboard', 'Electronics', 79.99, 150, '2023-03-10');

-- Sort by single column (ascending - default)
SELECT product_name, price
FROM products
ORDER BY price;

-- Sort by single column (descending)
SELECT product_name, price
FROM products
ORDER BY price DESC;

-- Sort by multiple columns
SELECT product_name, category, price
FROM products
ORDER BY category ASC, price DESC;

-- Sort by column position (not recommended for readability)
SELECT product_name, category, price
FROM products
ORDER BY 2, 3 DESC;  -- 2=category, 3=price

-- Sort by alias
SELECT product_name, price * 1.1 AS price_with_tax
FROM products
ORDER BY price_with_tax DESC;

-- Sort by expression
SELECT product_name, price, stock_quantity, price * stock_quantity AS inventory_value
FROM products
ORDER BY price * stock_quantity DESC;
```

### NULL Handling in ORDER BY

```sql
-- NULL values handling varies by database
-- Most databases: NULLs first in ASC, last in DESC
-- PostgreSQL: Can specify NULLS FIRST or NULLS LAST
SELECT product_name, category
FROM products
ORDER BY category NULLS LAST;

-- MySQL: Use COALESCE or IFNULL
SELECT product_name, category
FROM products
ORDER BY COALESCE(category, 'ZZZZZ');  -- NULLs sorted last
```

---

## LIMIT and OFFSET (결과 제한)

### Concept (개념)
LIMIT restricts the number of rows returned. OFFSET skips a specified number of rows before returning results. Together they enable pagination.

### Syntax Variations

| Database | Syntax |
|----------|--------|
| MySQL, PostgreSQL | LIMIT n OFFSET m |
| SQL Server | OFFSET m ROWS FETCH NEXT n ROWS ONLY |
| Oracle | ROWNUM <= n (legacy) or FETCH FIRST n ROWS ONLY |

### Example
```sql
-- Get top 3 most expensive products
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 3;

-- Skip first 2, get next 3 (pagination)
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 3 OFFSET 2;

-- Alternative syntax for LIMIT with OFFSET
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 2, 3;  -- LIMIT offset, count (MySQL)

-- Pagination example (page 2, 5 items per page)
-- Page 1: OFFSET 0
-- Page 2: OFFSET 5
-- Page 3: OFFSET 10
SELECT product_name, price
FROM products
ORDER BY product_id
LIMIT 5 OFFSET 5;  -- Page 2

-- SQL Server syntax
SELECT product_name, price
FROM products
ORDER BY price DESC
OFFSET 2 ROWS FETCH NEXT 3 ROWS ONLY;
```

---

## DISTINCT - Removing Duplicates (중복 제거)

### Concept (개념)
DISTINCT eliminates duplicate rows from the result set. It considers all selected columns when determining uniqueness.

### Example
```sql
-- Get unique categories
SELECT DISTINCT category
FROM products;

-- DISTINCT with multiple columns (unique combinations)
SELECT DISTINCT category, FLOOR(price / 100) AS price_range
FROM products;

-- Count distinct values
SELECT COUNT(DISTINCT category) AS category_count
FROM products;

-- ALL is the opposite of DISTINCT (default behavior)
SELECT ALL category FROM products;  -- Includes duplicates
SELECT category FROM products;      -- Same as above
```

---

## Aggregate Functions (집계 함수)

### Concept (개념)
Aggregate functions perform calculations on sets of values and return a single result. They are essential for data analysis and reporting.

### Common Aggregate Functions

| Function | Description | Korean |
|----------|-------------|--------|
| COUNT() | Count rows or non-NULL values | 개수 |
| SUM() | Sum of values | 합계 |
| AVG() | Average of values | 평균 |
| MAX() | Maximum value | 최댓값 |
| MIN() | Minimum value | 최솟값 |

### Example
```sql
-- COUNT variations
SELECT COUNT(*) AS total_products FROM products;           -- Count all rows
SELECT COUNT(category) AS products_with_category FROM products;  -- Count non-NULL
SELECT COUNT(DISTINCT category) AS unique_categories FROM products;  -- Count unique

-- SUM: Total and calculations
SELECT SUM(stock_quantity) AS total_stock FROM products;
SELECT SUM(price * stock_quantity) AS total_inventory_value FROM products;

-- AVG: Averages
SELECT AVG(price) AS average_price FROM products;
SELECT AVG(price)::NUMERIC(10,2) AS avg_price_rounded FROM products;  -- PostgreSQL

-- MAX and MIN
SELECT MAX(price) AS highest_price, MIN(price) AS lowest_price FROM products;
SELECT MAX(created_at) AS newest_product, MIN(created_at) AS oldest_product FROM products;

-- Combining multiple aggregates
SELECT
    COUNT(*) AS product_count,
    SUM(stock_quantity) AS total_stock,
    AVG(price) AS avg_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price,
    MAX(price) - MIN(price) AS price_range
FROM products;

-- Aggregate with WHERE
SELECT
    COUNT(*) AS electronics_count,
    AVG(price) AS avg_electronics_price
FROM products
WHERE category = 'Electronics';
```

### NULL Handling in Aggregates

```sql
-- Aggregate functions ignore NULL values (except COUNT(*))
-- If a column has values: 10, 20, NULL, 30
-- AVG = (10 + 20 + 30) / 3 = 20 (not / 4)
-- COUNT(column) = 3 (not 4)
-- COUNT(*) = 4 (counts all rows)

-- Example with NULLs
CREATE TABLE test_nulls (value INT);
INSERT INTO test_nulls VALUES (10), (20), (NULL), (30);

SELECT
    COUNT(*) AS count_all,        -- 4
    COUNT(value) AS count_value,  -- 3
    SUM(value) AS sum_value,      -- 60
    AVG(value) AS avg_value       -- 20 (60/3, not 60/4)
FROM test_nulls;
```

---

## GROUP BY - Grouping Data (데이터 그룹화)

### Concept (개념)
GROUP BY divides rows into groups based on column values. Aggregate functions then operate on each group separately, returning one row per group.

### Basic Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

### Important Rule
When using GROUP BY, every column in SELECT must either:
1. Be in the GROUP BY clause, OR
2. Be used in an aggregate function

### Example
```sql
-- Count products per category
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category;

-- Multiple aggregates per group
SELECT
    category,
    COUNT(*) AS product_count,
    SUM(stock_quantity) AS total_stock,
    AVG(price) AS avg_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price
FROM products
GROUP BY category;

-- Group by multiple columns
SELECT
    category,
    EXTRACT(MONTH FROM created_at) AS month,
    COUNT(*) AS products_added
FROM products
GROUP BY category, EXTRACT(MONTH FROM created_at)
ORDER BY category, month;

-- Group by with expression
SELECT
    CASE
        WHEN price < 50 THEN 'Budget'
        WHEN price < 200 THEN 'Mid-range'
        ELSE 'Premium'
    END AS price_tier,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products
GROUP BY
    CASE
        WHEN price < 50 THEN 'Budget'
        WHEN price < 200 THEN 'Mid-range'
        ELSE 'Premium'
    END;

-- Practical example: Sales summary
CREATE TABLE sales (
    sale_id INT,
    product_id INT,
    sale_date DATE,
    quantity INT,
    unit_price DECIMAL(10,2)
);

INSERT INTO sales VALUES
(1, 1, '2023-01-15', 2, 1299.99),
(2, 2, '2023-01-15', 5, 29.99),
(3, 3, '2023-01-16', 1, 249.99),
(4, 1, '2023-01-17', 1, 1299.99),
(5, 2, '2023-01-17', 3, 29.99);

-- Daily sales summary
SELECT
    sale_date,
    COUNT(*) AS num_transactions,
    SUM(quantity) AS total_items_sold,
    SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY sale_date
ORDER BY sale_date;
```

---

## HAVING - Filtering Groups (그룹 필터링)

### Concept (개념)
HAVING filters groups after GROUP BY is applied. While WHERE filters individual rows before grouping, HAVING filters groups based on aggregate values.

### WHERE vs HAVING

| Clause | When Applied | Filters By | Korean |
|--------|--------------|------------|--------|
| WHERE | Before GROUP BY | Row values | 행 필터링 |
| HAVING | After GROUP BY | Group/aggregate values | 그룹 필터링 |

### Basic Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE row_condition
GROUP BY column1
HAVING group_condition;
```

### Example
```sql
-- Find categories with more than 2 products
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 2;

-- Categories with average price over $100
SELECT category, AVG(price) AS avg_price
FROM products
GROUP BY category
HAVING AVG(price) > 100;

-- Combining WHERE and HAVING
-- WHERE: Filter products first (only items with stock)
-- HAVING: Then filter groups (only categories with high average price)
SELECT
    category,
    COUNT(*) AS in_stock_products,
    AVG(price) AS avg_price
FROM products
WHERE stock_quantity > 0      -- Row filter: only products in stock
GROUP BY category
HAVING AVG(price) > 50        -- Group filter: only expensive categories
ORDER BY avg_price DESC;

-- Multiple HAVING conditions
SELECT
    category,
    COUNT(*) AS product_count,
    SUM(stock_quantity) AS total_stock,
    AVG(price) AS avg_price
FROM products
GROUP BY category
HAVING COUNT(*) >= 2
   AND AVG(price) < 500
   AND SUM(stock_quantity) > 100;

-- Practical example: Find high-value customers
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)
);

INSERT INTO orders VALUES
(1, 101, '2023-01-10', 150.00),
(2, 102, '2023-01-12', 75.00),
(3, 101, '2023-01-15', 200.00),
(4, 103, '2023-01-18', 500.00),
(5, 101, '2023-01-20', 100.00),
(6, 102, '2023-01-22', 250.00);

-- Customers who have spent more than $300 total
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(total_amount) > 300
ORDER BY total_spent DESC;
```

---

## Query Execution Order (쿼리 실행 순서)

### Concept (개념)
Understanding the logical execution order helps write correct queries:

1. **FROM** - Identify source tables
2. **WHERE** - Filter individual rows
3. **GROUP BY** - Create groups
4. **HAVING** - Filter groups
5. **SELECT** - Choose columns and calculate expressions
6. **DISTINCT** - Remove duplicates
7. **ORDER BY** - Sort results
8. **LIMIT/OFFSET** - Limit results

### Example
```sql
-- This query follows the execution order
SELECT                                    -- 5
    category,                             -- (select columns)
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products                             -- 1 (from table)
WHERE stock_quantity > 0                  -- 2 (filter rows)
GROUP BY category                         -- 3 (group rows)
HAVING COUNT(*) >= 2                      -- 4 (filter groups)
ORDER BY avg_price DESC                   -- 6 (sort)
LIMIT 5;                                  -- 7 (limit results)

-- Because of execution order:
-- - Cannot use SELECT alias in WHERE
-- - Can use SELECT alias in ORDER BY (some databases)
-- - HAVING comes after GROUP BY, so can use aggregates
```

---

## Comprehensive Example

```sql
-- Complete example: Sales analysis report
CREATE TABLE sales_data (
    sale_id INT PRIMARY KEY,
    product_category VARCHAR(50),
    region VARCHAR(50),
    sale_date DATE,
    quantity INT,
    revenue DECIMAL(12,2)
);

INSERT INTO sales_data VALUES
(1, 'Electronics', 'North', '2023-01-15', 10, 5000.00),
(2, 'Clothing', 'South', '2023-01-16', 25, 1250.00),
(3, 'Electronics', 'North', '2023-01-17', 5, 2500.00),
(4, 'Furniture', 'East', '2023-01-18', 2, 1500.00),
(5, 'Clothing', 'North', '2023-01-19', 30, 1500.00),
(6, 'Electronics', 'South', '2023-01-20', 8, 4000.00),
(7, 'Furniture', 'North', '2023-01-21', 3, 2250.00),
(8, 'Clothing', 'East', '2023-01-22', 20, 1000.00);

-- Comprehensive analysis query
SELECT
    product_category,
    region,
    COUNT(*) AS num_sales,
    SUM(quantity) AS total_quantity,
    SUM(revenue) AS total_revenue,
    AVG(revenue) AS avg_revenue,
    MAX(revenue) AS max_sale,
    MIN(revenue) AS min_sale
FROM sales_data
WHERE sale_date BETWEEN '2023-01-01' AND '2023-01-31'
GROUP BY product_category, region
HAVING SUM(revenue) > 1000
ORDER BY total_revenue DESC
LIMIT 10;
```

---

## Summary (요약)

| Clause | Korean | Purpose |
|--------|--------|---------|
| ORDER BY | 정렬 | Sort result rows |
| ASC/DESC | 오름차순/내림차순 | Sort direction |
| LIMIT | 결과 제한 | Limit number of rows |
| OFFSET | 건너뛰기 | Skip rows for pagination |
| DISTINCT | 중복 제거 | Remove duplicate rows |
| COUNT | 개수 | Count rows/values |
| SUM | 합계 | Sum of values |
| AVG | 평균 | Average of values |
| MAX/MIN | 최대/최소 | Maximum/minimum value |
| GROUP BY | 그룹화 | Group rows for aggregation |
| HAVING | 그룹 필터 | Filter grouped results |

---
