# Chapter 10. DML - Data Manipulation Language (데이터 조작 언어)

## Table of Contents (목차)
1. [DML Overview](#1-dml-overview-개요)
2. [INSERT Statement](#2-insert-statement-삽입문)
3. [UPDATE Statement](#3-update-statement-수정문)
4. [DELETE Statement](#4-delete-statement-삭제문)
5. [INSERT ... SELECT](#5-insert--select)
6. [UPSERT Patterns](#6-upsert-patterns)
7. [Summary](#7-summary-요약)

---

## 1. DML Overview (개요)

### What is DML? (DML이란?)

**DML (Data Manipulation Language)** is used to manipulate data within database tables. It allows you to insert, update, delete, and retrieve data.

**DML (데이터 조작 언어)**은 데이터베이스 테이블 내의 데이터를 조작하는 데 사용됩니다. 데이터의 삽입, 수정, 삭제, 조회를 가능하게 합니다.

```
+------------------+
|       DML        |
+------------------+
         |
    +----+----+----+----+
    |    |    |    |    |
    v    v    v    v    v
INSERT UPDATE DELETE SELECT MERGE
  |      |      |      |      |
  v      v      v      v      v
 Add   Modify  Remove  Read  Upsert
 Data   Data   Data   Data   Data
```

### DML Commands (DML 명령어)

| Command | Description (설명) | Example |
|---------|-------------------|---------|
| INSERT | Add new rows (새 행 추가) | INSERT INTO table VALUES(...) |
| UPDATE | Modify existing rows (기존 행 수정) | UPDATE table SET col = value |
| DELETE | Remove rows (행 삭제) | DELETE FROM table WHERE... |
| SELECT | Retrieve data (데이터 조회) | SELECT * FROM table |
| MERGE | Insert or Update (삽입 또는 수정) | MERGE INTO table... |

### DML vs DDL Characteristics (DML vs DDL 특징)

```
+------------------+------------------+
|       DML        |       DDL        |
+------------------+------------------+
| Manipulates data | Defines structure|
| Can be rolled    | Auto-committed   |
| back             |                  |
| Row-level ops    | Table-level ops  |
| Triggers fire    | No triggers      |
+------------------+------------------+
```

---

## 2. INSERT Statement (삽입문)

### Basic INSERT Syntax (기본 INSERT 구문)

```sql
-- Full syntax (전체 구문)
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- Simplified (all columns in order)
-- 간소화 (모든 열을 순서대로)
INSERT INTO table_name
VALUES (value1, value2, ...);
```

### INSERT Examples (INSERT 예제)

```sql
-- Sample table structure
-- 예제 테이블 구조
CREATE TABLE employees (
    employee_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE,
    salary DECIMAL(10, 2),
    department_id INT
);

-- Insert with column names (recommended)
-- 열 이름 지정 삽입 (권장)
INSERT INTO employees (first_name, last_name, email, hire_date, salary, department_id)
VALUES ('John', 'Doe', 'john.doe@company.com', '2024-01-15', 75000.00, 10);

-- Insert without column names (all columns in order)
-- 열 이름 없이 삽입 (모든 열을 순서대로)
INSERT INTO employees
VALUES (NULL, 'Jane', 'Smith', 'jane.smith@company.com', '2024-01-20', 80000.00, 20);
-- NULL for AUTO_INCREMENT column

-- Insert with partial columns (using defaults)
-- 부분 열 삽입 (기본값 사용)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Bob', 'Wilson', 'bob.wilson@company.com');
-- hire_date, salary, department_id will be NULL
```

### INSERT Multiple Rows (여러 행 삽입)

```sql
-- Insert multiple rows in single statement
-- 단일 문장으로 여러 행 삽입
INSERT INTO employees (first_name, last_name, email, salary)
VALUES
    ('Alice', 'Brown', 'alice.brown@company.com', 65000),
    ('Charlie', 'Davis', 'charlie.davis@company.com', 70000),
    ('Diana', 'Evans', 'diana.evans@company.com', 72000),
    ('Edward', 'Foster', 'edward.foster@company.com', 68000);

-- This is more efficient than multiple INSERT statements
-- 여러 INSERT 문보다 더 효율적입니다
```

### INSERT Flow Diagram (INSERT 흐름 다이어그램)

```
INSERT Statement
       |
       v
+------------------+
| Parse & Validate |
+------------------+
       |
       v
+------------------+
| Check Constraints|
| - NOT NULL       |
| - UNIQUE         |
| - FOREIGN KEY    |
| - CHECK          |
+------------------+
       |
   +---+---+
   |       |
   v       v
Success  Failure
   |       |
   v       v
Insert   Rollback
 Row     Error
```

### INSERT with DEFAULT Values (DEFAULT 값 삽입)

```sql
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE DEFAULT (CURRENT_DATE),
    status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) DEFAULT 0.00,
    notes TEXT
);

-- Use DEFAULT keyword
INSERT INTO orders (customer_id, status, total_amount)
VALUES (101, DEFAULT, 150.00);
-- order_date = current date, status = 'Pending'

-- Insert with all defaults
INSERT INTO orders (customer_id)
VALUES (102);
-- Uses all default values

-- Explicit DEFAULT in values list
INSERT INTO orders (customer_id, order_date, status, total_amount)
VALUES (103, DEFAULT, DEFAULT, 299.99);
```

### INSERT with NULL Values (NULL 값 삽입)

```sql
-- Explicit NULL insertion
INSERT INTO employees (first_name, last_name, email, salary)
VALUES ('Test', 'User', NULL, NULL);
-- Only works if columns allow NULL

-- NULL vs DEFAULT
-- NULL: 명시적으로 NULL 저장
-- DEFAULT: 기본값 사용 (기본값이 없으면 NULL)
INSERT INTO orders (customer_id, notes)
VALUES (104, NULL);  -- Explicitly NULL
```

### INSERT with Expressions (표현식을 사용한 삽입)

```sql
-- Using expressions in INSERT
INSERT INTO employees (first_name, last_name, email, hire_date, salary)
VALUES (
    'Mike',
    'Johnson',
    CONCAT(LOWER('Mike'), '.', LOWER('Johnson'), '@company.com'),
    CURDATE(),
    75000 * 1.1  -- 10% bonus
);

-- Using subquery for a value
INSERT INTO employees (first_name, last_name, email, department_id)
VALUES (
    'Sarah',
    'Lee',
    'sarah.lee@company.com',
    (SELECT department_id FROM departments WHERE department_name = 'Engineering')
);
```

### INSERT with AUTO_INCREMENT (AUTO_INCREMENT 삽입)

```sql
-- Let AUTO_INCREMENT generate the ID
INSERT INTO employees (first_name, last_name, email)
VALUES ('New', 'Employee', 'new@company.com');

-- Get the last inserted ID (MySQL)
SELECT LAST_INSERT_ID();  -- Returns the generated ID

-- Insert with specific ID (override AUTO_INCREMENT)
INSERT INTO employees (employee_id, first_name, last_name, email)
VALUES (1000, 'Special', 'User', 'special@company.com');
-- Next AUTO_INCREMENT will be 1001

-- Reset AUTO_INCREMENT
ALTER TABLE employees AUTO_INCREMENT = 1;
```

---

## 3. UPDATE Statement (수정문)

### Basic UPDATE Syntax (기본 UPDATE 구문)

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition;
```

### UPDATE Examples (UPDATE 예제)

```sql
-- Update single column
-- 단일 열 수정
UPDATE employees
SET salary = 80000
WHERE employee_id = 1;

-- Update multiple columns
-- 여러 열 수정
UPDATE employees
SET salary = 85000,
    department_id = 20,
    email = 'john.d@company.com'
WHERE employee_id = 1;

-- Update with expression
-- 표현식으로 수정
UPDATE employees
SET salary = salary * 1.10  -- 10% raise
WHERE department_id = 10;

-- Update all rows (DANGEROUS - no WHERE clause)
-- 모든 행 수정 (위험 - WHERE 절 없음)
UPDATE employees
SET status = 'Active';
-- WARNING: This updates ALL rows!
```

### UPDATE Flow Diagram (UPDATE 흐름 다이어그램)

```
UPDATE Statement
       |
       v
+------------------+
| Parse Query      |
+------------------+
       |
       v
+------------------+
| Find Rows        |
| (WHERE clause)   |
+------------------+
       |
       v
+------------------+
| Check Constraints|
| for New Values   |
+------------------+
       |
   +---+---+
   |       |
   v       v
Success  Failure
   |       |
   v       v
Update   Rollback
 Rows    Error
       |
       v
+------------------+
| Return Row Count |
+------------------+
```

### UPDATE with Conditions (조건부 UPDATE)

```sql
-- Update with multiple conditions
UPDATE employees
SET bonus = 5000
WHERE department_id = 10
  AND hire_date < '2023-01-01'
  AND performance_rating >= 4;

-- Update using IN
UPDATE products
SET discount = 0.15
WHERE category_id IN (1, 3, 5);

-- Update using BETWEEN
UPDATE orders
SET status = 'Archived'
WHERE order_date BETWEEN '2020-01-01' AND '2020-12-31';

-- Update using LIKE
UPDATE customers
SET email_verified = TRUE
WHERE email LIKE '%@gmail.com';
```

### UPDATE with CASE (CASE를 사용한 UPDATE)

```sql
-- Conditional update based on values
-- 값에 따른 조건부 수정
UPDATE employees
SET salary = CASE
    WHEN department_id = 10 THEN salary * 1.15  -- 15% raise
    WHEN department_id = 20 THEN salary * 1.10  -- 10% raise
    WHEN department_id = 30 THEN salary * 1.05  -- 5% raise
    ELSE salary * 1.03  -- 3% raise
END
WHERE hire_date < '2023-01-01';

-- Update status based on quantity
UPDATE products
SET status = CASE
    WHEN stock_quantity = 0 THEN 'Out of Stock'
    WHEN stock_quantity < 10 THEN 'Low Stock'
    WHEN stock_quantity < 50 THEN 'In Stock'
    ELSE 'Well Stocked'
END;
```

### UPDATE with Subquery (서브쿼리를 사용한 UPDATE)

```sql
-- Update using subquery in SET
UPDATE employees
SET department_id = (
    SELECT department_id
    FROM departments
    WHERE department_name = 'Marketing'
)
WHERE employee_id = 5;

-- Update using subquery in WHERE
UPDATE employees
SET salary = salary * 1.1
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location = 'New York'
);

-- Update with correlated subquery
UPDATE products p
SET average_rating = (
    SELECT AVG(rating)
    FROM reviews r
    WHERE r.product_id = p.product_id
);
```

### UPDATE with JOIN (JOIN을 사용한 UPDATE)

```sql
-- MySQL UPDATE with JOIN
UPDATE employees e
INNER JOIN departments d ON e.department_id = d.department_id
SET e.location = d.location
WHERE d.department_name = 'Engineering';

-- Update using multiple tables
UPDATE orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
SET o.priority = 'High',
    o.discount = 0.10
WHERE c.membership_level = 'Gold';

-- SQL Server syntax
UPDATE e
SET e.location = d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering';
```

### UPDATE Best Practices (UPDATE 모범 사례)

```sql
-- 1. Always use WHERE clause (unless intentional)
-- 항상 WHERE 절 사용 (의도적인 경우 제외)

-- BAD: Updates all rows!
UPDATE products SET price = 0;

-- GOOD: Specific update
UPDATE products SET price = 0 WHERE product_id = 100;

-- 2. Test with SELECT first
-- 먼저 SELECT로 테스트
SELECT * FROM employees WHERE department_id = 10;
-- Then update
UPDATE employees SET bonus = 1000 WHERE department_id = 10;

-- 3. Use LIMIT for safety (MySQL)
-- 안전을 위해 LIMIT 사용
UPDATE employees
SET status = 'Inactive'
WHERE last_login < '2023-01-01'
LIMIT 100;  -- Process in batches

-- 4. Use transactions for important updates
-- 중요한 수정에는 트랜잭션 사용
START TRANSACTION;
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
COMMIT;  -- or ROLLBACK if error
```

---

## 4. DELETE Statement (삭제문)

### Basic DELETE Syntax (기본 DELETE 구문)

```sql
DELETE FROM table_name
WHERE condition;
```

### DELETE Examples (DELETE 예제)

```sql
-- Delete single row
-- 단일 행 삭제
DELETE FROM employees
WHERE employee_id = 100;

-- Delete multiple rows with condition
-- 조건으로 여러 행 삭제
DELETE FROM orders
WHERE status = 'Cancelled'
  AND order_date < '2023-01-01';

-- Delete with IN clause
DELETE FROM products
WHERE product_id IN (101, 102, 103);

-- Delete all rows (DANGEROUS!)
-- 모든 행 삭제 (위험!)
DELETE FROM temp_table;
-- WARNING: This deletes ALL rows!
```

### DELETE Flow Diagram (DELETE 흐름 다이어그램)

```
DELETE Statement
       |
       v
+------------------+
| Parse Query      |
+------------------+
       |
       v
+------------------+
| Find Rows        |
| (WHERE clause)   |
+------------------+
       |
       v
+------------------+
| Check Foreign    |
| Key Constraints  |
+------------------+
       |
   +---+---+
   |       |
   v       v
 Can     Cannot
Delete   Delete
   |       |
   v       v
Remove   Error
 Rows    (FK violation)
       |
       v
+------------------+
| Fire DELETE      |
| Triggers         |
+------------------+
```

### DELETE with Subquery (서브쿼리를 사용한 DELETE)

```sql
-- Delete using subquery in WHERE
DELETE FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE status = 'Closed'
);

-- Delete orphan records
-- 고아 레코드 삭제
DELETE FROM order_items
WHERE order_id NOT IN (
    SELECT order_id FROM orders
);

-- Delete duplicates (keep lowest ID)
-- 중복 삭제 (가장 낮은 ID 유지)
DELETE e1 FROM employees e1
INNER JOIN employees e2
WHERE e1.email = e2.email
  AND e1.employee_id > e2.employee_id;
```

### DELETE with JOIN (JOIN을 사용한 DELETE)

```sql
-- MySQL DELETE with JOIN
DELETE e
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE d.status = 'Inactive';

-- Delete from multiple tables
DELETE o, oi
FROM orders o
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date < '2020-01-01';

-- SQL Server syntax
DELETE e
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE d.status = 'Inactive';
```

### DELETE with LIMIT (LIMIT을 사용한 DELETE)

```sql
-- Delete with LIMIT (MySQL)
-- 대량 삭제를 배치로 처리
DELETE FROM logs
WHERE created_at < '2023-01-01'
LIMIT 10000;  -- Delete 10,000 at a time

-- Loop to delete all in batches (pseudo-code)
-- 배치로 전체 삭제하는 루프 (의사 코드)
-- WHILE EXISTS (SELECT 1 FROM logs WHERE created_at < '2023-01-01')
-- BEGIN
--     DELETE FROM logs WHERE created_at < '2023-01-01' LIMIT 10000;
--     SLEEP(1);  -- Brief pause to reduce lock contention
-- END
```

### DELETE vs TRUNCATE (DELETE와 TRUNCATE 비교)

```
+------------------+------------------+
|     DELETE       |    TRUNCATE      |
+------------------+------------------+
| DML statement    | DDL statement    |
| Uses WHERE       | No WHERE clause  |
| Row-by-row       | All at once      |
| Can rollback     | Cannot rollback  |
| Slower           | Faster           |
| Triggers fire    | No triggers      |
| Keeps identity   | Resets identity  |
| Logs each row    | Minimal logging  |
+------------------+------------------+
```

```sql
-- DELETE: When you need control
DELETE FROM orders WHERE order_date < '2020-01-01';

-- TRUNCATE: When clearing entire table
TRUNCATE TABLE temp_results;

-- DELETE in transaction (can rollback)
START TRANSACTION;
DELETE FROM important_data WHERE status = 'test';
-- Oops, wrong condition!
ROLLBACK;  -- Data restored
```

### Soft Delete Pattern (소프트 삭제 패턴)

```sql
-- Instead of actually deleting, mark as deleted
-- 실제 삭제 대신 삭제됨으로 표시

-- Table with soft delete column
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    is_deleted BOOLEAN DEFAULT FALSE,
    deleted_at TIMESTAMP NULL
);

-- Soft delete
UPDATE users
SET is_deleted = TRUE,
    deleted_at = CURRENT_TIMESTAMP
WHERE user_id = 100;

-- Query active users
SELECT * FROM users WHERE is_deleted = FALSE;

-- Restore deleted user
UPDATE users
SET is_deleted = FALSE,
    deleted_at = NULL
WHERE user_id = 100;

-- Permanently delete (after retention period)
DELETE FROM users
WHERE is_deleted = TRUE
  AND deleted_at < DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY);
```

---

## 5. INSERT ... SELECT

### Basic INSERT ... SELECT Syntax (기본 구문)

```sql
INSERT INTO target_table (column1, column2, ...)
SELECT column1, column2, ...
FROM source_table
WHERE condition;
```

### INSERT ... SELECT Examples (예제)

```sql
-- Copy all data from one table to another
-- 한 테이블에서 다른 테이블로 모든 데이터 복사
INSERT INTO employees_backup
SELECT * FROM employees;

-- Copy with specific columns
-- 특정 열만 복사
INSERT INTO employee_names (first_name, last_name)
SELECT first_name, last_name
FROM employees
WHERE department_id = 10;

-- Copy with transformation
-- 변환과 함께 복사
INSERT INTO employee_summary (full_name, year_hired, annual_salary)
SELECT
    CONCAT(first_name, ' ', last_name),
    YEAR(hire_date),
    salary * 12
FROM employees
WHERE status = 'Active';
```

### INSERT ... SELECT Flow (INSERT ... SELECT 흐름)

```
+------------------+          +------------------+
|  Source Table    |          |  Target Table    |
+------------------+          +------------------+
| id | name | dept |          | emp_name | dept  |
|----|------|------|          |----------|-------|
| 1  | Kim  | IT   |          |          |       |
| 2  | Lee  | HR   |          |          |       |
| 3  | Park | IT   |   --->   |          |       |
+------------------+          +------------------+

INSERT INTO target_table (emp_name, dept)
SELECT name, dept FROM source_table WHERE dept = 'IT';

Result (결과):
+------------------+
|  Target Table    |
+------------------+
| emp_name | dept  |
|----------|-------|
| Kim      | IT    |
| Park     | IT    |
+------------------+
```

### INSERT ... SELECT with JOIN (JOIN과 함께)

```sql
-- Insert from joined tables
INSERT INTO order_report (order_id, customer_name, product_name, quantity)
SELECT
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2024-01-01';
```

### INSERT ... SELECT with Aggregation (집계와 함께)

```sql
-- Create summary table
-- 요약 테이블 생성
CREATE TABLE monthly_sales_summary (
    year_month CHAR(7) PRIMARY KEY,
    total_orders INT,
    total_revenue DECIMAL(15, 2),
    avg_order_value DECIMAL(10, 2)
);

-- Insert aggregated data
INSERT INTO monthly_sales_summary (year_month, total_orders, total_revenue, avg_order_value)
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS year_month,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue,
    AVG(total_amount) AS avg_order_value
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY DATE_FORMAT(order_date, '%Y-%m');
```

### Archive Pattern (아카이브 패턴)

```sql
-- Archive old orders
-- 오래된 주문 아카이브

-- 1. Create archive table (if not exists)
CREATE TABLE IF NOT EXISTS orders_archive LIKE orders;

-- 2. Copy old data to archive
INSERT INTO orders_archive
SELECT * FROM orders
WHERE order_date < '2023-01-01';

-- 3. Delete archived data from main table
DELETE FROM orders
WHERE order_date < '2023-01-01';

-- Alternative: Using transaction
START TRANSACTION;

INSERT INTO orders_archive
SELECT * FROM orders WHERE order_date < '2023-01-01';

DELETE FROM orders WHERE order_date < '2023-01-01';

COMMIT;
```

---

## 6. UPSERT Patterns

### What is UPSERT? (UPSERT란?)

**UPSERT** = **UP**date + in**SERT**

Insert a new row if it doesn't exist, or update if it does exist.
행이 존재하지 않으면 삽입하고, 존재하면 수정합니다.

```
UPSERT Decision Flow (UPSERT 결정 흐름)

        +------------------+
        | Row with same    |
        | key exists?      |
        +------------------+
               |
          +----+----+
          |         |
          v         v
         YES        NO
          |         |
          v         v
       UPDATE    INSERT
```

### MySQL: INSERT ... ON DUPLICATE KEY UPDATE

```sql
-- Basic syntax
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...)
ON DUPLICATE KEY UPDATE
    column1 = new_value1,
    column2 = new_value2;
```

```sql
-- Example: Product inventory
-- 예제: 제품 재고
CREATE TABLE product_inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    last_updated TIMESTAMP
);

-- Insert or update
INSERT INTO product_inventory (product_id, product_name, quantity, last_updated)
VALUES (101, 'Widget A', 50, CURRENT_TIMESTAMP)
ON DUPLICATE KEY UPDATE
    quantity = quantity + VALUES(quantity),
    last_updated = CURRENT_TIMESTAMP;

-- First execution: Inserts new row
-- 첫 번째 실행: 새 행 삽입
-- product_id=101, quantity=50

-- Second execution: Updates existing row
-- 두 번째 실행: 기존 행 수정
-- product_id=101, quantity=100 (50+50)
```

### MySQL: INSERT ... ON DUPLICATE KEY UPDATE (Advanced)

```sql
-- Using VALUES() function to reference inserted values
-- VALUES() 함수로 삽입하려던 값 참조
INSERT INTO sales_summary (product_id, month, quantity, revenue)
VALUES (101, '2024-01', 100, 5000.00)
ON DUPLICATE KEY UPDATE
    quantity = quantity + VALUES(quantity),
    revenue = revenue + VALUES(revenue);

-- Conditional update
INSERT INTO user_stats (user_id, login_count, last_login)
VALUES (1, 1, CURRENT_TIMESTAMP)
ON DUPLICATE KEY UPDATE
    login_count = login_count + 1,
    last_login = CURRENT_TIMESTAMP;

-- Insert multiple rows with UPSERT
INSERT INTO inventory (sku, quantity)
VALUES
    ('SKU001', 10),
    ('SKU002', 20),
    ('SKU003', 30)
ON DUPLICATE KEY UPDATE
    quantity = quantity + VALUES(quantity);
```

### MySQL: REPLACE Statement

```sql
-- REPLACE deletes existing row and inserts new one
-- REPLACE는 기존 행을 삭제하고 새 행을 삽입
REPLACE INTO users (user_id, username, email)
VALUES (1, 'john_updated', 'john.new@email.com');

-- Warning: REPLACE deletes the entire row first
-- 주의: REPLACE는 먼저 전체 행을 삭제합니다
-- This means:
-- - AUTO_INCREMENT values may change
-- - Foreign key CASCADE DELETE may trigger
-- - Triggers fire for DELETE then INSERT

-- When to use REPLACE vs ON DUPLICATE KEY UPDATE:
-- REPLACE: When you want complete row replacement
-- ON DUPLICATE KEY UPDATE: When you want partial update
```

### SQL Server: MERGE Statement

```sql
-- SQL Server MERGE (Standard SQL)
MERGE INTO target_table AS target
USING source_table AS source
ON target.key_column = source.key_column
WHEN MATCHED THEN
    UPDATE SET
        target.column1 = source.column1,
        target.column2 = source.column2
WHEN NOT MATCHED THEN
    INSERT (key_column, column1, column2)
    VALUES (source.key_column, source.column1, source.column2);
```

```sql
-- Practical MERGE example
-- MERGE 실전 예제
MERGE INTO product_inventory AS target
USING (SELECT 101 AS product_id, 'Widget A' AS name, 50 AS qty) AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET
        target.quantity = target.quantity + source.qty,
        target.last_updated = GETDATE()
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, quantity, last_updated)
    VALUES (source.product_id, source.name, source.qty, GETDATE());
