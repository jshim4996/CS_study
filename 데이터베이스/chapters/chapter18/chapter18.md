# Chapter 18. Index Design (인덱스 설계)

> Learn how to design and implement effective database indexes to dramatically improve query performance.

---

## What is an Index? (인덱스란?)

### Concept (개념)
An index (인덱스) is a data structure that improves the speed of data retrieval operations on a database table. Similar to a book's index, it allows the database to find data without scanning every row in the table.

### How Indexes Work

```
Without Index (Full Table Scan):
+--------+------------+--------+
| emp_id | name       | salary |
+--------+------------+--------+
|   1    | Kim        | 50000  |  ← Scan
|   2    | Lee        | 60000  |  ← Scan
|   3    | Park       | 55000  |  ← Scan
|   ...  | ...        | ...    |  ← Scan every row
| 10000  | Choi       | 70000  |  ← Finally found!
+--------+------------+--------+

With Index (B-Tree Index on salary):
                    [55000]
                   /       \
            [50000]         [65000]
           /      \        /       \
        [45000] [52000] [60000] [70000]
                              ↓
                         Points to row with salary=70000
                         (Only 3-4 lookups instead of 10000!)
```

### Index Types

| Index Type | Korean | Description | Use Case |
|------------|--------|-------------|----------|
| B-Tree | B-트리 인덱스 | Balanced tree structure | Default, range queries |
| Hash | 해시 인덱스 | Hash table structure | Exact match only |
| Bitmap | 비트맵 인덱스 | Bit arrays | Low cardinality columns |
| Full-Text | 전문 인덱스 | Text search index | Searching text content |
| Spatial | 공간 인덱스 | Geographic data | GIS applications |

---

## CREATE INDEX Syntax (인덱스 생성 구문)

### Basic Syntax

```sql
-- Create a simple index
CREATE INDEX index_name ON table_name (column_name);

-- Create a unique index (ensures uniqueness)
CREATE UNIQUE INDEX index_name ON table_name (column_name);

-- Drop an index
DROP INDEX index_name ON table_name;  -- MySQL
DROP INDEX index_name;                 -- PostgreSQL
```

### Sample Data

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    status VARCHAR(20)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    order_date DATE,
    quantity INT,
    total_amount DECIMAL(12,2),
    status VARCHAR(20)
);

-- Insert sample data
INSERT INTO employees VALUES
(1, 'Kim Minjun', 'kim@company.com', 'Engineering', 85000, '2019-03-15', 'active'),
(2, 'Lee Soyeon', 'lee@company.com', 'Engineering', 75000, '2020-06-20', 'active'),
(3, 'Park Jihoon', 'park@company.com', 'Marketing', 65000, '2021-01-10', 'active'),
(4, 'Choi Yuna', 'choi@company.com', 'Marketing', 70000, '2020-09-01', 'inactive'),
(5, 'Jung Seungho', 'jung@company.com', 'HR', 55000, '2022-02-15', 'active');
```

### Creating Different Types of Indexes

```sql
-- Simple single-column index
CREATE INDEX idx_employees_department ON employees (department);

-- Unique index
CREATE UNIQUE INDEX idx_employees_email ON employees (email);

-- Index with sort order (PostgreSQL, some MySQL versions)
CREATE INDEX idx_employees_salary_desc ON employees (salary DESC);

-- Partial index (PostgreSQL) - index only active employees
CREATE INDEX idx_active_employees ON employees (department)
WHERE status = 'active';

-- MySQL: Prefix index for long text columns
CREATE INDEX idx_name_prefix ON employees (name(20));
```

---

## Composite Index (복합 인덱스)

### Concept (개념)
A composite index (복합 인덱스) is an index on multiple columns. The order of columns in the index is crucial for query optimization.

### Leftmost Prefix Rule (왼쪽 접두사 규칙)

```
Composite Index: (A, B, C)

