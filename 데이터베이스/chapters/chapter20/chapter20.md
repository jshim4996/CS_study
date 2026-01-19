# Chapter 20. Views and Stored Procedures (뷰와 저장 프로시저)

> Learn to create reusable database objects that simplify complex queries and encapsulate business logic.

---

## What is a View? (뷰란?)

### Concept (개념)
A view (뷰) is a virtual table based on the result of a SQL query. It does not store data physically but provides a way to encapsulate complex queries, restrict access to sensitive columns, and simplify data access.

### View Characteristics

```
Physical Table vs View:

Physical Table:                 View:
+------------------+            +------------------+
| Actual data      |            | No stored data   |
| stored on disk   |            | Query stored     |
| Uses storage     |            | No storage used  |
+------------------+            +------------------+
        ↑                               ↑
    Data lives here              Query definition
                                 lives here
                                        ↓
                                 When accessed,
                                 query executes
                                        ↓
                                 Returns result
```

### Sample Data

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    status VARCHAR(20)
);

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50),
    manager_id INT,
    budget DECIMAL(12,2)
);

CREATE TABLE salary_history (
    history_id INT PRIMARY KEY,
    employee_id INT,
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    change_date DATE
);

INSERT INTO departments VALUES
(1, 'Engineering', 101, 500000),
(2, 'Marketing', 102, 300000),
(3, 'HR', 103, 200000);

INSERT INTO employees VALUES
(101, 'Kim Minjun', 'kim@company.com', 1, 95000, '2018-03-15', 'active'),
(102, 'Lee Soyeon', 'lee@company.com', 2, 85000, '2019-06-20', 'active'),
(103, 'Park Jihoon', 'park@company.com', 3, 75000, '2020-01-10', 'active'),
(104, 'Choi Yuna', 'choi@company.com', 1, 80000, '2020-09-01', 'active'),
(105, 'Jung Seungho', 'jung@company.com', 2, 65000, '2021-02-15', 'inactive');
```

---

## Creating and Using Views (뷰 생성과 사용)

### Basic CREATE VIEW Syntax

```sql
-- Basic view creation
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;

-- Create or replace (update existing view)
CREATE OR REPLACE VIEW view_name AS
SELECT ...;

-- Drop a view
DROP VIEW view_name;
DROP VIEW IF EXISTS view_name;
```

### Example 1: Simple View

```sql
-- Create a view showing active employees
CREATE VIEW active_employees AS
SELECT employee_id, name, email, department_id
FROM employees
WHERE status = 'active';

-- Use the view like a table
SELECT * FROM active_employees;

/*
Result:
| employee_id | name         | email            | department_id |
|-------------|--------------|------------------|---------------|
| 101         | Kim Minjun   | kim@company.com  | 1             |
| 102         | Lee Soyeon   | lee@company.com  | 2             |
| 103         | Park Jihoon  | park@company.com | 3             |
| 104         | Choi Yuna    | choi@company.com | 1             |
*/

-- Filter view results
SELECT * FROM active_employees WHERE department_id = 1;
```

### Example 2: View with JOIN

```sql
-- Create a view combining employees and departments
CREATE VIEW employee_details AS
SELECT
    e.employee_id,
    e.name AS employee_name,
    e.email,
    d.department_name,
    e.salary,
    e.hire_date
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.status = 'active';

-- Query the view
SELECT * FROM employee_details WHERE department_name = 'Engineering';

/*
Result:
| employee_id | employee_name | email           | department_name | salary | hire_date  |
|-------------|---------------|-----------------|-----------------|--------|------------|
| 101         | Kim Minjun    | kim@company.com | Engineering     | 95000  | 2018-03-15 |
| 104         | Choi Yuna     | choi@company.com| Engineering     | 80000  | 2020-09-01 |
*/
```

### Example 3: View with Aggregation

```sql
-- Department summary view
CREATE VIEW department_summary AS
SELECT
    d.department_id,
    d.department_name,
    COUNT(e.employee_id) AS employee_count,
    AVG(e.salary) AS avg_salary,
    SUM(e.salary) AS total_salary,
    MAX(e.salary) AS max_salary,
    MIN(e.salary) AS min_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
    AND e.status = 'active'