```

### PostgreSQL: INSERT ... ON CONFLICT

```sql
-- PostgreSQL UPSERT syntax
INSERT INTO users (user_id, username, email, login_count)
VALUES (1, 'john', 'john@email.com', 1)
ON CONFLICT (user_id)
DO UPDATE SET
    login_count = users.login_count + 1,
    last_login = CURRENT_TIMESTAMP;

-- ON CONFLICT DO NOTHING (ignore duplicates)
INSERT INTO users (user_id, username, email)
VALUES (1, 'john', 'john@email.com')
ON CONFLICT (user_id) DO NOTHING;

-- Conflict on unique constraint
INSERT INTO products (sku, product_name, price)
VALUES ('ABC123', 'New Product', 29.99)
ON CONFLICT ON CONSTRAINT products_sku_key
DO UPDATE SET
    price = EXCLUDED.price;  -- EXCLUDED = the proposed insertion row
```

### UPSERT Comparison Table (UPSERT 비교표)

```
+------------------+------------------------------------+
| Database         | UPSERT Syntax                      |
+------------------+------------------------------------+
| MySQL            | INSERT ... ON DUPLICATE KEY UPDATE |
|                  | REPLACE INTO                       |
+------------------+------------------------------------+
| SQL Server       | MERGE ... WHEN MATCHED/NOT MATCHED |
+------------------+------------------------------------+
| PostgreSQL       | INSERT ... ON CONFLICT DO UPDATE   |
+------------------+------------------------------------+
| Oracle           | MERGE ... WHEN MATCHED/NOT MATCHED |
+------------------+------------------------------------+
| SQLite           | INSERT OR REPLACE                  |
|                  | INSERT ... ON CONFLICT DO UPDATE   |
+------------------+------------------------------------+
```

### UPSERT Best Practices (UPSERT 모범 사례)

```sql
-- 1. Always have proper unique constraint/primary key
-- 항상 적절한 유니크 제약조건/기본키 필요
CREATE TABLE user_sessions (
    session_id VARCHAR(100) PRIMARY KEY,
    user_id INT NOT NULL,
    last_activity TIMESTAMP,
    data JSON
);

