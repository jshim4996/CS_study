# Chapter 19. Query Optimization (쿼리 최적화)

> Learn to analyze execution plans and optimize SQL queries for maximum performance.

---

## What is Query Optimization? (쿼리 최적화란?)

### Concept (개념)
Query optimization (쿼리 최적화) is the process of improving query performance by analyzing execution plans, restructuring queries, and ensuring proper index usage. The database's query optimizer automatically chooses execution strategies, but understanding how it works helps write better queries.

### Query Processing Flow

```
SQL Query
    ↓
+-------------------+
| 1. Parser         | → Syntax check, create parse tree
+-------------------+
    ↓
+-------------------+
| 2. Optimizer      | → Generate possible execution plans
|    (Cost-based)   | → Estimate costs for each plan
|                   | → Choose lowest cost plan
+-------------------+
    ↓
+-------------------+
| 3. Executor       | → Execute chosen plan
+-------------------+
    ↓
Result Set
```

### Sample Data for Examples

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    registration_date DATE
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(12,2),
    status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(200),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- Assume tables have 100,000+ rows for realistic optimization scenarios
```

---

## Execution Plan (EXPLAIN) (실행 계획)

### Concept (개념)
An execution plan (실행 계획) shows how the database will execute a query. The EXPLAIN command reveals the steps the database takes, including which indexes are used and how tables are accessed.

### Basic EXPLAIN Syntax

```sql
-- MySQL
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- MySQL with additional format options
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN FORMAT=TREE SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;  -- Actually executes

-- PostgreSQL
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT * FROM orders WHERE customer_id = 123;

-- SQL Server
SET SHOWPLAN_TEXT ON;
SELECT * FROM orders WHERE customer_id = 123;
-- Or graphical plan in SSMS
```

### MySQL EXPLAIN Output Columns

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

/*
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | orders | ref  | idx_customer  | idx  | 4       | const| 15   | Using where |
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
*/
```

### EXPLAIN Columns Explained

| Column | Korean | Description |
|--------|--------|-------------|
| id | 식별자 | Query block identifier |
| select_type | 쿼리 유형 | SIMPLE, PRIMARY, SUBQUERY, etc. |
| table | 테이블 | Table being accessed |
| type | 접근 유형 | How rows are found (see below) |
| possible_keys | 사용 가능 인덱스 | Indexes that could be used |
| key | 사용된 인덱스 | Index actually chosen |
| key_len | 키 길이 | Bytes of index used |
| ref | 참조 | Columns/constants compared |
| rows | 예상 행 수 | Estimated rows to examine |
| Extra | 추가 정보 | Additional execution details |

### Access Types (type column) - Best to Worst

```
Performance (Best to Worst):
+----------+--------------------------------------------------+
| Type     | Description                                      |
+----------+--------------------------------------------------+
| system   | Table has only one row                           |
| const    | Exactly one matching row (PRIMARY KEY/UNIQUE)    |
| eq_ref   | One row per row from previous table (JOIN)       |
| ref      | Multiple rows from index                         |
| range    | Index range scan (BETWEEN, <, >, IN)             |
| index    | Full index scan (all index entries)              |
| ALL      | Full table scan (worst - reads every row)        |
+----------+--------------------------------------------------+

Visualization:
                                    Performance
                                    ←───────────────→
+--------+--------+--------+-------+-------+-------+
| system | const  | eq_ref | ref   | range | index | ALL |
+--------+--------+--------+-------+-------+-------+-----+
   Best                                              Worst
```

### Extra Column Values

```sql
/*
Common Extra values:
+---------------------+------------------------------------------------+
| Value               | Meaning                                        |
+---------------------+------------------------------------------------+
| Using index         | Covering index (no table access)               |
| Using where         | WHERE filtering after retrieving rows          |
| Using temporary     | Temporary table needed (GROUP BY, DISTINCT)    |
| Using filesort      | Additional sorting needed (ORDER BY)           |
| Using index condition| Index condition pushdown (ICP)                 |
| Using join buffer   | Join buffer used (Block Nested Loop)           |
+---------------------+------------------------------------------------+

Warning signs (may need optimization):
  - Using temporary
  - Using filesort
  - ALL type with large tables
  - No key used (NULL in key column)
*/
```