GROUP BY d.department_id, d.department_name;

-- Use the summary view
SELECT * FROM department_summary ORDER BY avg_salary DESC;

/*
Result:
| department_id | department_name | employee_count | avg_salary | total_salary | max_salary | min_salary |
|---------------|-----------------|----------------|------------|--------------|------------|------------|
| 1             | Engineering     | 2              | 87500.00   | 175000.00    | 95000      | 80000      |
| 2             | Marketing       | 1              | 85000.00   | 85000.00     | 85000      | 85000      |
| 3             | HR              | 1              | 75000.00   | 75000.00     | 75000      | 75000      |
*/
```

### View for Data Security

```sql
-- Hide sensitive salary information
CREATE VIEW employee_public_info AS
SELECT employee_id, name, email, department_id, hire_date
FROM employees;
-- Note: salary column is excluded

-- Grant access to the view, not the base table
GRANT SELECT ON employee_public_info TO public_user;
-- Users with public_user role can see employee info but not salaries
```

### Updatable Views

```sql
-- Simple views can be updatable (allow INSERT, UPDATE, DELETE)
CREATE VIEW engineering_employees AS
SELECT employee_id, name, email, salary
FROM employees
WHERE department_id = 1;

-- This may work (depends on database and view complexity)
UPDATE engineering_employees
SET salary = salary * 1.1
WHERE employee_id = 104;

-- The underlying employees table is updated

/*
Conditions for updatable views (vary by database):
- No aggregation (GROUP BY, HAVING, aggregate functions)
- No DISTINCT
- No subqueries in SELECT
- No UNION
- Single base table (usually)
- All NOT NULL columns must be included
*/

-- WITH CHECK OPTION: Prevent updates that would remove rows from view
CREATE VIEW engineering_employees_checked AS
SELECT employee_id, name, email, salary, department_id
FROM employees
WHERE department_id = 1
WITH CHECK OPTION;

-- This would fail because it changes department_id to 2
-- which would remove the row from the view
UPDATE engineering_employees_checked
SET department_id = 2
WHERE employee_id = 104;
-- Error: CHECK OPTION failed
```

---

## Stored Procedures (저장 프로시저)

### Concept (개념)
A stored procedure (저장 프로시저) is a set of SQL statements stored in the database that can be executed as a single unit. It can accept parameters, contain control flow logic, and return results.

### Stored Procedure Flow

```
Application                    Database
+----------------+            +------------------+
|                |   CALL     |                  |
| CALL proc(args)| ---------> | Stored Procedure |
|                |            |  - SQL Statement |
|                |            |  - SQL Statement |
|                |            |  - IF/ELSE       |
|                |            |  - LOOP          |
|                | <--------- |  - RETURN        |
|   Result       |            |                  |
+----------------+            +------------------+

Benefits:
- Reduced network traffic (one call vs multiple queries)
- Centralized business logic
- Better security (grant EXECUTE, not table access)
- Precompiled for better performance
```

### Basic Syntax (MySQL)

```sql
-- Create a simple procedure
DELIMITER //
CREATE PROCEDURE procedure_name()
BEGIN
    -- SQL statements here
    SELECT * FROM employees;
END //
DELIMITER ;

-- Call the procedure
CALL procedure_name();

-- Drop a procedure
DROP PROCEDURE IF EXISTS procedure_name;
```

### Procedure with Parameters

```sql
-- IN parameter: Input only
-- OUT parameter: Output only
-- INOUT parameter: Both input and output

DELIMITER //
CREATE PROCEDURE get_employee_by_dept(
    IN dept_id INT,
    OUT emp_count INT
)
BEGIN
    -- Return employees in the department
    SELECT * FROM employees WHERE department_id = dept_id;

    -- Set the output parameter
    SELECT COUNT(*) INTO emp_count
    FROM employees
    WHERE department_id = dept_id;
