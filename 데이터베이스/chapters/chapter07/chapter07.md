# Chapter 07. Subqueries (서브쿼리)

> Learn to write powerful nested queries that allow complex data retrieval by embedding one query inside another.

---

## What is a Subquery? (서브쿼리란?)

### Concept (개념)
A subquery (서브쿼리) is a query nested inside another query. The inner query executes first and its result is used by the outer query. Subqueries can appear in SELECT, FROM, WHERE, and HAVING clauses.

### Terminology
- **Outer Query (외부 쿼리)**: The main query containing the subquery
- **Inner Query (내부 쿼리)**: The nested query (subquery)
- **Correlated Subquery (상관 서브쿼리)**: Subquery that references the outer query
- **Non-correlated Subquery (비상관 서브쿼리)**: Independent subquery

### Sample Data
```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50),
    budget DECIMAL(12,2)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE
);

INSERT INTO departments VALUES
(1, 'Engineering', 500000),
(2, 'Marketing', 300000),
(3, 'HR', 200000),
(4, 'Finance', 400000);

INSERT INTO employees VALUES
(101, 'Kim Minjun', 1, 85000, '2019-03-15'),
(102, 'Lee Soyeon', 1, 75000, '2020-06-20'),
(103, 'Park Jihoon', 2, 65000, '2021-01-10'),
(104, 'Choi Yuna', 2, 70000, '2020-09-01'),
(105, 'Jung Seungho', 3, 55000, '2022-02-15'),
(106, 'Kang Minji', 1, 90000, '2018-08-20'),
(107, 'Han Dongwoo', 4, 80000, '2019-11-05');
```

---

## Scalar Subquery (스칼라 서브쿼리)

### Concept (개념)
A scalar subquery (스칼라 서브쿼리) returns a single value (one row, one column). It can be used anywhere a single value is expected, such as in SELECT or WHERE clauses.

### Example
```sql
-- Scalar subquery in SELECT: Show each employee's salary vs average
SELECT
    name,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;

/*
Result:
| name          | salary | avg_salary | diff_from_avg |
|---------------|--------|------------|---------------|
| Kim Minjun    | 85000  | 74285.71   | 10714.29      |
| Lee Soyeon    | 75000  | 74285.71   | 714.29        |
| Park Jihoon   | 65000  | 74285.71   | -9285.71      |
| ...           |        |            |               |
*/

-- Scalar subquery in WHERE: Find employees earning above average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Scalar subquery with specific value
SELECT name, salary
FROM employees
WHERE department_id = (
    SELECT department_id
    FROM departments
    WHERE department_name = 'Engineering'
);

-- Scalar subquery must return exactly one value
-- This would ERROR if more than one row returned:
-- SELECT name FROM employees WHERE salary = (SELECT salary FROM employees);
```

---

## Inline View (인라인 뷰)

### Concept (개념)
An inline view (인라인 뷰) is a subquery in the FROM clause. It creates a temporary result set that the outer query can select from, essentially acting as a virtual table.

### Example
```sql
-- Inline view: Department salary statistics
SELECT
    dept_stats.department_name,
    dept_stats.avg_salary,
    dept_stats.employee_count
FROM (
    SELECT
        d.department_name,
        AVG(e.salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_id, d.department_name
) AS dept_stats
WHERE dept_stats.avg_salary > 70000;

/*
Result:
| department_name | avg_salary | employee_count |
|-----------------|------------|----------------|
| Engineering     | 83333.33   | 3              |
| Finance         | 80000.00   | 1              |
*/

-- Multiple inline views
SELECT
    high_earners.name,
    high_earners.salary,
    dept_avg.avg_dept_salary
FROM (
    SELECT name, salary, department_id
    FROM employees
    WHERE salary > 75000
) AS high_earners
JOIN (
    SELECT department_id, AVG(salary) AS avg_dept_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg ON high_earners.department_id = dept_avg.department_id;

-- Inline view for pagination with row numbers
SELECT *
FROM (
    SELECT
        ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
        name,
        salary
    FROM employees
) AS numbered_employees
WHERE row_num BETWEEN 3 AND 5;
```