-- 2. Use explicit column lists
-- 명시적 열 목록 사용
INSERT INTO user_sessions (session_id, user_id, last_activity)
VALUES ('abc123', 1, CURRENT_TIMESTAMP)
ON DUPLICATE KEY UPDATE
    last_activity = CURRENT_TIMESTAMP;

-- 3. Consider using transactions for complex upserts
-- 복잡한 upsert에는 트랜잭션 사용 고려
START TRANSACTION;
-- Multiple related upsert operations
COMMIT;

-- 4. Be careful with AUTO_INCREMENT and REPLACE
-- AUTO_INCREMENT와 REPLACE 사용 시 주의
-- Prefer ON DUPLICATE KEY UPDATE to preserve ID
```

---

## 7. Summary (요약)

### DML Commands Quick Reference (DML 명령어 빠른 참조)

| Command | Purpose | Basic Syntax |
|---------|---------|--------------|
| INSERT | Add rows | INSERT INTO table VALUES(...) |
| UPDATE | Modify rows | UPDATE table SET col=val WHERE... |
| DELETE | Remove rows | DELETE FROM table WHERE... |
| INSERT...SELECT | Copy data | INSERT INTO t1 SELECT...FROM t2 |
| UPSERT | Insert or Update | Varies by database |

### INSERT Patterns (INSERT 패턴)

```sql
-- Single row
INSERT INTO table (cols) VALUES (vals);