END //
DELIMITER ;

-- Call with parameters
CALL get_employee_by_dept(1, @count);
SELECT @count AS employee_count;
```

### Procedure with Control Flow

```sql
DELIMITER //
CREATE PROCEDURE give_raise(
    IN emp_id INT,
    IN raise_percent DECIMAL(5,2),
    OUT result_message VARCHAR(100)
)
BEGIN
    DECLARE current_salary DECIMAL(10,2);
    DECLARE new_salary DECIMAL(10,2);

    -- Get current salary
    SELECT salary INTO current_salary
    FROM employees
    WHERE employee_id = emp_id;

    -- Check if employee exists
    IF current_salary IS NULL THEN
        SET result_message = 'Employee not found';
    ELSE
        -- Calculate new salary
        SET new_salary = current_salary * (1 + raise_percent / 100);

        -- Check salary cap
        IF new_salary > 200000 THEN
            SET new_salary = 200000;
            SET result_message = 'Raise applied, capped at 200000';
        ELSE
            SET result_message = 'Raise applied successfully';
        END IF;

        -- Update salary
        UPDATE employees
        SET salary = new_salary
        WHERE employee_id = emp_id;

        -- Log the change
        INSERT INTO salary_history (employee_id, old_salary, new_salary, change_date)
        VALUES (emp_id, current_salary, new_salary, CURDATE());
    END IF;
END //
DELIMITER ;

-- Usage
CALL give_raise(101, 10, @message);
SELECT @message;
```

### Procedure with Loop

```sql
DELIMITER //
CREATE PROCEDURE apply_annual_raise()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_id INT;
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE raise_percent DECIMAL(5,2);

    -- Cursor to iterate through employees
    DECLARE emp_cursor CURSOR FOR
        SELECT employee_id, salary FROM employees WHERE status = 'active';

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN emp_cursor;

    read_loop: LOOP
        FETCH emp_cursor INTO emp_id, emp_salary;

        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Calculate raise based on salary tier
        IF emp_salary < 60000 THEN
            SET raise_percent = 5.0;
        ELSEIF emp_salary < 80000 THEN
            SET raise_percent = 4.0;
        ELSE
            SET raise_percent = 3.0;
        END IF;

        -- Apply raise
        UPDATE employees
        SET salary = salary * (1 + raise_percent / 100)
        WHERE employee_id = emp_id;
    END LOOP;

    CLOSE emp_cursor;

    SELECT 'Annual raises applied successfully' AS result;
END //
DELIMITER ;
```

### PostgreSQL Syntax

```sql
-- PostgreSQL uses different syntax
CREATE OR REPLACE FUNCTION get_employee_count(dept_id INT)
RETURNS INT AS $$
DECLARE
    emp_count INT;
BEGIN
    SELECT COUNT(*) INTO emp_count
    FROM employees
    WHERE department_id = dept_id;

    RETURN emp_count;
END;
$$ LANGUAGE plpgsql;

-- Call the function
SELECT get_employee_count(1);