Queries that CAN use this index:
  ✓ WHERE A = ?
  ✓ WHERE A = ? AND B = ?
  ✓ WHERE A = ? AND B = ? AND C = ?
  ✓ WHERE A = ? AND C = ?  (uses only A portion)
  ✓ ORDER BY A
  ✓ ORDER BY A, B
  ✓ ORDER BY A, B, C

Queries that CANNOT use this index:
  ✗ WHERE B = ?           (doesn't start with A)
  ✗ WHERE C = ?           (doesn't start with A)
  ✗ WHERE B = ? AND C = ? (doesn't start with A)
  ✗ ORDER BY B, A         (wrong order)
```

### Visualizing Composite Index

```
Composite Index on (department, salary, hire_date):

Level 1: department
+-------------+-------------+-------------+
| Engineering | HR          | Marketing   |
+-------------+-------------+-------------+
      |              |             |
      v              v             v

Level 2: salary (within each department)
+-------+-------+   +-------+   +-------+-------+
| 75000 | 85000 |   | 55000 |   | 65000 | 70000 |
+-------+-------+   +-------+   +-------+-------+
    |       |           |           |       |
    v       v           v           v       v

Level 3: hire_date (within each department+salary)
```

### Example

```sql
-- Create composite index
CREATE INDEX idx_emp_dept_salary ON employees (department, salary);

-- Query 1: Uses the full index
SELECT * FROM employees
WHERE department = 'Engineering' AND salary > 70000;
-- Index used: idx_emp_dept_salary (both columns)

-- Query 2: Uses only the first column
SELECT * FROM employees
WHERE department = 'Marketing';
-- Index used: idx_emp_dept_salary (department only)

-- Query 3: Cannot use the index efficiently
SELECT * FROM employees
WHERE salary > 70000;
-- Index NOT used: query doesn't include 'department'

-- Better: Create a separate index for salary-only queries
CREATE INDEX idx_emp_salary ON employees (salary);
```

### Column Order Strategy

```sql
-- General rule: Order columns by:
-- 1. Equality conditions first
-- 2. Range conditions last
-- 3. Most selective columns first

-- Example: Finding active employees in a department with salary range
-- Query pattern: WHERE department = ? AND status = ? AND salary > ?

-- Good: Equality columns first, range column last
CREATE INDEX idx_good ON employees (department, status, salary);

-- Bad: Range column in the middle (limits index usage)
CREATE INDEX idx_bad ON employees (department, salary, status);
```

---

## Index Selectivity (인덱스 선택도)

### Concept (개념)
Index selectivity (선택도) measures how unique the values in an indexed column are. High selectivity means more unique values, which makes the index more effective.

### Calculating Selectivity

```
Selectivity = Number of Distinct Values / Total Number of Rows

High Selectivity (Good for indexing):
  - email: 10000 distinct / 10000 rows = 1.0 (100% unique)
  - employee_id: 10000 / 10000 = 1.0

Low Selectivity (Poor for indexing):
  - gender: 2 / 10000 = 0.0002 (only M/F)
  - status: 3 / 10000 = 0.0003 (active/inactive/pending)
```

### Selectivity Diagram

```
High Selectivity (email):
+------------------+
| email            | ← Each value points to ~1 row
+------------------+
| kim@example.com  | → Row 1
| lee@example.com  | → Row 2
| park@example.com | → Row 3
| ...              |
+------------------+

Low Selectivity (status):
+------------------+
| status           | ← Each value points to many rows
+------------------+
| active           | → Row 1, 2, 4, 5, 7, 8, 9, 10...
| inactive         | → Row 3, 6, 11, 15...
+------------------+
```

### Checking Selectivity in Practice

```sql
-- Calculate selectivity for each column
SELECT
    COUNT(DISTINCT department) / COUNT(*) AS dept_selectivity,
    COUNT(DISTINCT email) / COUNT(*) AS email_selectivity,
    COUNT(DISTINCT status) / COUNT(*) AS status_selectivity,
    COUNT(DISTINCT salary) / COUNT(*) AS salary_selectivity
FROM employees;

/*
Result:
| dept_selectivity | email_selectivity | status_selectivity | salary_selectivity |
|------------------|-------------------|--------------------|--------------------|
| 0.0003           | 1.0000            | 0.0002             | 0.8500             |

Interpretation:
- email: Excellent for indexing (unique)
- salary: Good for indexing (many distinct values)
- department: Poor for indexing alone (few distinct values)
- status: Poor for indexing alone (very few distinct values)
*/

-- View index statistics (MySQL)
SHOW INDEX FROM employees;

-- Analyze table to update statistics (MySQL)
ANALYZE TABLE employees;

-- PostgreSQL: View index statistics
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE tablename = 'employees';
```

### Selectivity and Composite Indexes

```sql
-- Low selectivity columns can be useful in composite indexes
-- when combined with high selectivity columns

-- status alone: low selectivity (not useful)
-- department alone: low selectivity (not useful)
-- (department, status, employee_id): Combined high selectivity!

CREATE INDEX idx_dept_status_emp ON employees (department, status, employee_id);

-- This works well for queries like:
SELECT * FROM employees
WHERE department = 'Engineering'
  AND status = 'active'
  AND employee_id > 100;
```

---

## Covering Index (커버링 인덱스)

### Concept (개념)
A covering index (커버링 인덱스) includes all columns needed by a query, allowing the database to retrieve data entirely from the index without accessing the table data (index-only scan).

### How Covering Index Works

```
Regular Index Query:
+------------------+         +------------------+
| Index            |         | Table Data       |
| (department)     |         | (all columns)    |
+------------------+         +------------------+
| Engineering → ---|-------->| Row 1: Full data |
| Marketing → -----|-------->| Row 2: Full data |
+------------------+         +------------------+
     Step 1                       Step 2
  (Index Lookup)              (Table Access)

Covering Index Query:
+----------------------------------+
| Covering Index                   |
| (department, name, salary)       |
+----------------------------------+
| Engineering, Kim, 85000          | ← All needed data in index!
| Engineering, Lee, 75000          |
| Marketing, Park, 65000           |
+----------------------------------+
     No table access needed!
```

### Example

```sql
-- Query to optimize
SELECT department, name, salary
FROM employees
WHERE department = 'Engineering';

-- Regular index: requires table lookup
CREATE INDEX idx_dept ON employees (department);

-- Covering index: no table lookup needed
CREATE INDEX idx_dept_covering ON employees (department, name, salary);

-- MySQL: Check if covering index is used
EXPLAIN SELECT department, name, salary
FROM employees
WHERE department = 'Engineering';

/*
Result shows "Using index" in Extra column when covering index is used
+----+-------------+-----------+------+-------------------+-------------------+---------+-------+------+-------------+
| id | select_type | table     | type | possible_keys     | key               | key_len | ref   | rows | Extra       |
+----+-------------+-----------+------+-------------------+-------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | employees | ref  | idx_dept_covering | idx_dept_covering | 52      | const |    2 | Using index |
+----+-------------+-----------+------+-------------------+-------------------+---------+-------+------+-------------+
*/

-- PostgreSQL: Check for Index Only Scan
EXPLAIN ANALYZE SELECT department, name, salary
FROM employees
WHERE department = 'Engineering';

/*
Result shows "Index Only Scan" when covering index is used:
Index Only Scan using idx_dept_covering on employees (cost=0.14..8.16 rows=2 width=64)
*/
```

### Include Columns (MySQL 8.0+, PostgreSQL)

```sql
-- PostgreSQL: INCLUDE clause for covering indexes
-- Included columns are stored in index but not used for searching
CREATE INDEX idx_dept_include ON employees (department)
INCLUDE (name, salary);

-- Benefits:
-- - Index is smaller (included columns not in B-tree structure)
-- - Still provides covering index benefits
-- - Good when you filter by department but need to return name, salary

-- MySQL 8.0+: Functional indexes for similar purpose
CREATE INDEX idx_dept_name ON employees (department, name, salary);
```

### When to Use Covering Indexes

```sql
-- Good candidates for covering indexes:
-- 1. Frequently executed queries
-- 2. Queries returning few columns
-- 3. Queries with specific column requirements

-- Example: Dashboard query executed thousands of times per day
-- Get department summary
SELECT department, COUNT(*), AVG(salary)
FROM employees
WHERE status = 'active'
GROUP BY department;

-- Covering index for this query
CREATE INDEX idx_dept_summary ON employees (status, department, salary);

-- Bad candidates:
-- 1. SELECT * queries (would need all columns in index)
-- 2. Rarely executed queries
-- 3. Tables with many updates (index maintenance overhead)
```

---

## Index Design Strategies (인덱스 설계 전략)

### Strategy 1: Analyze Query Patterns

```sql
-- Step 1: Identify frequent queries
-- Common query patterns in an e-commerce system:

-- Pattern 1: Get orders by customer (high frequency)
SELECT * FROM orders WHERE customer_id = ?;

-- Pattern 2: Get orders by date range (high frequency)
SELECT * FROM orders WHERE order_date BETWEEN ? AND ?;

-- Pattern 3: Get orders by status (medium frequency)
SELECT * FROM orders WHERE status = ?;

-- Pattern 4: Combined search (high frequency)
SELECT * FROM orders
WHERE customer_id = ?
  AND order_date BETWEEN ? AND ?
  AND status = ?;

-- Step 2: Design indexes based on patterns
CREATE INDEX idx_orders_customer ON orders (customer_id);
CREATE INDEX idx_orders_date ON orders (order_date);
CREATE INDEX idx_orders_combined ON orders (customer_id, status, order_date);
-- Note: customer_id first (equality), order_date last (range)
```

### Strategy 2: Consider Write vs Read Ratio

```
High Read, Low Write (e.g., product catalog):
+-----------------------------------------+
| More indexes are beneficial             |
| - Create indexes for common queries     |
| - Covering indexes for frequent queries |
| - Index maintenance cost is low         |
+-----------------------------------------+

High Write, Low Read (e.g., log tables):
+-----------------------------------------+
| Minimal indexes only                    |
| - Only essential indexes (PK, FK)       |
| - Each INSERT/UPDATE must update indexes|
| - Consider no secondary indexes         |
+-----------------------------------------+

Balanced Read/Write (e.g., orders table):
+-----------------------------------------+
| Strategic indexing                      |
| - Index most critical queries           |
| - Monitor and adjust based on performance|
| - Consider partial indexes              |
+-----------------------------------------+
```

### Strategy 3: Monitor and Adjust

```sql
-- MySQL: Find unused indexes
SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    i.INDEX_NAME,
    s.ROWS_READ,
    s.ROWS_INSERTED,
    s.ROWS_UPDATED,
    s.ROWS_DELETED
FROM information_schema.TABLES t
JOIN information_schema.STATISTICS i
    ON t.TABLE_SCHEMA = i.TABLE_SCHEMA
    AND t.TABLE_NAME = i.TABLE_NAME
LEFT JOIN performance_schema.table_io_waits_summary_by_index_usage s
    ON t.TABLE_SCHEMA = s.OBJECT_SCHEMA
    AND t.TABLE_NAME = s.OBJECT_NAME
    AND i.INDEX_NAME = s.INDEX_NAME
WHERE t.TABLE_SCHEMA = 'your_database'
  AND s.INDEX_NAME IS NOT NULL
  AND s.COUNT_READ = 0;

-- PostgreSQL: Find unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Remove unused indexes to improve write performance
-- DROP INDEX idx_unused_index;
```

### Strategy 4: Index Size Considerations

```sql
-- Check index sizes (MySQL)
SELECT
    TABLE_NAME,
    INDEX_NAME,
    ROUND(STAT_VALUE * @@innodb_page_size / 1024 / 1024, 2) AS size_mb
FROM mysql.innodb_index_stats
WHERE stat_name = 'size'
  AND database_name = 'your_database'
ORDER BY STAT_VALUE DESC;

-- PostgreSQL: Check index sizes
SELECT
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Index Design Checklist

```
+--------------------------------------------------+
| Index Design Checklist                           |
+--------------------------------------------------+
| [ ] Identify top 10 most frequent queries        |
| [ ] Add indexes for WHERE clause columns         |
| [ ] Add indexes for JOIN columns                 |
| [ ] Add indexes for ORDER BY columns             |
| [ ] Consider composite indexes for multi-column  |
|     conditions                                   |
| [ ] Check selectivity before creating indexes    |
| [ ] Consider covering indexes for critical       |
|     queries                                      |
| [ ] Monitor index usage after creation           |
| [ ] Remove unused indexes periodically           |
| [ ] Balance between read and write performance   |
+--------------------------------------------------+
```

---

## Anti-Patterns to Avoid (피해야 할 안티패턴)

### 1. Over-Indexing

```sql
-- BAD: Creating an index for every column
CREATE INDEX idx_1 ON employees (name);
CREATE INDEX idx_2 ON employees (email);
CREATE INDEX idx_3 ON employees (department);
CREATE INDEX idx_4 ON employees (salary);
CREATE INDEX idx_5 ON employees (hire_date);
CREATE INDEX idx_6 ON employees (status);

-- Problems:
-- - Slow INSERT/UPDATE/DELETE operations
-- - Increased storage space
-- - Query optimizer may make poor choices

-- GOOD: Create indexes based on actual query patterns
CREATE INDEX idx_dept_status ON employees (department, status);
CREATE UNIQUE INDEX idx_email ON employees (email);
```

### 2. Indexing Low Selectivity Columns Alone

```sql
-- BAD: Index on boolean or status column alone
CREATE INDEX idx_status ON employees (status);
-- With only 2-3 distinct values, not useful

-- GOOD: Combine with other columns
CREATE INDEX idx_dept_status ON employees (department, status);
-- Or use partial index (PostgreSQL)
CREATE INDEX idx_active ON employees (department) WHERE status = 'active';
```

### 3. Wrong Column Order in Composite Index

```sql
-- Query to optimize:
SELECT * FROM orders
WHERE customer_id = 123
  AND order_date > '2024-01-01';

-- BAD: Range column first
CREATE INDEX idx_bad ON orders (order_date, customer_id);
-- Only order_date portion can be used efficiently

-- GOOD: Equality column first
CREATE INDEX idx_good ON orders (customer_id, order_date);
-- Both columns can be used efficiently
```

### 4. Ignoring Index Maintenance

```sql
-- Indexes can become fragmented over time
-- Regular maintenance is important

-- MySQL: Optimize table (rebuilds indexes)
OPTIMIZE TABLE employees;

-- PostgreSQL: Rebuild index
REINDEX INDEX idx_employees_dept;

-- PostgreSQL: Rebuild all indexes on a table
REINDEX TABLE employees;

-- Schedule maintenance during low-traffic periods
```

---

## Summary (요약)

| Topic | Korean | Key Points |
|-------|--------|------------|
| Index | 인덱스 | Data structure for faster data retrieval |
| CREATE INDEX | 인덱스 생성 | Basic syntax for creating indexes |
| Composite Index | 복합 인덱스 | Multi-column index, leftmost prefix rule applies |
| Selectivity | 선택도 | Higher is better, unique values / total rows |
| Covering Index | 커버링 인덱스 | Contains all columns needed by query |
| Design Strategy | 설계 전략 | Based on query patterns, read/write ratio |

### Quick Reference

```
When to Create Index:
  ✓ Frequently used in WHERE clause
  ✓ Used in JOIN conditions
  ✓ Used in ORDER BY
  ✓ High selectivity columns
  ✓ Foreign key columns

When NOT to Create Index:
  ✗ Small tables (< 1000 rows)
  ✗ Columns rarely used in queries
  ✗ Low selectivity columns alone
  ✗ Frequently updated columns
  ✗ Tables with heavy INSERT operations
```

---
