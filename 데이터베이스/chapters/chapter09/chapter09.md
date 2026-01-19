# Chapter 09. DDL - Data Definition Language (데이터 정의 언어)

## Table of Contents (목차)
1. [DDL Overview](#1-ddl-overview-개요)
2. [CREATE TABLE](#2-create-table-테이블-생성)
3. [Data Types](#3-data-types-데이터-타입)
4. [Constraints](#4-constraints-제약조건)
5. [ALTER TABLE](#5-alter-table-테이블-수정)
6. [DROP and TRUNCATE](#6-drop-and-truncate)
7. [Summary](#7-summary-요약)

---

## 1. DDL Overview (개요)

### What is DDL? (DDL이란?)

**DDL (Data Definition Language)** is used to define and modify the structure of database objects such as tables, indexes, and views.

**DDL (데이터 정의 언어)**은 테이블, 인덱스, 뷰와 같은 데이터베이스 객체의 구조를 정의하고 수정하는 데 사용됩니다.

```
+------------------+
|       DDL        |
+------------------+
         |
    +----+----+----+----+
    |    |    |    |    |
    v    v    v    v    v
CREATE ALTER DROP TRUNCATE RENAME
  |      |     |      |       |
  v      v     v      v       v
Create Modify Remove Clear  Rename
Object Object Object  Data   Object
```

### DDL Commands (DDL 명령어)

| Command | Description (설명) | Example |
|---------|-------------------|---------|
| CREATE | Create new object (새 객체 생성) | CREATE TABLE |
| ALTER | Modify existing object (기존 객체 수정) | ALTER TABLE |
| DROP | Delete object completely (객체 완전 삭제) | DROP TABLE |
| TRUNCATE | Remove all data (모든 데이터 제거) | TRUNCATE TABLE |
| RENAME | Rename object (객체 이름 변경) | RENAME TABLE |

### DDL Characteristics (DDL 특징)

1. **Auto-commit**: DDL statements are automatically committed
   - **자동 커밋**: DDL 문은 자동으로 커밋됨
2. **Schema changes**: Modifies database structure, not data
   - **스키마 변경**: 데이터가 아닌 데이터베이스 구조 수정
3. **Immediate effect**: Changes take effect immediately
   - **즉시 적용**: 변경사항이 즉시 적용됨

---

## 2. CREATE TABLE (테이블 생성)

### Basic Syntax (기본 구문)

```sql
CREATE TABLE table_name (
    column1 datatype [constraints],
    column2 datatype [constraints],
    ...
    [table_constraints]
);
```

### Simple Example (간단한 예제)

```sql
-- Create a basic employees table
-- 기본 직원 테이블 생성
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE,
    salary DECIMAL(10, 2),
    department_id INT
);
```

### Complete Example with All Features (모든 기능을 포함한 완전한 예제)

```sql
-- Create department table first (referenced by employees)
-- 먼저 부서 테이블 생성 (직원 테이블에서 참조)
CREATE TABLE departments (
    department_id INT AUTO_INCREMENT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE,
    manager_id INT,
    location VARCHAR(100) DEFAULT 'Head Office',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create employees table with foreign key
-- 외래키가 있는 직원 테이블 생성
CREATE TABLE employees (
    employee_id INT AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    hire_date DATE NOT NULL,
    job_title VARCHAR(50),
    salary DECIMAL(10, 2) CHECK (salary > 0),
    commission_pct DECIMAL(3, 2),
    manager_id INT,
    department_id INT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- Primary Key
    PRIMARY KEY (employee_id),

    -- Foreign Key
    CONSTRAINT fk_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE,

    -- Check constraint
    CONSTRAINT chk_commission
        CHECK (commission_pct IS NULL OR (commission_pct >= 0 AND commission_pct <= 1))
);
```

### CREATE TABLE Structure Diagram (CREATE TABLE 구조 다이어그램)

```
CREATE TABLE employees
+------------------------------------------------------------------+
| Column Definition (열 정의)                                        |
+------------------------------------------------------------------+
| employee_id    | INT           | AUTO_INCREMENT | PRIMARY KEY     |
| first_name     | VARCHAR(50)   | NOT NULL       |                 |
| email          | VARCHAR(100)  | NOT NULL       | UNIQUE          |
| salary         | DECIMAL(10,2) |                | CHECK(>0)       |
| department_id  | INT           |                | FOREIGN KEY     |
+------------------------------------------------------------------+
| Table Constraints (테이블 제약조건)                                 |
+------------------------------------------------------------------+
| PRIMARY KEY (employee_id)                                         |
| FOREIGN KEY (department_id) REFERENCES departments(department_id) |
+------------------------------------------------------------------+
```

### CREATE TABLE ... AS SELECT (CTAS)

```sql
-- Create table from query result
-- 쿼리 결과로 테이블 생성
CREATE TABLE high_salary_employees AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > 100000;

-- Create empty table with same structure
-- 같은 구조의 빈 테이블 생성
CREATE TABLE employees_backup AS
SELECT * FROM employees
WHERE 1 = 0;  -- No rows copied

-- MySQL: CREATE TABLE LIKE
CREATE TABLE employees_archive LIKE employees;
```

### Temporary Tables (임시 테이블)

```sql
-- Create temporary table (session-scoped)
-- 임시 테이블 생성 (세션 범위)
CREATE TEMPORARY TABLE temp_results (
    id INT AUTO_INCREMENT PRIMARY KEY,
    result_value DECIMAL(10, 2),
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Temporary table with query
CREATE TEMPORARY TABLE temp_high_sales AS
SELECT product_id, SUM(quantity) as total_sold
FROM sales
WHERE sale_date >= '2024-01-01'
GROUP BY product_id
HAVING SUM(quantity) > 1000;
```

---

## 3. Data Types (데이터 타입)

### Numeric Types (숫자 타입)

```
+------------------+------------------+---------------------------+
| Type             | Size             | Range/Description          |
+------------------+------------------+---------------------------+
| TINYINT          | 1 byte           | -128 to 127               |
| SMALLINT         | 2 bytes          | -32,768 to 32,767         |
| MEDIUMINT        | 3 bytes          | -8,388,608 to 8,388,607   |
| INT / INTEGER    | 4 bytes          | -2.1B to 2.1B             |
| BIGINT           | 8 bytes          | Very large integers       |
+------------------+------------------+---------------------------+
| DECIMAL(p,s)     | Variable         | Exact numeric             |
| NUMERIC(p,s)     | Variable         | Same as DECIMAL           |
| FLOAT            | 4 bytes          | Approximate (7 digits)    |
| DOUBLE           | 8 bytes          | Approximate (15 digits)   |
+------------------+------------------+---------------------------+
```

```sql
-- Numeric type examples
-- 숫자 타입 예제
CREATE TABLE financial_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    quantity INT,                      -- Whole numbers
    unit_price DECIMAL(10, 2),         -- Exact decimal (currency)
    tax_rate DECIMAL(5, 4),            -- 0.0000 to 9.9999
    large_number BIGINT,               -- Very large integers
    measurement DOUBLE                 -- Scientific calculations
);
```

### String Types (문자열 타입)

```
+------------------+------------------+---------------------------+
| Type             | Max Length       | Description               |
+------------------+------------------+---------------------------+
| CHAR(n)          | 255 chars        | Fixed-length string       |
| VARCHAR(n)       | 65,535 chars     | Variable-length string    |
| TEXT             | 65,535 chars     | Long text                 |
| MEDIUMTEXT       | 16 MB            | Medium-length text        |
| LONGTEXT         | 4 GB             | Very long text            |
+------------------+------------------+---------------------------+
| BINARY(n)        | 255 bytes        | Fixed-length binary       |
| VARBINARY(n)     | 65,535 bytes     | Variable-length binary    |
| BLOB             | 65,535 bytes     | Binary Large Object       |
+------------------+------------------+---------------------------+
```

```sql
-- String type examples
-- 문자열 타입 예제
CREATE TABLE content_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    status_code CHAR(3),               -- Fixed: 'ACT', 'INA'
    username VARCHAR(50),              -- Variable: up to 50 chars
    description TEXT,                  -- Long descriptions
    article_body LONGTEXT,             -- Full articles
    profile_image BLOB                 -- Binary image data
);
```

### CHAR vs VARCHAR (CHAR와 VARCHAR 비교)

```
CHAR(10) - Fixed Length (고정 길이)
+-------------------------------------------+
| 'Hello'     | 'Hello     '  (padded)      |
| Storage: Always 10 bytes                   |
| Faster for fixed-size data                 |
+-------------------------------------------+

VARCHAR(10) - Variable Length (가변 길이)
+-------------------------------------------+
| 'Hello'     | 'Hello' + length byte        |
| Storage: 5 + 1 = 6 bytes                   |
| More efficient for varying sizes           |
+-------------------------------------------+
```

```sql
-- When to use CHAR vs VARCHAR
-- CHAR: Fixed-length codes (ISO country codes, status codes)
-- VARCHAR: Names, addresses, variable-length data

CREATE TABLE locations (
    country_code CHAR(2),      -- 'US', 'KR', 'JP'
    zip_code CHAR(5),          -- '12345'
    city_name VARCHAR(100),    -- Variable length
    address VARCHAR(255)       -- Variable length
);
```

### Date and Time Types (날짜 및 시간 타입)

```
+------------------+------------------+---------------------------+
| Type             | Format           | Range                      |
+------------------+------------------+---------------------------+
| DATE             | YYYY-MM-DD       | 1000-01-01 to 9999-12-31  |
| TIME             | HH:MM:SS         | -838:59:59 to 838:59:59   |
| DATETIME         | YYYY-MM-DD HH:MM:SS | 1000 to 9999           |
| TIMESTAMP        | YYYY-MM-DD HH:MM:SS | 1970 to 2038           |
| YEAR             | YYYY             | 1901 to 2155              |
+------------------+------------------+---------------------------+
```

```sql
-- Date/Time type examples
-- 날짜/시간 타입 예제
CREATE TABLE events (
    event_id INT AUTO_INCREMENT PRIMARY KEY,
    event_date DATE,                           -- Just date
    event_time TIME,                           -- Just time
    event_datetime DATETIME,                   -- Date and time
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Auto-set
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
               ON UPDATE CURRENT_TIMESTAMP,    -- Auto-update
    year_only YEAR                             -- Just year
);

-- DATETIME vs TIMESTAMP
-- DATETIME: Stored as-is, no timezone conversion
-- TIMESTAMP: Converts to/from UTC, good for global apps
```

### Other Types (기타 타입)

```sql
-- Boolean (stored as TINYINT(1) in MySQL)
-- 불리언 (MySQL에서는 TINYINT(1)로 저장)
CREATE TABLE user_settings (
    user_id INT PRIMARY KEY,
    is_active BOOLEAN DEFAULT TRUE,
    email_notifications BOOLEAN DEFAULT FALSE
);

-- ENUM: Predefined list of values
-- ENUM: 미리 정의된 값 목록
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    status ENUM('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled')
           DEFAULT 'Pending'
);

-- SET: Multiple values from predefined list
-- SET: 미리 정의된 목록에서 여러 값 선택
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    tags SET('New', 'Sale', 'Popular', 'Limited')
);

-- JSON (MySQL 5.7+)
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    preferences JSON
);
```

### Choosing Data Types (데이터 타입 선택 가이드)

```
+------------------+----------------------------------------+
| Scenario         | Recommended Type                       |
+------------------+----------------------------------------+
| Primary Key      | INT or BIGINT (AUTO_INCREMENT)         |
| Names            | VARCHAR(50-100)                        |
| Email            | VARCHAR(255)                           |
| Phone            | VARCHAR(20)                            |
| Price/Currency   | DECIMAL(10, 2)                         |
| Percentage       | DECIMAL(5, 2) or DECIMAL(5, 4)         |
| True/False       | BOOLEAN / TINYINT(1)                   |
| Birth Date       | DATE                                   |
| Created Time     | TIMESTAMP / DATETIME                   |
| Long Text        | TEXT or MEDIUMTEXT                     |
| Fixed Codes      | CHAR(n) or ENUM                        |
+------------------+----------------------------------------+
```

---

## 4. Constraints (제약조건)

### Constraints Overview (제약조건 개요)

Constraints enforce rules on data to maintain integrity.
제약조건은 데이터 무결성을 유지하기 위해 규칙을 적용합니다.

```
+------------------+----------------------------------------+
| Constraint       | Purpose (목적)                          |
+------------------+----------------------------------------+
| PRIMARY KEY      | Unique identifier (고유 식별자)          |
| FOREIGN KEY      | Referential integrity (참조 무결성)      |
| UNIQUE           | No duplicate values (중복 불가)          |
| NOT NULL         | Value required (값 필수)                 |
| CHECK            | Custom validation (사용자 정의 검증)      |
| DEFAULT          | Default value (기본값)                   |
+------------------+----------------------------------------+
```

### PRIMARY KEY (기본키)

Uniquely identifies each row in a table.
테이블의 각 행을 고유하게 식별합니다.

```sql
-- Column-level primary key (열 수준 기본키)
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

-- Table-level primary key (테이블 수준 기본키)
CREATE TABLE products (
    product_id INT,
    product_name VARCHAR(100),
    PRIMARY KEY (product_id)
);

-- Auto-increment primary key (자동 증가 기본키)
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100)
);

-- Composite primary key (복합 기본키)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

### FOREIGN KEY (외래키)

```
+----------------+          +------------------+
| departments    |          | employees        |
+----------------+          +------------------+
| department_id  |<---------| department_id    |
| (PK)           |    FK    | (FK)             |
+----------------+          +------------------+
      Parent                      Child
    (Referenced)                (Referencing)
```

```sql
-- Foreign key with referential actions
-- 참조 동작이 있는 외래키
CREATE TABLE employees (
    employee_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT,

    CONSTRAINT fk_emp_dept
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE SET NULL     -- Parent deleted: set NULL
        ON UPDATE CASCADE      -- Parent updated: cascade change
);
```

### Foreign Key Referential Actions (외래키 참조 동작)

| Action | Description (설명) |
|--------|-------------------|
| CASCADE | Propagate changes to child (자식에 변경 전파) |
| SET NULL | Set child FK to NULL (자식 FK를 NULL로 설정) |
| SET DEFAULT | Set child FK to default (자식 FK를 기본값으로 설정) |
| RESTRICT | Block parent change (부모 변경 차단) |
| NO ACTION | Same as RESTRICT (RESTRICT와 동일) |

```sql
-- Different referential actions
-- ON DELETE CASCADE: Delete child rows when parent deleted
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- ON DELETE SET NULL: Set FK to NULL when parent deleted
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
        ON DELETE SET NULL
);

-- ON DELETE RESTRICT: Prevent deletion if children exist
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
        ON DELETE RESTRICT
);
```

### UNIQUE Constraint (유일성 제약조건)

```sql
-- Single column UNIQUE
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20) UNIQUE
);

-- Composite UNIQUE (multiple columns)
-- 복합 유일성 (여러 열)
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    UNIQUE (student_id, course_id, semester)  -- Same student can't enroll twice in same course/semester
);

-- Named UNIQUE constraint
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    sku VARCHAR(50),
    CONSTRAINT uq_product_sku UNIQUE (sku)
);
```

### NOT NULL Constraint (NOT NULL 제약조건)

```sql
-- NOT NULL prevents NULL values
-- NOT NULL은 NULL 값을 방지합니다
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,      -- Required
    last_name VARCHAR(50) NOT NULL,       -- Required
    email VARCHAR(255) NOT NULL UNIQUE,   -- Required and unique
    phone VARCHAR(20),                    -- Optional (allows NULL)
    address TEXT                          -- Optional
);