---

## Reading Execution Plans (실행 계획 해석)

### Example 1: Simple Query

```sql
-- Query
SELECT * FROM orders WHERE order_id = 12345;

-- EXPLAIN output
/*
+----+-------------+--------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table  | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+--------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | orders | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
+----+-------------+--------+-------+---------------+---------+---------+-------+------+-------+
*/

-- Interpretation:
-- type=const: Using PRIMARY KEY, exactly one row
-- rows=1: Perfect, only reading one row
-- This is an optimal query!
```

### Example 2: Range Scan

```sql
-- Query
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';

-- Without index:
/*
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
|  1 | SIMPLE      | orders | ALL  | NULL          | NULL | NULL    | NULL | 150000 | Using where |
+----+-------------+--------+------+---------------+------+---------+------+--------+-------------+
*/
-- Problem: type=ALL means full table scan!

-- After adding index:
CREATE INDEX idx_order_date ON orders (order_date);

/*
+----+-------------+--------+-------+----------------+----------------+---------+------+------+-------------+
| id | select_type | table  | type  | possible_keys  | key            | key_len | ref  | rows | Extra       |
+----+-------------+--------+-------+----------------+----------------+---------+------+------+-------------+
|  1 | SIMPLE      | orders | range | idx_order_date | idx_order_date | 3       | NULL | 3500 | Using where |
+----+-------------+--------+-------+----------------+----------------+---------+------+------+-------------+
*/
-- Better: type=range, only 3500 rows instead of 150000
```

### Example 3: JOIN Query

```sql
-- Query
SELECT c.name, o.order_date, o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.city = 'Seoul';

-- EXPLAIN output:
/*
+----+-------------+-------+------+------------------+--------------+---------+-------------------+------+-------------+
| id | select_type | table | type | possible_keys    | key          | key_len | ref               | rows | Extra       |
+----+-------------+-------+------+------------------+--------------+---------+-------------------+------+-------------+
|  1 | SIMPLE      | c     | ref  | PRIMARY,idx_city | idx_city     | 52      | const             | 5000 | Using where |
|  1 | SIMPLE      | o     | ref  | idx_customer     | idx_customer | 4       | db.c.customer_id  | 15   |             |
+----+-------------+-------+------+------------------+--------------+---------+-------------------+------+-------------+
*/

-- Reading order: Process tables top to bottom
-- 1. First: customers table (c)
--    - Uses idx_city index to find customers in Seoul
--    - Estimates 5000 rows
-- 2. Second: orders table (o)
--    - For each customer, uses idx_customer to find their orders
--    - Estimates 15 orders per customer
-- Total estimated rows: 5000 * 15 = 75000
```

### Execution Plan Diagram

```
Query: SELECT c.name, o.order_date FROM customers c
       JOIN orders o ON c.customer_id = o.customer_id
       WHERE c.city = 'Seoul'

Execution Flow:
+------------------+
| Index Scan       |    Step 1: Find customers in Seoul
| idx_city         |    using city index
| (customers)      |
| rows: ~5000      |
+--------+---------+
         |
         v
+--------+---------+
| Nested Loop      |    Step 2: For each customer,
| JOIN             |    lookup their orders
+--------+---------+
         |
         v
+--------+---------+
| Index Scan       |    Step 3: Find orders for
| idx_customer     |    each customer_id
| (orders)         |
| rows: ~15 each   |
+--------+---------+
         |
         v
+--------+---------+
| Result           |    Return matched rows
+------------------+
```

---

## Full Table Scan vs Index Scan (전체 테이블 스캔 vs 인덱스 스캔)

### Concept (개념)

```
Full Table Scan (전체 테이블 스캔):
+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 | ... scan all rows
+---+---+---+---+---+---+---+---+---+---+
 ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑
Check every single row

Index Scan (인덱스 스캔):
    Index B-Tree
       [50]
      /    \
   [25]    [75]
   / \     /  \
 [10][30][60][90]
   ↓
Jump directly to needed rows
```

### When Each is Used