-- Multiple rows
INSERT INTO table (cols) VALUES (vals1), (vals2), (vals3);

-- From SELECT
INSERT INTO table (cols) SELECT cols FROM other_table;
```

### UPDATE Patterns (UPDATE 패턴)

```sql
-- Basic
UPDATE table SET col = val WHERE condition;

-- With CASE
UPDATE table SET col = CASE WHEN... THEN... END;

-- With JOIN
UPDATE t1 JOIN t2 ON... SET t1.col = t2.col;
```

### DELETE Patterns (DELETE 패턴)

```sql
-- Basic
DELETE FROM table WHERE condition;

-- With subquery
DELETE FROM t1 WHERE col IN (SELECT col FROM t2);

-- With JOIN (MySQL)
DELETE t1 FROM t1 JOIN t2 ON... WHERE...;
```

### UPSERT by Database (데이터베이스별 UPSERT)

```sql
-- MySQL
INSERT INTO table (...) VALUES (...) ON DUPLICATE KEY UPDATE...;

-- PostgreSQL
INSERT INTO table (...) VALUES (...) ON CONFLICT DO UPDATE...;

-- SQL Server
MERGE INTO table USING... WHEN MATCHED THEN UPDATE... WHEN NOT MATCHED THEN INSERT...;
```

### Key Points (핵심 포인트)

1. **Always use WHERE clause** in UPDATE and DELETE (unless intentional)
2. **Test with SELECT first** before UPDATE/DELETE
3. **Use transactions** for related operations
4. **INSERT...SELECT** is efficient for bulk operations
5. **UPSERT** simplifies insert-or-update logic
6. **Soft delete** preserves data for recovery

---

*End of Chapter 10*