-- Combining NOT NULL with DEFAULT
CREATE TABLE articles (
    article_id INT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'Draft',
    view_count INT NOT NULL DEFAULT 0
);
```

### CHECK Constraint (CHECK 제약조건)

```sql
-- MySQL 8.0.16+ supports CHECK constraints
-- MySQL 8.0.16+ 버전에서 CHECK 제약조건 지원

-- Column-level CHECK
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    salary DECIMAL(10, 2) CHECK (salary > 0),
    age INT CHECK (age >= 18 AND age <= 120)
);

-- Table-level CHECK with name
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    regular_price DECIMAL(10, 2),
    sale_price DECIMAL(10, 2),

    CONSTRAINT chk_sale_price
        CHECK (sale_price IS NULL OR sale_price < regular_price),
    CONSTRAINT chk_positive_price
        CHECK (regular_price > 0)
);

-- Complex CHECK constraints
CREATE TABLE events (
    event_id INT PRIMARY KEY,
    start_date DATE,
    end_date DATE,
    max_participants INT,
    current_participants INT DEFAULT 0,

    CONSTRAINT chk_dates CHECK (end_date >= start_date),
    CONSTRAINT chk_participants CHECK (current_participants <= max_participants)
);
```

### DEFAULT Constraint (DEFAULT 제약조건)

```sql
-- Various DEFAULT values
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    role VARCHAR(20) DEFAULT 'user',
    is_active BOOLEAN DEFAULT TRUE,
    credits INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- INSERT with defaults
