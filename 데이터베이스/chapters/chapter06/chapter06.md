# Chapter 06. JOIN (조인)

> Master the art of combining data from multiple tables using various JOIN types to create meaningful relationships.

---

## What is a JOIN? (JOIN이란?)

### Concept (개념)
A JOIN (조인) is an operation that combines rows from two or more tables based on a related column. JOINs are fundamental to relational databases because data is typically normalized across multiple tables.

### Why Use JOINs?
- **Data Normalization (정규화)**: Related data is stored in separate tables
- **Avoid Redundancy (중복 방지)**: Don't repeat data across tables
- **Data Integrity (무결성)**: Maintain consistent relationships
- **Query Flexibility (쿼리 유연성)**: Combine data in various ways

### Sample Data for Examples
```sql
-- Create sample tables
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50),
    location VARCHAR(50)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE
);

CREATE TABLE projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100),
    budget DECIMAL(12,2)
);

CREATE TABLE employee_projects (
    employee_id INT,
    project_id INT,
    role VARCHAR(50),
    hours_allocated INT,
    PRIMARY KEY (employee_id, project_id)
);

-- Insert sample data
INSERT INTO departments VALUES
(1, 'Engineering', 'Building A'),
(2, 'Marketing', 'Building B'),
(3, 'HR', 'Building A'),
(4, 'Finance', 'Building C');  -- No employees in Finance

INSERT INTO employees VALUES
(101, 'Kim Minjun', 1, 75000, '2020-01-15'),
(102, 'Lee Soyeon', 1, 80000, '2019-06-20'),
(103, 'Park Jihoon', 2, 65000, '2021-03-10'),
(104, 'Choi Yuna', 3, 60000, '2020-09-01'),
(105, 'Jung Seungho', NULL, 55000, '2022-01-05');  -- No department

INSERT INTO projects VALUES
(1, 'Website Redesign', 50000),
(2, 'Mobile App', 100000),
(3, 'Data Analytics', 75000);

INSERT INTO employee_projects VALUES
(101, 1, 'Developer', 20),
(101, 2, 'Lead Developer', 30),
(102, 2, 'Developer', 40),
(103, 1, 'Designer', 15);
```

---

## INNER JOIN (내부 조인)

### Concept (개념)
INNER JOIN returns only the rows where there is a match in both tables. Rows without matching values in the join condition are excluded from the result.

### Syntax
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;

-- Alternative syntax (implicit join)
SELECT columns
FROM table1, table2
WHERE table1.column = table2.column;
```

### Example
```sql
-- Basic INNER JOIN
SELECT
    e.employee_id,
    e.name,
    d.department_name,
    d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

/*
Result: (Jung Seungho excluded - no department)
| employee_id | name         | department_name | location   |
|-------------|--------------|-----------------|------------|
| 101         | Kim Minjun   | Engineering     | Building A |
| 102         | Lee Soyeon   | Engineering     | Building A |
| 103         | Park Jihoon  | Marketing       | Building B |
| 104         | Choi Yuna    | HR              | Building A |
*/

-- INNER JOIN with additional conditions
SELECT
    e.name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 65000;

-- Multiple conditions in JOIN
SELECT
    e.name,
    d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id
    AND d.location = 'Building A';
```

---

## LEFT JOIN (LEFT OUTER JOIN, 왼쪽 외부 조인)

### Concept (개념)
LEFT JOIN returns all rows from the left table and matching rows from the right table. If no match exists, NULL values are returned for right table columns.

### Syntax
```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

### Example
```sql
-- LEFT JOIN: All employees, even without department
SELECT
    e.employee_id,
    e.name,
    d.department_name,
    d.location
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

/*
Result: (All employees included)
| employee_id | name          | department_name | location   |
|-------------|---------------|-----------------|------------|
| 101         | Kim Minjun    | Engineering     | Building A |
| 102         | Lee Soyeon    | Engineering     | Building A |
| 103         | Park Jihoon   | Marketing       | Building B |
| 104         | Choi Yuna     | HR              | Building A |
| 105         | Jung Seungho  | NULL            | NULL       |
*/

-- Find employees without a department
SELECT
    e.employee_id,
    e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;

/*
Result:
| employee_id | name          |
|-------------|---------------|
| 105         | Jung Seungho  |
*/
```

---