---

## Nested Subquery (중첩 서브쿼리)

### Concept (개념)
A nested subquery (중첩 서브쿼리) is used in the WHERE clause and returns multiple rows. It's typically used with operators like IN, ANY, ALL, or comparison operators.

### Example
```sql
-- Nested subquery with IN: Employees in high-budget departments
SELECT name, salary
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE budget > 300000
);

/*
Result: Engineering (500000) and Finance (400000) employees
| name          | salary |
|---------------|--------|
| Kim Minjun    | 85000  |
| Lee Soyeon    | 75000  |
| Kang Minji    | 90000  |
| Han Dongwoo   | 80000  |
*/

-- Nested subquery with comparison: Employees earning more than any Marketing employee
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT MAX(salary)
    FROM employees
    WHERE department_id = (
        SELECT department_id
        FROM departments
        WHERE department_name = 'Marketing'
    )
);

-- Multiple levels of nesting
SELECT name, salary
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE budget > (
        SELECT AVG(budget)
        FROM departments
    )
);
```

### ANY and ALL Operators

```sql
-- ANY: Compare with any value in the subquery result
-- "Salary greater than at least one Engineering salary"
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department_id = 1
);
-- Equivalent to: salary > MIN(Engineering salaries)

-- ALL: Compare with all values in the subquery result
-- "Salary greater than all Marketing salaries"
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department_id = 2
);
-- Equivalent to: salary > MAX(Marketing salaries)

/*
ANY/ALL with different operators:
= ANY  → same as IN
<> ALL → same as NOT IN
> ANY  → greater than minimum
> ALL  → greater than maximum
< ANY  → less than maximum
< ALL  → less than minimum
*/
```

---

## Correlated Subquery (상관 서브쿼리)

### Concept (개념)
A correlated subquery (상관 서브쿼리) references columns from the outer query. It executes once for each row processed by the outer query, making it potentially slower but more flexible.

### Example
```sql
-- Correlated subquery: Employees earning above their department average
SELECT
    e.name,
    e.salary,
    e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id  -- References outer query
);

/*
Result:
| name          | salary | department_id |
|---------------|--------|---------------|
| Kim Minjun    | 85000  | 1             |
| Kang Minji    | 90000  | 1             |
| Choi Yuna     | 70000  | 2             |
*/

-- Correlated subquery in SELECT: Show department average for each employee
SELECT
    e.name,
    e.salary,
    (
        SELECT AVG(e2.salary)
        FROM employees e2
        WHERE e2.department_id = e.department_id
    ) AS dept_avg_salary
FROM employees e;

-- Find the highest earner in each department
SELECT e.name, e.salary, e.department_id
FROM employees e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

---

## EXISTS and NOT EXISTS (존재 여부 확인)

### Concept (개념)
EXISTS returns TRUE if the subquery returns at least one row. NOT EXISTS returns TRUE if the subquery returns no rows. These are typically used with correlated subqueries.

### Example
```sql
-- EXISTS: Departments that have employees
SELECT d.department_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);

/*
Result: (Finance excluded - no employees)
| department_name |
|-----------------|
| Engineering     |
| Marketing       |
| HR              |
*/

-- NOT EXISTS: Departments without employees
SELECT d.department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);

/*
Result:
| department_name |
|-----------------|
| Finance         |
*/

-- EXISTS with multiple conditions
SELECT d.department_name, d.budget
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
    AND e.salary > 80000
);

-- Using EXISTS instead of IN for better performance with large datasets
-- These are equivalent but EXISTS may be faster:
-- Using IN:
SELECT name FROM employees
WHERE department_id IN (SELECT department_id FROM departments WHERE budget > 300000);

