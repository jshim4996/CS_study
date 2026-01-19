# Chapter 04. SQL Overview and SELECT (SQL 개요와 SELECT)

> Master the fundamentals of SQL, the universal language for database interaction, focusing on data retrieval with SELECT statements.

---

## What is SQL? (SQL이란?)

### Concept (개념)
SQL (Structured Query Language, 구조화 질의 언어) is a standard programming language designed for managing and manipulating relational databases. It provides a consistent way to interact with different database management systems.

### History
- 1970s: Developed at IBM as SEQUEL (Structured English Query Language)
- 1986: Became ANSI standard
- Multiple versions: SQL-86, SQL-89, SQL-92, SQL:1999, SQL:2003, SQL:2011, SQL:2016

### Characteristics of SQL
- **Declarative Language (선언적 언어)**: Specify what you want, not how to get it
- **Set-Based (집합 기반)**: Operates on sets of data
- **Standardized (표준화)**: ANSI/ISO standard (with vendor extensions)
- **Case-Insensitive (대소문자 무관)**: Keywords can be upper or lower case

---

## DDL, DML, DCL, TCL

### Concept (개념)
SQL commands are categorized based on their function in database management.

### Categories

#### 1. DDL - Data Definition Language (데이터 정의 언어)
Define and modify database structure:
- `CREATE`: Create database objects (tables, indexes, views)
- `ALTER`: Modify existing objects
- `DROP`: Delete objects
- `TRUNCATE`: Remove all data from table

```sql
-- DDL Examples
CREATE TABLE students (id INT PRIMARY KEY, name VARCHAR(100));
ALTER TABLE students ADD email VARCHAR(100);
DROP TABLE students;
```

#### 2. DML - Data Manipulation Language (데이터 조작 언어)
Manipulate data within tables:
- `SELECT`: Retrieve data
- `INSERT`: Add new records
- `UPDATE`: Modify existing records
- `DELETE`: Remove records

```sql
-- DML Examples
SELECT * FROM students;
INSERT INTO students VALUES (1, 'Kim');
UPDATE students SET name = 'Lee' WHERE id = 1;
DELETE FROM students WHERE id = 1;
```

#### 3. DCL - Data Control Language (데이터 제어 언어)
Control access and permissions:
- `GRANT`: Give privileges
- `REVOKE`: Remove privileges

```sql
-- DCL Examples
GRANT SELECT, INSERT ON students TO user1;
REVOKE INSERT ON students FROM user1;
```

#### 4. TCL - Transaction Control Language (트랜잭션 제어 언어)
Manage transactions:
- `COMMIT`: Save changes
- `ROLLBACK`: Undo changes
- `SAVEPOINT`: Set a savepoint

```sql
-- TCL Examples
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

## SELECT Statement (SELECT 문)

### Concept (개념)
The SELECT statement is used to retrieve data from one or more tables. It is the most commonly used SQL statement.

### Basic Syntax
```sql
SELECT column1, column2, ...
FROM table_name;
```

### Selecting All Columns
```sql
-- Select all columns using asterisk (*)
SELECT * FROM employees;
```

### Selecting Specific Columns
```sql
-- Select only specific columns
SELECT first_name, last_name, salary
FROM employees;
```

### Column Aliases
```sql
-- Using AS keyword for aliases
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary AS monthly_salary,
    salary * 12 AS annual_salary
FROM employees;

-- AS keyword is optional
SELECT first_name "First Name", salary * 12 annual_salary
FROM employees;
```

### Example
```sql
-- Sample table creation and data
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE
);

INSERT INTO employees VALUES
(1, 'John', 'Kim', 'john.kim@company.com', 'Engineering', 75000, '2020-01-15'),
(2, 'Sarah', 'Lee', 'sarah.lee@company.com', 'Marketing', 65000, '2019-06-20'),
(3, 'Mike', 'Park', 'mike.park@company.com', 'Engineering', 80000, '2018-03-10'),
(4, 'Emily', 'Choi', 'emily.choi@company.com', 'HR', 60000, '2021-09-01'),
(5, 'David', 'Jung', 'david.jung@company.com', 'Engineering', 72000, '2020-07-15');

-- Basic SELECT examples
SELECT * FROM employees;

SELECT first_name, last_name, department
FROM employees;