INSERT INTO users (username) VALUES ('john_doe');
-- role='user', is_active=TRUE, credits=0, created_at=now
```

### Constraints Summary Diagram (제약조건 요약 다이어그램)

```
+------------------------------------------------------------------+
|                        employees Table                            |
+------------------------------------------------------------------+
| employee_id  | INT         | PRIMARY KEY, AUTO_INCREMENT         |
| first_name   | VARCHAR(50) | NOT NULL                             |
| email        | VARCHAR(255)| NOT NULL, UNIQUE                     |
| salary       | DECIMAL(10,2)| CHECK (salary > 0)                  |
| dept_id      | INT         | FOREIGN KEY -> departments           |
| status       | VARCHAR(20) | DEFAULT 'Active'                     |
+------------------------------------------------------------------+

Constraint Types Applied (적용된 제약조건 유형):
+-------------------+------------------------------------------+
| PRIMARY KEY       | employee_id - unique identifier          |
| NOT NULL          | first_name, email - required fields      |
| UNIQUE            | email - no duplicates                    |
| CHECK             | salary > 0 - validation rule             |
| FOREIGN KEY       | dept_id -> departments - referential     |
| DEFAULT           | status = 'Active' - default value        |
+-------------------+------------------------------------------+
```

---

## 5. ALTER TABLE (테이블 수정)

### ADD Column (열 추가)

```sql
-- Add single column (단일 열 추가)
ALTER TABLE employees
ADD middle_name VARCHAR(50);