-- Using EXISTS:
SELECT e.name FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
    AND d.budget > 300000
);
```

---

## IN vs EXISTS (IN과 EXISTS 비교)

### Concept (개념)
Both IN and EXISTS can achieve similar results, but they differ in performance characteristics and behavior with NULL values.

### Comparison

| Aspect | IN | EXISTS |
|--------|-----|--------|
| Performance with large outer query | Usually slower | Usually faster |
| Performance with large subquery | Usually faster | Usually slower |
| NULL handling | Returns UNKNOWN | Handles NULLs correctly |
| Correlated | Typically non-correlated | Typically correlated |
| Readable for | Simple set membership | Complex conditions |

### Example
```sql
-- IN: Good when subquery result is small
SELECT name FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE budget > 300000
);

-- EXISTS: Good when outer query result is small
SELECT e.name FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
    AND d.budget > 300000
);

-- NULL handling difference
-- Create test data with NULL
INSERT INTO employees VALUES (108, 'Test User', NULL, 60000, '2023-01-01');

-- IN with NULL in subquery: may miss rows
SELECT name FROM employees
WHERE department_id NOT IN (
    SELECT department_id FROM departments WHERE budget < 300000
);
-- This may not return 'Test User' if there's a NULL in the subquery

-- EXISTS handles NULLs correctly
SELECT e.name FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
    AND d.budget < 300000
);
```

### Performance Tips

```sql
-- Tip 1: Use EXISTS for large tables when checking existence
-- Instead of:
SELECT * FROM orders o
WHERE customer_id IN (SELECT customer_id FROM vip_customers);

-- Use:
SELECT * FROM orders o
WHERE EXISTS (SELECT 1 FROM vip_customers v WHERE v.customer_id = o.customer_id);

-- Tip 2: Use IN for small, fixed value sets
SELECT * FROM products
WHERE category IN ('Electronics', 'Books', 'Clothing');

-- Tip 3: Check execution plans to compare performance
EXPLAIN SELECT name FROM employees WHERE department_id IN (...);
EXPLAIN SELECT name FROM employees WHERE EXISTS (...);
```

---

## Common Table Expressions (CTE) - WITH Clause

### Concept (개념)
A CTE (Common Table Expression) defined with WITH clause provides a way to write more readable subqueries. CTEs can be referenced multiple times and can be recursive.

### Example
```sql
-- Simple CTE instead of inline view
WITH dept_stats AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department_id
)
SELECT
    d.department_name,
    ds.avg_salary,
    ds.employee_count
FROM departments d
JOIN dept_stats ds ON d.department_id = ds.department_id
WHERE ds.avg_salary > 70000;

-- Multiple CTEs
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 70000
),
engineering_dept AS (
    SELECT department_id FROM departments WHERE department_name = 'Engineering'
)
SELECT he.name, he.salary
FROM high_earners he
WHERE he.department_id IN (SELECT department_id FROM engineering_dept);

-- CTE referenced multiple times
WITH avg_salary AS (
    SELECT AVG(salary) AS avg_sal FROM employees
)
SELECT
    name,
    salary,
    (SELECT avg_sal FROM avg_salary) AS company_avg,
    salary - (SELECT avg_sal FROM avg_salary) AS diff
FROM employees;
```

---

## Summary (요약)

| Subquery Type | Korean | Location | Returns |
|---------------|--------|----------|---------|
| Scalar Subquery | 스칼라 서브쿼리 | SELECT, WHERE | Single value |
| Inline View | 인라인 뷰 | FROM | Virtual table |
| Nested Subquery | 중첩 서브쿼리 | WHERE | Multiple rows |
| Correlated Subquery | 상관 서브쿼리 | Any | References outer query |
| EXISTS | 존재 확인 | WHERE | TRUE/FALSE |
| CTE (WITH) | 공통 테이블 표현식 | Before SELECT | Named subquery |

---