SELECT
    first_name || ' ' || last_name AS full_name,  -- Concatenation
    salary,
    salary * 1.1 AS salary_after_raise
FROM employees;
```

---

## WHERE Clause (WHERE 절)

### Concept (개념)
The WHERE clause filters rows based on specified conditions. Only rows that satisfy the condition are included in the result.

### Basic Syntax
```sql
SELECT columns
FROM table_name
WHERE condition;
```

### Example
```sql
-- Filter by exact value
SELECT * FROM employees
WHERE department = 'Engineering';

-- Filter by numeric comparison
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 70000;

-- Filter by date
SELECT * FROM employees
WHERE hire_date > '2020-01-01';
```

---

## Comparison and Logical Operators (비교 연산자와 논리 연산자)

### Comparison Operators (비교 연산자)

| Operator | Description | Korean |
|----------|-------------|--------|
| `=` | Equal to | 같음 |
| `<>` or `!=` | Not equal to | 같지 않음 |
| `>` | Greater than | 초과 |
| `<` | Less than | 미만 |
| `>=` | Greater than or equal | 이상 |
| `<=` | Less than or equal | 이하 |

```sql
-- Comparison operators
SELECT * FROM employees WHERE salary = 75000;
SELECT * FROM employees WHERE salary <> 75000;
SELECT * FROM employees WHERE salary > 70000;
SELECT * FROM employees WHERE salary >= 70000;
SELECT * FROM employees WHERE salary < 70000;
SELECT * FROM employees WHERE salary <= 70000;
```

### Logical Operators (논리 연산자)

| Operator | Description | Korean |
|----------|-------------|--------|
| `AND` | Both conditions must be true | 그리고 |
| `OR` | At least one condition must be true | 또는 |
| `NOT` | Negates a condition | 부정 |

```sql
-- AND: Both conditions must be true
SELECT * FROM employees
WHERE department = 'Engineering' AND salary > 75000;

-- OR: Either condition can be true
SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing';

-- NOT: Negation
SELECT * FROM employees
WHERE NOT department = 'HR';

-- Combining operators (use parentheses for clarity)
SELECT * FROM employees
WHERE (department = 'Engineering' OR department = 'Marketing')
AND salary > 65000;

-- Without parentheses, AND has higher precedence than OR
-- This behaves differently:
SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing' AND salary > 65000;
-- Equivalent to: Engineering OR (Marketing AND salary > 65000)
```

---

## LIKE, IN, BETWEEN

### LIKE - Pattern Matching (패턴 매칭)

| Wildcard | Description | Korean |
|----------|-------------|--------|
| `%` | Any sequence of characters (0 or more) | 모든 문자열 |
| `_` | Any single character | 단일 문자 |

```sql
-- LIKE examples
-- Names starting with 'J'
SELECT * FROM employees
WHERE first_name LIKE 'J%';

-- Names ending with 'e'
SELECT * FROM employees
WHERE first_name LIKE '%e';

-- Names containing 'ar'
SELECT * FROM employees
WHERE first_name LIKE '%ar%';

-- Names with exactly 4 characters
SELECT * FROM employees
WHERE first_name LIKE '____';

-- Second letter is 'a'
SELECT * FROM employees
WHERE first_name LIKE '_a%';

-- Case-insensitive search (varies by database)
-- MySQL: LIKE is case-insensitive by default
-- PostgreSQL: Use ILIKE for case-insensitive
SELECT * FROM employees
WHERE LOWER(first_name) LIKE 'j%';
```

### IN - Value List (값 목록)

```sql
-- IN: Check if value is in a list
SELECT * FROM employees
WHERE department IN ('Engineering', 'Marketing', 'HR');

-- Equivalent to using OR
SELECT * FROM employees
WHERE department = 'Engineering'
   OR department = 'Marketing'
   OR department = 'HR';

-- NOT IN: Exclude values in list
SELECT * FROM employees
WHERE department NOT IN ('HR', 'Finance');

-- IN with subquery
SELECT * FROM employees
WHERE department IN (
    SELECT department_name
    FROM departments
    WHERE budget > 100000
);
```

### BETWEEN - Range (범위)

```sql
-- BETWEEN: Inclusive range
SELECT * FROM employees
WHERE salary BETWEEN 60000 AND 75000;