-- Add column with constraints (제약조건과 함께 열 추가)
ALTER TABLE employees
ADD phone VARCHAR(20) NOT NULL DEFAULT 'N/A';

-- Add column at specific position (MySQL)
-- 특정 위치에 열 추가 (MySQL)
ALTER TABLE employees
ADD birth_date DATE AFTER last_name;

ALTER TABLE employees
ADD employee_code VARCHAR(10) FIRST;

-- Add multiple columns (여러 열 추가)
ALTER TABLE employees
ADD COLUMN address VARCHAR(255),
ADD COLUMN city VARCHAR(100),
ADD COLUMN postal_code VARCHAR(20);
```

### DROP Column (열 삭제)

```sql
-- Drop single column (단일 열 삭제)
ALTER TABLE employees
DROP COLUMN middle_name;

-- Drop multiple columns (MySQL 8.0+)
-- 여러 열 삭제
ALTER TABLE employees
DROP COLUMN temp_field1,
DROP COLUMN temp_field2;
```

### MODIFY / ALTER Column (열 수정)

```sql
-- Change data type (데이터 타입 변경)
ALTER TABLE employees
MODIFY COLUMN phone VARCHAR(30);

-- Add NOT NULL constraint (NOT NULL 제약조건 추가)
ALTER TABLE employees
MODIFY COLUMN phone VARCHAR(30) NOT NULL;