## RIGHT JOIN (RIGHT OUTER JOIN, 오른쪽 외부 조인)

### Concept (개념)
RIGHT JOIN returns all rows from the right table and matching rows from the left table. If no match exists, NULL values are returned for left table columns.

### Syntax
```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;
```

### Example
```sql
-- RIGHT JOIN: All departments, even without employees
SELECT
    d.department_id,
    d.department_name,
    e.employee_id,
    e.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

/*
Result: (All departments included, Finance has no employees)
| department_id | department_name | employee_id | name         |
|---------------|-----------------|-------------|--------------|
| 1             | Engineering     | 101         | Kim Minjun   |
| 1             | Engineering     | 102         | Lee Soyeon   |
| 2             | Marketing       | 103         | Park Jihoon  |
| 3             | HR              | 104         | Choi Yuna    |
| 4             | Finance         | NULL        | NULL         |
*/

-- Find departments with no employees
SELECT
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;

/*
Result:
| department_id | department_name |
|---------------|-----------------|
| 4             | Finance         |
*/

-- Note: LEFT JOIN with tables swapped = RIGHT JOIN
SELECT d.department_name, e.name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id;
-- This gives the same result as the RIGHT JOIN above
```

---

## FULL OUTER JOIN (완전 외부 조인)

### Concept (개념)
FULL OUTER JOIN returns all rows from both tables. Where matches exist, data is combined. Where no match exists, NULL values fill the missing side.

### Syntax
```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2 ON table1.column = table2.column;
```

### Example
```sql
-- FULL OUTER JOIN: All employees and all departments
SELECT
    e.employee_id,
    e.name,
    d.department_id,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

/*
Result:
| employee_id | name          | department_id | department_name |
|-------------|---------------|---------------|-----------------|
| 101         | Kim Minjun    | 1             | Engineering     |
| 102         | Lee Soyeon    | 1             | Engineering     |
| 103         | Park Jihoon   | 2             | Marketing       |
| 104         | Choi Yuna     | 3             | HR              |
| 105         | Jung Seungho  | NULL          | NULL            |
| NULL        | NULL          | 4             | Finance         |
*/

-- MySQL doesn't support FULL OUTER JOIN directly
-- Workaround using UNION
SELECT e.employee_id, e.name, d.department_id, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
UNION
SELECT e.employee_id, e.name, d.department_id, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- Find unmatched rows (either side)
SELECT
    e.employee_id,
    e.name,
    d.department_id,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL OR d.department_id IS NULL;
```

---

## CROSS JOIN (교차 조인)

### Concept (개념)
CROSS JOIN returns the Cartesian product of both tables - every row from the first table combined with every row from the second table. No join condition is needed.

### Syntax
```sql
SELECT columns
FROM table1
CROSS JOIN table2;

-- Alternative syntax
SELECT columns
FROM table1, table2;
```

### Example
```sql
-- CROSS JOIN: All combinations
SELECT
    e.name,
    p.project_name
FROM employees e
CROSS JOIN projects p;

/*
Result: 5 employees × 3 projects = 15 rows
| name          | project_name     |
|---------------|------------------|
| Kim Minjun    | Website Redesign |
| Kim Minjun    | Mobile App       |
| Kim Minjun    | Data Analytics   |
| Lee Soyeon    | Website Redesign |
| Lee Soyeon    | Mobile App       |
| ... (15 rows total)              |
*/

-- Practical use: Generate all date-product combinations
CREATE TABLE dates_dim (date_value DATE);
INSERT INTO dates_dim VALUES ('2023-01-01'), ('2023-01-02'), ('2023-01-03');

CREATE TABLE products (product_id INT, product_name VARCHAR(50));
INSERT INTO products VALUES (1, 'Product A'), (2, 'Product B');

-- Calendar for sales reporting (even for days with no sales)
SELECT
    d.date_value,
    p.product_name
FROM dates_dim d
CROSS JOIN products p
ORDER BY d.date_value, p.product_name;
```

---

## SELF JOIN (자기 조인)

### Concept (개념)
A SELF JOIN is a join where a table is joined with itself. This is useful for hierarchical or recursive data structures.