```sql
-- Full Table Scan scenarios:

-- 1. No suitable index
SELECT * FROM orders WHERE YEAR(order_date) = 2024;
-- Function on column prevents index usage

-- 2. Low selectivity condition
SELECT * FROM orders WHERE status = 'active';
-- If 90% of rows are 'active', index doesn't help

-- 3. Selecting most of the table
SELECT * FROM orders WHERE total_amount > 0;
-- Need most rows anyway, sequential read is faster

-- 4. Small tables
SELECT * FROM config_settings;
-- Table fits in memory, full scan is fast
```

### Index Usage Killers (인덱스 무력화)

```sql
-- 1. Functions on indexed columns
-- BAD: Function prevents index usage
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- GOOD: Rewrite without function
SELECT * FROM orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';


-- 2. Implicit type conversion
-- BAD: customer_id is INT but compared with string
SELECT * FROM orders WHERE customer_id = '123';

-- GOOD: Use correct type
SELECT * FROM orders WHERE customer_id = 123;


-- 3. OR conditions with non-indexed columns
-- BAD: Cannot use index efficiently
SELECT * FROM orders
WHERE customer_id = 123 OR total_amount > 1000;

-- GOOD: Use UNION instead
SELECT * FROM orders WHERE customer_id = 123
UNION
SELECT * FROM orders WHERE total_amount > 1000;


-- 4. LIKE with leading wildcard
-- BAD: Cannot use index
SELECT * FROM products WHERE name LIKE '%phone%';

-- GOOD: If possible, use prefix match
SELECT * FROM products WHERE name LIKE 'phone%';


-- 5. NOT and != operators
-- BAD: Usually cannot use index
SELECT * FROM orders WHERE status != 'cancelled';

-- GOOD: If possible, rewrite with positive condition
SELECT * FROM orders WHERE status IN ('pending', 'shipped', 'delivered');


-- 6. NULL comparisons in some cases
-- May or may not use index depending on database
SELECT * FROM orders WHERE customer_id IS NULL;
```

### Forcing Index Usage (Hints)

```sql
-- MySQL: Force index usage
SELECT * FROM orders FORCE INDEX (idx_order_date)
WHERE order_date > '2024-01-01';

-- MySQL: Ignore index
SELECT * FROM orders IGNORE INDEX (idx_order_date)
WHERE order_date > '2024-01-01';

-- PostgreSQL: Disable specific scan types for testing
SET enable_seqscan = OFF;  -- Disable sequential scan
SET enable_indexscan = OFF;  -- Disable index scan

-- Note: Index hints should be used carefully
-- The optimizer usually makes good choices
```

---

## Query Tuning Techniques (쿼리 튜닝 기법)

### 1. SELECT Only Needed Columns

```sql
-- BAD: SELECT * retrieves unnecessary data
SELECT * FROM orders WHERE customer_id = 123;

-- GOOD: Select only required columns
SELECT order_id, order_date, total_amount
FROM orders WHERE customer_id = 123;

-- Benefits:
-- - Less data transferred
-- - Potential for covering index
-- - Reduced memory usage
```

### 2. Use Appropriate JOIN Types

```sql
-- Understand JOIN algorithms:
/*
+------------------+------------------------------------------+
| Algorithm        | Best For                                 |
+------------------+------------------------------------------+
| Nested Loop Join | Small outer table, indexed inner table   |
| Hash Join        | Large tables, no usable index            |
| Merge Join       | Both tables sorted on join column        |
+------------------+------------------------------------------+
*/

-- Nested Loop visualization:
-- For each row in customers:
--     For each matching row in orders:
--         Output row

-- Ensure proper indexes for JOIN columns
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- The inner table (orders) should have index on join column
SELECT c.name, o.order_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

### 3. Optimize Subqueries

```sql
-- BAD: Correlated subquery (executes once per row)
SELECT c.name,
       (SELECT COUNT(*) FROM orders o
        WHERE o.customer_id = c.customer_id) AS order_count
FROM customers c;

-- GOOD: Use JOIN with aggregation
SELECT c.name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;