-- Remove NOT NULL (allow NULL) (NOT NULL 제거)
ALTER TABLE employees
MODIFY COLUMN phone VARCHAR(30) NULL;

-- Change default value (기본값 변경)
ALTER TABLE employees
ALTER COLUMN status SET DEFAULT 'Active';

-- Remove default value (기본값 제거)
ALTER TABLE employees
ALTER COLUMN status DROP DEFAULT;

-- SQL Server syntax
ALTER TABLE employees
ALTER COLUMN phone VARCHAR(30) NOT NULL;
```

### RENAME Column (열 이름 변경)

```sql
-- MySQL
ALTER TABLE employees
RENAME COLUMN phone TO phone_number;

-- Alternative MySQL syntax
ALTER TABLE employees
CHANGE phone phone_number VARCHAR(30);

-- SQL Server
EXEC sp_rename 'employees.phone', 'phone_number', 'COLUMN';

-- PostgreSQL
ALTER TABLE employees
RENAME COLUMN phone TO phone_number;
```

### ADD / DROP Constraints (제약조건 추가/삭제)

```sql
-- Add PRIMARY KEY
ALTER TABLE products
ADD PRIMARY KEY (product_id);

-- Drop PRIMARY KEY
ALTER TABLE products
DROP PRIMARY KEY;