-- PostgreSQL procedure (PostgreSQL 11+)
CREATE OR REPLACE PROCEDURE give_raise_pg(
    emp_id INT,
    raise_percent DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE employees
    SET salary = salary * (1 + raise_percent / 100)
    WHERE employee_id = emp_id;

    COMMIT;
END;
$$;

-- Call the procedure
CALL give_raise_pg(101, 10);
```

---

## Triggers (트리거)

### Concept (개념)
A trigger (트리거) is a special type of stored procedure that automatically executes when a specified database event occurs (INSERT, UPDATE, DELETE). Triggers are useful for enforcing business rules, maintaining audit logs, and ensuring data integrity.

### Trigger Timing and Events

```
Trigger Timing:
+------------+----------------------------------------+
| BEFORE     | Execute before the triggering event    |
|            | Can modify NEW values or cancel action |
+------------+----------------------------------------+
| AFTER      | Execute after the triggering event     |
|            | Typically for logging/auditing         |
+------------+----------------------------------------+

Trigger Events:
+------------+----------------------------------------+
| INSERT     | Fire when new row is inserted          |
| UPDATE     | Fire when existing row is updated      |
| DELETE     | Fire when row is deleted               |
+------------+----------------------------------------+

Row vs Statement Level:
+---------------+----------------------------------------+
| FOR EACH ROW  | Execute once per affected row          |
| FOR EACH STMT | Execute once per statement (PostgreSQL)|
+---------------+----------------------------------------+
```

### Trigger Flow Diagram

```
User executes: UPDATE employees SET salary = 90000 WHERE employee_id = 101

                                    +-------------------+
                                    | BEFORE UPDATE     |
                                    | Trigger executes  |
                                    | (can modify NEW)  |
                                    +-------------------+
                                            ↓
                                    +-------------------+
                                    | UPDATE operation  |
                                    | actually executes |
                                    +-------------------+
                                            ↓
                                    +-------------------+
                                    | AFTER UPDATE      |
                                    | Trigger executes  |
                                    | (audit logging)   |
                                    +-------------------+
```

### Example 1: Audit Trigger

```sql
-- Create audit table
CREATE TABLE employee_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT,
    action VARCHAR(10),
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create AFTER UPDATE trigger for auditing
DELIMITER //
CREATE TRIGGER trg_employee_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO employee_audit (
            employee_id,
            action,
            old_salary,
            new_salary,
            changed_by
        )
        VALUES (
            NEW.employee_id,
            'UPDATE',
            OLD.salary,
            NEW.salary,
            CURRENT_USER()
        );
    END IF;
END //
DELIMITER ;

-- Test the trigger
UPDATE employees SET salary = 100000 WHERE employee_id = 101;

-- Check audit log
SELECT * FROM employee_audit;

/*
Result:
| audit_id | employee_id | action | old_salary | new_salary | changed_by   | changed_at          |
|----------|-------------|--------|------------|------------|--------------|---------------------|
| 1        | 101         | UPDATE | 95000.00   | 100000.00  | root@local   | 2024-01-15 10:30:00 |
*/
```

### Example 2: Data Validation Trigger

```sql
-- BEFORE INSERT trigger to validate data
DELIMITER //
CREATE TRIGGER trg_validate_employee
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Validate email format
    IF NEW.email NOT LIKE '%@%.%' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid email format';
    END IF;

    -- Validate salary range
    IF NEW.salary < 30000 OR NEW.salary > 300000 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary must be between 30000 and 300000';
    END IF;

    -- Set default status if not provided
    IF NEW.status IS NULL THEN
        SET NEW.status = 'active';
    END IF;

    -- Set hire_date to today if not provided
    IF NEW.hire_date IS NULL THEN
        SET NEW.hire_date = CURDATE();
    END IF;
END //
DELIMITER ;

-- Test: This will fail
INSERT INTO employees (employee_id, name, email, department_id, salary)
VALUES (106, 'Test User', 'invalid-email', 1, 50000);
-- Error: Invalid email format

-- Test: This will succeed
INSERT INTO employees (employee_id, name, email, department_id, salary)
VALUES (106, 'Test User', 'test@company.com', 1, 50000);
```

### Example 3: Automatic Update Trigger

```sql
-- Create table with updated_at column
ALTER TABLE employees ADD COLUMN updated_at TIMESTAMP;

-- Trigger to automatically update timestamp
DELIMITER //
CREATE TRIGGER trg_employee_updated
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    SET NEW.updated_at = CURRENT_TIMESTAMP;
END //
DELIMITER ;

-- Now every UPDATE automatically sets updated_at
UPDATE employees SET salary = 95000 WHERE employee_id = 101;
-- updated_at is automatically set to current time
```

### Example 4: Cascading Trigger

```sql
-- When department budget changes, log it
DELIMITER //
CREATE TRIGGER trg_department_budget_change
AFTER UPDATE ON departments
FOR EACH ROW
BEGIN
    IF OLD.budget != NEW.budget THEN
        -- Log the budget change
        INSERT INTO budget_change_log (
            department_id,
            old_budget,
            new_budget,
            change_date
        )
        VALUES (
            NEW.department_id,
            OLD.budget,
            NEW.budget,
            NOW()
        );

        -- Notify if significant change (> 20%)
        IF ABS(NEW.budget - OLD.budget) / OLD.budget > 0.20 THEN
            INSERT INTO notifications (message, created_at)
            VALUES (
                CONCAT('Significant budget change for dept ', NEW.department_name),
                NOW()
            );
        END IF;
    END IF;
END //
DELIMITER ;
```

### PostgreSQL Trigger Syntax

```sql
-- PostgreSQL requires a trigger function
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO employee_audit (employee_id, action, old_salary, new_salary)
        VALUES (NEW.employee_id, 'UPDATE', OLD.salary, NEW.salary);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create the trigger using the function
CREATE TRIGGER trg_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

### Trigger Best Practices

```
+--------------------------------------------------+
| Trigger Best Practices                           |
+--------------------------------------------------+
| DO:                                              |
| [ ] Keep triggers simple and fast                |
| [ ] Use for auditing and data validation         |
| [ ] Document trigger behavior thoroughly         |
| [ ] Test triggers with various scenarios         |
|                                                  |
| DON'T:                                           |
| [ ] Put complex business logic in triggers       |
| [ ] Create triggers that call other triggers     |
|     (trigger chains are hard to debug)           |
| [ ] Use triggers for operations that should be   |
|     explicit in application code                 |
| [ ] Forget about triggers when debugging issues  |
+--------------------------------------------------+
```

---

## View vs Stored Procedure vs Trigger

```
+------------------+-----------------+-----------------+------------------+
| Feature          | View            | Stored Procedure| Trigger          |
+------------------+-----------------+-----------------+------------------+
| Purpose          | Simplify queries| Encapsulate     | Automatic        |
|                  | Hide complexity | business logic  | event response   |
+------------------+-----------------+-----------------+------------------+
| Invocation       | SELECT from     | CALL explicitly | Automatic on     |
|                  | view            |                 | DML events       |
+------------------+-----------------+-----------------+------------------+
| Parameters       | No              | Yes (IN/OUT)    | No (uses NEW/OLD)|
+------------------+-----------------+-----------------+------------------+
| Return value     | Result set      | Result set,     | None (modifies   |
|                  |                 | OUT params      | operation)       |
+------------------+-----------------+-----------------+------------------+
| Use case         | Data abstraction| Complex         | Auditing,        |
|                  | Security        | operations      | Validation       |
+------------------+-----------------+-----------------+------------------+
| Can modify data  | Sometimes       | Yes             | Yes              |
|                  | (simple views)  |                 |                  |
+------------------+-----------------+-----------------+------------------+
```

---

## Summary (요약)

| Topic | Korean | Key Points |
|-------|--------|------------|
| View | 뷰 | Virtual table based on query, no data storage |
| CREATE VIEW | 뷰 생성 | Encapsulate queries, provide security layer |
| Stored Procedure | 저장 프로시저 | Reusable SQL code with parameters and logic |
| Trigger | 트리거 | Automatic execution on INSERT/UPDATE/DELETE |

### Quick Reference

```sql
-- View
CREATE VIEW view_name AS SELECT ...;
SELECT * FROM view_name;
DROP VIEW view_name;

-- Stored Procedure (MySQL)
CREATE PROCEDURE proc_name(IN param INT)
BEGIN ... END;
CALL proc_name(value);
DROP PROCEDURE proc_name;

-- Trigger (MySQL)
CREATE TRIGGER trg_name
BEFORE/AFTER INSERT/UPDATE/DELETE ON table
FOR EACH ROW
BEGIN ... END;
DROP TRIGGER trg_name;
```

---