-- Or use a derived table/CTE
WITH order_counts AS (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
)
SELECT c.name, COALESCE(oc.order_count, 0) AS order_count
FROM customers c
LEFT JOIN order_counts oc ON c.customer_id = oc.customer_id;
```

### 4. Optimize GROUP BY and ORDER BY

```sql
-- Ensure index matches GROUP BY columns
CREATE INDEX idx_orders_date_status ON orders (order_date, status);

-- This query can use the index
SELECT order_date, status, COUNT(*)
FROM orders
GROUP BY order_date, status;

-- Avoid sorting if possible
-- If you need only top N, use LIMIT
SELECT customer_id, SUM(total_amount) AS total
FROM orders
GROUP BY customer_id
ORDER BY total DESC
LIMIT 10;

-- Consider using index for ORDER BY
CREATE INDEX idx_orders_amount ON orders (total_amount DESC);
```

### 5. Batch Operations

```sql
-- BAD: Many small queries
-- In application code:
-- for customer_id in customer_ids:
--     SELECT * FROM orders WHERE customer_id = ?

-- GOOD: Single batch query
SELECT * FROM orders
WHERE customer_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

-- For INSERT operations:
-- BAD: Individual inserts
INSERT INTO orders VALUES (1, ...);
INSERT INTO orders VALUES (2, ...);
INSERT INTO orders VALUES (3, ...);

-- GOOD: Batch insert
INSERT INTO orders VALUES
(1, ...),
(2, ...),
(3, ...);
```

### 6. Use LIMIT for Large Results

```sql
-- When you only need a few rows, use LIMIT
SELECT * FROM orders
ORDER BY order_date DESC
LIMIT 10;

-- Pagination with LIMIT and OFFSET
SELECT * FROM orders
ORDER BY order_id
LIMIT 20 OFFSET 40;  -- Page 3 (rows 41-60)

-- Better pagination for large datasets (keyset pagination)
-- Instead of OFFSET, use WHERE with last seen value
SELECT * FROM orders
WHERE order_id > 1234  -- last order_id from previous page
ORDER BY order_id
LIMIT 20;
```

---

## N+1 Problem and Solutions (N+1 문제와 해결책)

### What is N+1 Problem? (N+1 문제란?)

The N+1 problem occurs when you execute N additional queries for N parent records, plus 1 initial query. This is extremely inefficient and common in ORM-based applications.

```
N+1 Problem Visualization:

Initial Query (1):
+-------------------+
| SELECT * FROM     |
| customers         |
| LIMIT 100         |
+-------------------+
        ↓
Returns 100 customers

Additional Queries (N = 100):
For customer 1: SELECT * FROM orders WHERE customer_id = 1
For customer 2: SELECT * FROM orders WHERE customer_id = 2
For customer 3: SELECT * FROM orders WHERE customer_id = 3
...
For customer 100: SELECT * FROM orders WHERE customer_id = 100

Total: 1 + 100 = 101 queries!
```

### Example of N+1 Problem

```sql
-- Simulating N+1 in application code (pseudo-code):

-- Query 1: Get all customers
customers = execute("SELECT * FROM customers LIMIT 100");

-- N more queries: Get orders for each customer
for customer in customers:
    orders = execute("SELECT * FROM orders WHERE customer_id = " + customer.id);
    customer.orders = orders;

-- This generates 101 queries for 100 customers!
```

### Solution 1: JOIN (조인)

```sql
-- Instead of N+1, use a single JOIN query
SELECT
    c.customer_id,
    c.name,
    c.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id IN (1, 2, 3, ... 100);

-- Result includes all data in one query
-- Application code groups orders by customer
```

### Solution 2: Batch Loading (배치 로딩)

```sql
-- Query 1: Get customers
SELECT * FROM customers LIMIT 100;
-- Returns customer_ids: [1, 2, 3, ... 100]

-- Query 2: Get ALL orders for those customers in ONE query
SELECT * FROM orders
WHERE customer_id IN (1, 2, 3, 4, 5, ... 100);

-- Only 2 queries instead of 101!
-- Application code matches orders to customers

-- Implementation:
customer_ids = [c.id for c in customers]  -- [1, 2, ... 100]
orders = execute("SELECT * FROM orders WHERE customer_id IN (" +
                 ",".join(customer_ids) + ")");