-- Equivalent to using AND
SELECT * FROM employees
WHERE salary >= 60000 AND salary <= 75000;

-- NOT BETWEEN
SELECT * FROM employees
WHERE salary NOT BETWEEN 60000 AND 75000;

-- BETWEEN with dates
SELECT * FROM employees
WHERE hire_date BETWEEN '2019-01-01' AND '2020-12-31';

-- BETWEEN with strings (alphabetical order)
SELECT * FROM employees
WHERE first_name BETWEEN 'A' AND 'M';
```

---

## NULL Handling (NULL 처리)

### Concept (개념)
NULL represents missing or unknown data. NULL is not equal to anything, including another NULL. Special operators are needed to work with NULL values.

### NULL Operators

```sql
-- Add column that allows NULL
ALTER TABLE employees ADD manager_id INT;

UPDATE employees SET manager_id = 3 WHERE employee_id IN (1, 2);
UPDATE employees SET manager_id = NULL WHERE employee_id IN (3, 4, 5);

-- Check for NULL using IS NULL
SELECT * FROM employees
WHERE manager_id IS NULL;

-- Check for NOT NULL
SELECT * FROM employees
WHERE manager_id IS NOT NULL;

-- WRONG: This will not work!
SELECT * FROM employees
WHERE manager_id = NULL;  -- Returns no rows, even if NULLs exist

-- NULL in comparisons (result is UNKNOWN, not TRUE or FALSE)
SELECT * FROM employees
WHERE salary > NULL;  -- Returns no rows
```

### NULL and Aggregate Functions

```sql
-- NULL values are ignored by aggregate functions
SELECT AVG(salary) FROM employees;  -- NULLs excluded from calculation

SELECT COUNT(*) FROM employees;        -- Counts all rows
SELECT COUNT(manager_id) FROM employees;  -- Counts non-NULL values only
```

### COALESCE and IFNULL

```sql
-- COALESCE: Return first non-NULL value
SELECT
    first_name,
    COALESCE(manager_id, 0) AS manager_id_or_zero,
    COALESCE(manager_id, -1, 0) AS first_non_null
FROM employees;

-- IFNULL (MySQL) / ISNULL (SQL Server)
-- Return second value if first is NULL
SELECT
    first_name,
    IFNULL(manager_id, 0) AS manager_id_or_zero
FROM employees;

-- NVL in Oracle
-- SELECT NVL(manager_id, 0) FROM employees;
```

### NULLIF

```sql
-- NULLIF: Return NULL if two values are equal
SELECT NULLIF(10, 10);  -- Returns NULL
SELECT NULLIF(10, 20);  -- Returns 10

-- Useful for avoiding division by zero
SELECT
    total_sales,
    NULLIF(number_of_sales, 0) AS safe_divisor,
    total_sales / NULLIF(number_of_sales, 0) AS avg_per_sale
FROM sales_data;
```

---

## Complete SELECT Example

```sql
-- Comprehensive SELECT example combining all concepts
SELECT
    employee_id,
    first_name || ' ' || last_name AS full_name,
    department,
    salary,
    COALESCE(manager_id, 0) AS manager
FROM employees
WHERE
    department IN ('Engineering', 'Marketing')
    AND salary BETWEEN 65000 AND 80000
    AND first_name LIKE '%a%'
    AND manager_id IS NOT NULL;

-- Result explanation:
-- Filters for Engineering or Marketing department
-- Salary between 65000 and 80000 (inclusive)
-- First name contains 'a'
-- Has a manager (manager_id is not NULL)
```

---

## Summary (요약)

| Concept | Korean | Description |
|---------|--------|-------------|
| SQL | 구조화 질의 언어 | Standard language for databases |
| DDL | 데이터 정의 언어 | CREATE, ALTER, DROP |
| DML | 데이터 조작 언어 | SELECT, INSERT, UPDATE, DELETE |
| DCL | 데이터 제어 언어 | GRANT, REVOKE |
| SELECT | 선택 | Retrieve data from tables |
| WHERE | 조건절 | Filter rows by condition |
| LIKE | 패턴 매칭 | Pattern matching with wildcards |
| IN | 포함 | Check value in list |
| BETWEEN | 범위 | Range comparison |
| NULL | 널 | Missing or unknown value |
| IS NULL | 널 체크 | Check for NULL value |

---