-- Add FOREIGN KEY
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept
FOREIGN KEY (department_id) REFERENCES departments(department_id);

-- Drop FOREIGN KEY
ALTER TABLE employees
DROP FOREIGN KEY fk_emp_dept;

-- Add UNIQUE constraint
ALTER TABLE users
ADD CONSTRAINT uq_email UNIQUE (email);

-- Drop UNIQUE constraint
ALTER TABLE users
DROP INDEX uq_email;

-- Add CHECK constraint (MySQL 8.0.16+)
ALTER TABLE products
ADD CONSTRAINT chk_price CHECK (price > 0);

-- Drop CHECK constraint
ALTER TABLE products
DROP CHECK chk_price;
```

### RENAME Table (테이블 이름 변경)

```sql
-- MySQL
RENAME TABLE old_name TO new_name;
-- or
ALTER TABLE old_name RENAME TO new_name;

-- Rename multiple tables
RENAME TABLE
    employees TO staff,
    departments TO divisions;

-- SQL Server
EXEC sp_rename 'old_name', 'new_name';

-- PostgreSQL
ALTER TABLE old_name RENAME TO new_name;
```

### ALTER TABLE Examples (ALTER TABLE 예제 모음)

```sql
-- Practical example: Evolving a table
-- 실전 예제: 테이블 진화시키기

-- Initial table
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Add columns over time
ALTER TABLE customers
ADD email VARCHAR(255) UNIQUE,
ADD phone VARCHAR(20),
ADD address TEXT,
ADD created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

-- Modify column for larger data
ALTER TABLE customers
MODIFY name VARCHAR(200) NOT NULL;

-- Add foreign key to new table
CREATE TABLE customer_types (
    type_id INT PRIMARY KEY,
    type_name VARCHAR(50)
);

ALTER TABLE customers
ADD customer_type_id INT,
ADD CONSTRAINT fk_customer_type
    FOREIGN KEY (customer_type_id)
    REFERENCES customer_types(type_id);

-- Rename for clarity
ALTER TABLE customers
RENAME COLUMN name TO full_name;
```

---

## 6. DROP and TRUNCATE

### DROP TABLE (테이블 삭제)

```sql
-- Drop single table (단일 테이블 삭제)
DROP TABLE table_name;

-- Drop if exists (exists일 때만 삭제)
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables (여러 테이블 삭제)
DROP TABLE IF EXISTS table1, table2, table3;

-- Drop with dependencies (MySQL)
-- 종속성 무시하고 삭제
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE parent_table;
SET FOREIGN_KEY_CHECKS = 1;
```

### TRUNCATE TABLE (테이블 비우기)

```sql
-- Remove all rows, keep structure
-- 모든 행 삭제, 구조 유지
TRUNCATE TABLE table_name;

-- MySQL syntax
TRUNCATE TABLE orders;
```

### DELETE vs TRUNCATE vs DROP (비교)

```
+------------------+------------------+------------------+
|     DELETE       |    TRUNCATE      |      DROP        |
+------------------+------------------+------------------+
| DML command      | DDL command      | DDL command      |
| Can use WHERE    | No WHERE clause  | Removes table    |
| Row by row       | All at once      | All at once      |
| Slower           | Faster           | Fastest          |
| Can rollback     | Cannot rollback  | Cannot rollback  |
| Triggers fire    | No triggers      | No triggers      |
| Keeps identity   | Resets identity  | Table gone       |
| Logs each row    | Minimal logging  | Minimal logging  |
+------------------+------------------+------------------+
```

```sql
-- DELETE: Remove specific rows (특정 행 삭제)
DELETE FROM orders WHERE order_date < '2020-01-01';