-- Group orders by customer_id in application memory
```

### Solution 3: Eager Loading (ORM)

```sql
-- Many ORMs provide eager loading options

-- Django example:
# BAD (N+1)
customers = Customer.objects.all()[:100]
for c in customers:
    print(c.orders.all())  # Triggers N queries

# GOOD (Eager load)
customers = Customer.objects.prefetch_related('orders')[:100]
for c in customers:
    print(c.orders.all())  # No additional queries

-- SQLAlchemy example:
# BAD
customers = session.query(Customer).limit(100).all()
for c in customers:
    print(c.orders)  # N+1 queries

# GOOD
customers = session.query(Customer).options(
    joinedload(Customer.orders)
).limit(100).all()
```

### Solution 4: Subquery Loading

```sql
-- Use subquery to load related data efficiently
-- This is what ORMs do with subqueryload

-- First query: Get customers
SELECT * FROM customers WHERE ... LIMIT 100;

-- Second query: Get orders using subquery
SELECT * FROM orders
WHERE customer_id IN (
    SELECT customer_id FROM customers WHERE ... LIMIT 100
);
```

### N+1 Detection

```sql
-- Signs of N+1 problem:
-- 1. Many similar queries in slow query log
-- 2. Query count scales with data size
-- 3. Poor performance that worsens with more records

-- MySQL: Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.1;

-- Check for repetitive queries
-- Look for patterns like:
-- SELECT * FROM orders WHERE customer_id = 1
-- SELECT * FROM orders WHERE customer_id = 2
-- SELECT * FROM orders WHERE customer_id = 3
-- ...
```

### Performance Comparison

```
Scenario: 100 customers, each with 10 orders

N+1 Approach:
+-----------------+------------------+
| Queries         | 101              |
| Network trips   | 101              |
| DB overhead     | 101 query parses |
| Time estimate   | ~500ms           |
+-----------------+------------------+

JOIN Approach:
+-----------------+------------------+
| Queries         | 1                |
| Network trips   | 1                |
| DB overhead     | 1 query parse    |
| Time estimate   | ~20ms            |
+-----------------+------------------+

Batch Loading:
+-----------------+------------------+
| Queries         | 2                |
| Network trips   | 2                |
| DB overhead     | 2 query parses   |
| Time estimate   | ~25ms            |
+-----------------+------------------+
```

---

## Query Optimization Checklist (쿼리 최적화 체크리스트)

```
+------------------------------------------------------------+
| Query Optimization Checklist                                |
+------------------------------------------------------------+
| [ ] Run EXPLAIN and analyze the execution plan              |
| [ ] Check for full table scans on large tables              |
| [ ] Verify indexes are being used                           |
| [ ] Look for "Using temporary" or "Using filesort"          |
| [ ] Ensure JOIN columns are indexed                         |
| [ ] Replace SELECT * with specific columns                  |
| [ ] Check for N+1 query patterns                            |
| [ ] Avoid functions on indexed columns in WHERE             |
| [ ] Use appropriate data types to avoid implicit conversion |
| [ ] Consider query rewrite (subquery to JOIN, etc.)         |
| [ ] Test with realistic data volumes                        |
| [ ] Monitor slow query log regularly                        |
+------------------------------------------------------------+
```

---

## Summary (요약)

| Topic | Korean | Key Points |
|-------|--------|------------|
| EXPLAIN | 실행 계획 | Shows how database executes query |
| Access Type | 접근 유형 | const > eq_ref > ref > range > index > ALL |
| Full Table Scan | 전체 테이블 스캔 | Reads every row, avoid for large tables |
| Index Scan | 인덱스 스캔 | Uses index to find rows efficiently |
| Query Tuning | 쿼리 튜닝 | Rewrite queries for better performance |
| N+1 Problem | N+1 문제 | Execute 1+N queries instead of 1 or 2 |

### Quick Performance Wins

```
1. Add missing indexes for WHERE and JOIN columns
2. Replace SELECT * with specific columns
3. Fix N+1 problems with JOIN or batch loading
4. Avoid functions on indexed columns
5. Use LIMIT when only few rows needed
6. Rewrite correlated subqueries as JOINs
7. Ensure proper data types (avoid implicit conversion)
```

---