### Example
```sql
-- Add manager column to employees
ALTER TABLE employees ADD manager_id INT;
UPDATE employees SET manager_id = 101 WHERE employee_id IN (102, 103);
UPDATE employees SET manager_id = 104 WHERE employee_id = 105;

-- SELF JOIN: Employee and their manager
SELECT
    e.employee_id,
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;

/*
Result:
| employee_id | employee_name | manager_name |
|-------------|---------------|--------------|
| 101         | Kim Minjun    | NULL         |
| 102         | Lee Soyeon    | Kim Minjun   |
| 103         | Park Jihoon   | Kim Minjun   |
| 104         | Choi Yuna     | NULL         |
| 105         | Jung Seungho  | Choi Yuna    |
*/

-- Find employees who earn more than their manager
SELECT
    e.name AS employee_name,
    e.salary AS employee_salary,
    m.name AS manager_name,
    m.salary AS manager_salary
FROM employees e
INNER JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;

-- Find all employees in the same department
SELECT
    e1.name AS employee1,
    e2.name AS employee2,
    e1.department_id
FROM employees e1
INNER JOIN employees e2
    ON e1.department_id = e2.department_id
    AND e1.employee_id < e2.employee_id;  -- Avoid duplicates and self-matches
```

---

## Multiple Table JOIN (다중 테이블 조인)

### Concept (개념)
You can join more than two tables by chaining JOIN clauses. Each JOIN combines the result of previous joins with another table.

### Example
```sql
-- Three-way JOIN: Employees, Departments, and Projects
SELECT
    e.name AS employee_name,
    d.department_name,
    p.project_name,
    ep.role,
    ep.hours_allocated
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN employee_projects ep ON e.employee_id = ep.employee_id
INNER JOIN projects p ON ep.project_id = p.project_id;

/*
Result:
| employee_name | department_name | project_name     | role           | hours |
|---------------|-----------------|------------------|----------------|-------|
| Kim Minjun    | Engineering     | Website Redesign | Developer      | 20    |
| Kim Minjun    | Engineering     | Mobile App       | Lead Developer | 30    |
| Lee Soyeon    | Engineering     | Mobile App       | Developer      | 40    |
| Park Jihoon   | Marketing       | Website Redesign | Designer       | 15    |
*/

-- Mixed JOIN types
SELECT
    d.department_name,
    e.name AS employee_name,
    p.project_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
LEFT JOIN projects p ON ep.project_id = p.project_id;

-- Comprehensive example with aggregation
SELECT
    d.department_name,
    COUNT(DISTINCT e.employee_id) AS employee_count,
    COUNT(DISTINCT ep.project_id) AS project_count,
    SUM(p.budget) AS total_project_budget
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
LEFT JOIN projects p ON ep.project_id = p.project_id
GROUP BY d.department_id, d.department_name
ORDER BY total_project_budget DESC NULLS LAST;
```

---

## JOIN Summary and Comparison (조인 요약 및 비교)

### Visual Comparison

| JOIN Type | Left Table | Right Table | Result |
|-----------|------------|-------------|--------|
| INNER JOIN | Matching only | Matching only | Intersection |
| LEFT JOIN | All rows | Matching only | Left + matches |
| RIGHT JOIN | Matching only | All rows | Right + matches |
| FULL OUTER JOIN | All rows | All rows | Union |
| CROSS JOIN | All rows | All rows | Cartesian product |

### Choosing the Right JOIN

```sql
-- INNER JOIN: When you need matching data from both tables
-- "Find all employees and their department names"
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- LEFT JOIN: When you need all rows from the primary table
-- "Find all employees, showing their department if assigned"
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- RIGHT JOIN: When you need all rows from the secondary table
-- "Find all departments, showing their employees if any"
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN: When you need all data from both tables
-- "Find all employees and departments, matched or not"
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- CROSS JOIN: When you need all possible combinations
-- "Generate all employee-project assignment possibilities"
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

---

## Summary (요약)

| JOIN Type | Korean | Returns |
|-----------|--------|---------|
| INNER JOIN | 내부 조인 | Matching rows only |
| LEFT JOIN | 왼쪽 외부 조인 | All left + matching right |
| RIGHT JOIN | 오른쪽 외부 조인 | Matching left + all right |
| FULL OUTER JOIN | 완전 외부 조인 | All rows from both tables |
| CROSS JOIN | 교차 조인 | Cartesian product |
| SELF JOIN | 자기 조인 | Table joined with itself |

---