-- DELETE all (can rollback in transaction)
-- 모든 행 삭제 (트랜잭션에서 롤백 가능)
START TRANSACTION;
DELETE FROM orders;
-- ROLLBACK;  -- Can undo
COMMIT;

-- TRUNCATE: Remove all rows efficiently (효율적으로 모든 행 삭제)
TRUNCATE TABLE orders;
-- Cannot rollback, resets AUTO_INCREMENT

-- DROP: Remove table entirely (테이블 완전 삭제)
DROP TABLE orders;
-- Table no longer exists
```

### When to Use Each (각각 언제 사용할까)

```
+------------------+----------------------------------------+
| Scenario         | Recommended Command                    |
+------------------+----------------------------------------+
| Delete specific  | DELETE WHERE condition                 |
| rows             |                                        |
+------------------+----------------------------------------+
| Delete all rows, | TRUNCATE TABLE                         |
| keep structure   | (fast, resets identity)                |
+------------------+----------------------------------------+
| Archive table    | CREATE new AS SELECT, then TRUNCATE    |
| before clearing  |                                        |
+------------------+----------------------------------------+
| Remove table     | DROP TABLE                             |
| completely       |                                        |
+------------------+----------------------------------------+
| Delete in        | DELETE (allows ROLLBACK)               |
| transaction      |                                        |
+------------------+----------------------------------------+
```

### Practical Example (실전 예제)

```sql
-- Scenario: Archive and clear old orders
-- 시나리오: 오래된 주문 보관 및 정리

-- 1. Create archive table
CREATE TABLE orders_archive LIKE orders;

-- 2. Copy old data to archive
INSERT INTO orders_archive
SELECT * FROM orders WHERE order_date < '2022-01-01';

-- 3. Delete archived data from main table
DELETE FROM orders WHERE order_date < '2022-01-01';

-- 4. If needed to completely reset a table:
-- CREATE TABLE orders_backup AS SELECT * FROM orders;
-- TRUNCATE TABLE orders;
```

---

## 7. Summary (요약)

### DDL Commands Quick Reference (DDL 명령어 빠른 참조)

| Command | Purpose | Syntax |
|---------|---------|--------|
| CREATE TABLE | Create new table | CREATE TABLE name (columns...) |
| ALTER TABLE ADD | Add column | ALTER TABLE name ADD column type |
| ALTER TABLE DROP | Drop column | ALTER TABLE name DROP COLUMN col |
| ALTER TABLE MODIFY | Change column | ALTER TABLE name MODIFY col type |
| DROP TABLE | Delete table | DROP TABLE [IF EXISTS] name |
| TRUNCATE TABLE | Empty table | TRUNCATE TABLE name |

### Data Type Selection Guide (데이터 타입 선택 가이드)

```
Numbers:
- INT for IDs and counts
- DECIMAL(p,s) for money
- BIGINT for very large numbers

Strings:
- VARCHAR(n) for variable text
- CHAR(n) for fixed-length codes
- TEXT for long content

Dates:
- DATE for dates only
- DATETIME for date and time
- TIMESTAMP for auto-tracking
```

### Constraints Best Practices (제약조건 모범 사례)

1. Always define PRIMARY KEY
2. Use FOREIGN KEY for relationships
3. Apply NOT NULL to required fields
4. Use CHECK for validation rules
5. Set DEFAULT for common values
6. Use UNIQUE for business identifiers

### Key Points (핵심 포인트)

1. **DDL is auto-committed** - changes are permanent immediately
2. **Choose appropriate data types** - affects storage and performance
3. **Use constraints** - enforce data integrity at database level
4. **ALTER TABLE carefully** - plan schema changes in advance
5. **TRUNCATE vs DELETE** - use TRUNCATE for speed, DELETE for control

---

*End of Chapter 09*
