# Chapter 08. Other SQL Features (기타 SQL)

## Table of Contents (목차)
1. [UNION and UNION ALL](#1-union-and-union-all)
2. [CASE WHEN Expression](#2-case-when-expression)
3. [String Functions](#3-string-functions-문자열-함수)
4. [Date Functions](#4-date-functions-날짜-함수)
5. [Type Conversion Functions](#5-type-conversion-functions-형변환-함수)
6. [Summary](#6-summary-요약)

---

## 1. UNION and UNION ALL

### Concept (개념)

**UNION** combines the result sets of two or more SELECT statements into a single result set.

**UNION**은 두 개 이상의 SELECT 문의 결과를 하나의 결과 집합으로 결합합니다.

```
+------------------+     +------------------+
|   Query 1        |     |   Query 2        |
| +----+------+    |     | +----+------+    |
| | id | name |    |     | | id | name |    |
| +----+------+    |     | +----+------+    |
| | 1  | Kim  |    |     | | 3  | Park |    |
| | 2  | Lee  |    |     | | 4  | Choi |    |
| +----+------+    |     | +----+------+    |
+------------------+     +------------------+
         |                        |
         +----------+-------------+
                    |
                    v  UNION
         +------------------+
         |   Combined       |
         | +----+------+    |
         | | id | name |    |
         | +----+------+    |
         | | 1  | Kim  |    |
         | | 2  | Lee  |    |
         | | 3  | Park |    |
         | | 4  | Choi |    |
         | +----+------+    |
         +------------------+
```

### UNION vs UNION ALL

| Feature | UNION | UNION ALL |
|---------|-------|-----------|
| Duplicate Removal (중복 제거) | Yes (제거함) | No (제거 안함) |
| Performance (성능) | Slower (느림) | Faster (빠름) |
| Use Case (사용 시점) | Need unique rows | Keep all rows |

### Syntax and Examples (구문 및 예제)

```sql
-- Sample Tables (예제 테이블)
-- employees_seoul: Seoul branch employees
-- employees_busan: Busan branch employees

-- UNION: Removes duplicates (중복 제거)
SELECT name, department FROM employees_seoul
UNION
SELECT name, department FROM employees_busan;

-- UNION ALL: Keeps duplicates (중복 유지)
SELECT name, department FROM employees_seoul
UNION ALL
SELECT name, department FROM employees_busan;
```

### Rules for UNION (UNION 규칙)

1. **Same number of columns** (동일한 열 개수)
2. **Compatible data types** (호환되는 데이터 타입)
3. **Column names from first SELECT** (열 이름은 첫 번째 SELECT에서 가져옴)

```sql
-- Correct: Same number of columns
SELECT id, name, salary FROM full_time_employees
UNION
SELECT id, name, hourly_rate * 160 FROM part_time_employees;

-- Error: Different number of columns
SELECT id, name, salary FROM full_time_employees
UNION
SELECT id, name FROM part_time_employees;  -- ERROR!
```

### Practical Example (실전 예제)

```sql
-- Combine active and archived orders
-- 활성 주문과 보관된 주문 결합
SELECT order_id, customer_name, order_date, 'Active' AS status
FROM active_orders
WHERE order_date >= '2024-01-01'
UNION ALL
SELECT order_id, customer_name, order_date, 'Archived' AS status
FROM archived_orders
WHERE order_date >= '2024-01-01'
ORDER BY order_date DESC;
```

---

## 2. CASE WHEN Expression

### Concept (개념)

**CASE WHEN** provides conditional logic in SQL, similar to if-else statements in programming languages.

**CASE WHEN**은 프로그래밍 언어의 if-else 문과 유사하게 SQL에서 조건부 로직을 제공합니다.

```
+--------------------+
|    CASE WHEN       |
+--------------------+
         |
    +----+----+
    |         |
    v         v
Searched    Simple
  CASE       CASE
```

### Syntax Types (구문 유형)

#### Simple CASE (단순 CASE)
```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE default_result
END
```

#### Searched CASE (검색 CASE)
```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

### Examples (예제)

```sql
-- Simple CASE: Grade to description
-- 단순 CASE: 등급을 설명으로 변환
SELECT
    student_name,
    grade,
    CASE grade
        WHEN 'A' THEN 'Excellent (우수)'
        WHEN 'B' THEN 'Good (양호)'
        WHEN 'C' THEN 'Average (보통)'
        WHEN 'D' THEN 'Below Average (미흡)'
        WHEN 'F' THEN 'Fail (불합격)'
        ELSE 'Unknown (알 수 없음)'
    END AS grade_description
FROM students;

-- Searched CASE: Score to grade
-- 검색 CASE: 점수를 등급으로 변환
SELECT
    student_name,
    score,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 70 THEN 'C'
        WHEN score >= 60 THEN 'D'
        ELSE 'F'
    END AS grade
FROM exam_results;
```

### CASE in Different Clauses (다양한 절에서의 CASE)

```sql
-- In SELECT clause (SELECT 절)
SELECT
    product_name,
    CASE WHEN stock > 100 THEN 'High'
         WHEN stock > 50 THEN 'Medium'
         ELSE 'Low'
    END AS stock_level
FROM products;

-- In ORDER BY clause (ORDER BY 절)
SELECT * FROM orders
ORDER BY
    CASE status
        WHEN 'Urgent' THEN 1
        WHEN 'Normal' THEN 2
        WHEN 'Low' THEN 3
    END;

-- In WHERE clause (WHERE 절)
SELECT * FROM employees
WHERE
    CASE
        WHEN department = 'Sales' THEN salary > 50000
        WHEN department = 'IT' THEN salary > 60000
        ELSE salary > 40000
    END;

-- In aggregate functions (집계 함수 내)
SELECT
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) AS completed_count,
    COUNT(CASE WHEN status = 'Pending' THEN 1 END) AS pending_count,
    COUNT(CASE WHEN status = 'Cancelled' THEN 1 END) AS cancelled_count
FROM orders;
```

### Pivot Table using CASE (CASE를 이용한 피벗 테이블)

```sql
-- Monthly sales pivot
-- 월별 매출 피벗
SELECT
    product_name,
    SUM(CASE WHEN MONTH(sale_date) = 1 THEN amount ELSE 0 END) AS Jan,
    SUM(CASE WHEN MONTH(sale_date) = 2 THEN amount ELSE 0 END) AS Feb,
    SUM(CASE WHEN MONTH(sale_date) = 3 THEN amount ELSE 0 END) AS Mar
FROM sales
GROUP BY product_name;
```

---

## 3. String Functions (문자열 함수)

### Common String Functions Overview (주요 문자열 함수 개요)

```
+-------------------+----------------------------------------+
| Function          | Description (설명)                      |
+-------------------+----------------------------------------+
| CONCAT            | Concatenate strings (문자열 연결)        |
| SUBSTRING/SUBSTR  | Extract substring (부분 문자열 추출)     |
| TRIM/LTRIM/RTRIM  | Remove spaces (공백 제거)               |
| UPPER/LOWER       | Change case (대소문자 변환)             |
| LENGTH/LEN        | String length (문자열 길이)             |
| REPLACE           | Replace substring (문자열 대체)         |
| LEFT/RIGHT        | Extract from left/right (왼쪽/오른쪽 추출)|
| LPAD/RPAD         | Pad string (문자열 패딩)                |
+-------------------+----------------------------------------+
```

### CONCAT - String Concatenation (문자열 연결)

```sql
-- Basic CONCAT
SELECT CONCAT('Hello', ' ', 'World');  -- Result: 'Hello World'

-- CONCAT with columns
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;

-- CONCAT_WS: Concat with separator (구분자로 연결)
SELECT CONCAT_WS('-', '2024', '01', '15');  -- Result: '2024-01-15'

-- Building email addresses
SELECT CONCAT(LOWER(first_name), '.', LOWER(last_name), '@company.com') AS email
FROM employees;
```

### SUBSTRING/SUBSTR - Extract Substring (부분 문자열 추출)

```sql
-- Syntax: SUBSTRING(string, start, length)
-- 구문: SUBSTRING(문자열, 시작위치, 길이)

SELECT SUBSTRING('Hello World', 1, 5);   -- Result: 'Hello'
SELECT SUBSTRING('Hello World', 7);      -- Result: 'World' (to end)
SELECT SUBSTRING('Hello World', -5);     -- Result: 'World' (from end)

-- Extract area code from phone number
-- 전화번호에서 지역번호 추출
SELECT
    phone_number,
    SUBSTRING(phone_number, 1, 3) AS area_code
FROM customers;

-- Extract year from date string
SELECT SUBSTRING('2024-01-15', 1, 4) AS year;  -- Result: '2024'
```

### TRIM, LTRIM, RTRIM - Remove Spaces (공백 제거)

```sql
-- TRIM: Remove leading and trailing spaces
-- 앞뒤 공백 제거
SELECT TRIM('   Hello World   ');     -- Result: 'Hello World'

-- LTRIM: Remove leading spaces (왼쪽 공백 제거)
SELECT LTRIM('   Hello');             -- Result: 'Hello'

-- RTRIM: Remove trailing spaces (오른쪽 공백 제거)
SELECT RTRIM('Hello   ');             -- Result: 'Hello'

-- TRIM specific characters (특정 문자 제거)
SELECT TRIM(BOTH '*' FROM '***Hello***');  -- Result: 'Hello'
SELECT TRIM(LEADING '0' FROM '00123');     -- Result: '123'

-- Clean up user input
UPDATE users
SET email = TRIM(LOWER(email))
WHERE email != TRIM(LOWER(email));
```

### UPPER and LOWER - Case Conversion (대소문자 변환)

```sql
SELECT UPPER('hello');  -- Result: 'HELLO'
SELECT LOWER('HELLO');  -- Result: 'hello'

-- Case-insensitive search (대소문자 구분 없는 검색)
SELECT * FROM products
WHERE LOWER(product_name) LIKE '%apple%';

-- Standardize data format
UPDATE customers
SET country_code = UPPER(country_code);
```

### LENGTH/LEN - String Length (문자열 길이)

```sql
-- LENGTH (MySQL, PostgreSQL)
SELECT LENGTH('Hello');  -- Result: 5

-- LEN (SQL Server)
SELECT LEN('Hello');     -- Result: 5

-- Find long descriptions
SELECT product_name, LENGTH(description) AS desc_length
FROM products
WHERE LENGTH(description) > 500;

-- Validate input length
SELECT
    username,
    CASE
        WHEN LENGTH(username) < 3 THEN 'Too short'
        WHEN LENGTH(username) > 20 THEN 'Too long'
        ELSE 'Valid'
    END AS validation
FROM users;
```

### REPLACE - String Replacement (문자열 대체)

```sql
-- Basic replacement
SELECT REPLACE('Hello World', 'World', 'SQL');  -- Result: 'Hello SQL'

-- Remove characters
SELECT REPLACE('010-1234-5678', '-', '');  -- Result: '01012345678'

-- Update data with replacement
UPDATE products
SET description = REPLACE(description, 'old_term', 'new_term');

-- Multiple replacements
SELECT REPLACE(REPLACE(phone, '-', ''), ' ', '') AS clean_phone
FROM contacts;
```

### LEFT and RIGHT - Extract from Ends (양끝에서 추출)

```sql
-- LEFT: Extract from beginning (앞에서 추출)
SELECT LEFT('Hello World', 5);   -- Result: 'Hello'

-- RIGHT: Extract from end (뒤에서 추출)
SELECT RIGHT('Hello World', 5);  -- Result: 'World'

-- Get file extension
SELECT RIGHT(filename, LENGTH(filename) - LOCATE('.', filename)) AS extension
FROM files;

-- Get first initial
SELECT LEFT(first_name, 1) AS initial FROM employees;
```

### LPAD and RPAD - Padding (패딩)

```sql
-- LPAD: Left padding (왼쪽 채우기)
SELECT LPAD('123', 5, '0');   -- Result: '00123'
SELECT LPAD('42', 5, '*');    -- Result: '***42'

-- RPAD: Right padding (오른쪽 채우기)
SELECT RPAD('Hello', 10, '-');  -- Result: 'Hello-----'

-- Format employee ID
SELECT LPAD(CAST(emp_id AS CHAR), 6, '0') AS formatted_id
FROM employees;
-- emp_id: 42 -> '000042'
```

---

## 4. Date Functions (날짜 함수)

### Date Functions Overview (날짜 함수 개요)

```
+---------------------+------------------------------------------+
| Function            | Description (설명)                        |
+---------------------+------------------------------------------+
| NOW()/GETDATE()     | Current date and time (현재 날짜와 시간)   |
| CURDATE()/CURRENT_DATE | Current date (현재 날짜)               |
| DATE_FORMAT/FORMAT  | Format date (날짜 포맷)                   |
| DATEDIFF            | Difference between dates (날짜 차이)      |
| DATE_ADD/DATEADD    | Add interval to date (날짜에 간격 추가)    |
| DATE_SUB            | Subtract interval (날짜에서 간격 빼기)     |
| YEAR/MONTH/DAY      | Extract date parts (날짜 부분 추출)        |
| EXTRACT             | Extract component (구성 요소 추출)         |
+---------------------+------------------------------------------+
```

### NOW() - Current Date and Time (현재 날짜와 시간)

```sql
-- MySQL
SELECT NOW();           -- Result: '2024-01-15 14:30:25'
SELECT CURDATE();       -- Result: '2024-01-15'
SELECT CURTIME();       -- Result: '14:30:25'

-- SQL Server
SELECT GETDATE();       -- Result: '2024-01-15 14:30:25.123'
SELECT CURRENT_TIMESTAMP;

-- PostgreSQL
SELECT NOW();
SELECT CURRENT_DATE;
SELECT CURRENT_TIME;

-- Record creation timestamp
INSERT INTO orders (customer_id, order_date, status)
VALUES (1001, NOW(), 'Pending');
```

### DATE_FORMAT - Formatting Dates (날짜 포맷팅)

```sql
-- MySQL DATE_FORMAT
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d');          -- '2024-01-15'
SELECT DATE_FORMAT(NOW(), '%Y년 %m월 %d일');     -- '2024년 01월 15일'
SELECT DATE_FORMAT(NOW(), '%W, %M %d, %Y');     -- 'Monday, January 15, 2024'
SELECT DATE_FORMAT(NOW(), '%H:%i:%s');          -- '14:30:25'

-- Common format specifiers (주요 포맷 지정자)
-- %Y: 4-digit year (4자리 연도)
-- %y: 2-digit year (2자리 연도)
-- %m: Month (01-12) (월)
-- %d: Day (01-31) (일)
-- %H: Hour 24h (시 24시간)
-- %h: Hour 12h (시 12시간)
-- %i: Minutes (분)
-- %s: Seconds (초)
-- %W: Weekday name (요일 이름)
-- %M: Month name (월 이름)

-- SQL Server FORMAT
SELECT FORMAT(GETDATE(), 'yyyy-MM-dd');
SELECT FORMAT(GETDATE(), 'MMMM dd, yyyy');
```

### DATEDIFF - Date Difference (날짜 차이)

```sql
-- MySQL: DATEDIFF(date1, date2) returns days
SELECT DATEDIFF('2024-01-15', '2024-01-01');  -- Result: 14

-- Calculate age in days
SELECT
    name,
    DATEDIFF(CURDATE(), birth_date) AS days_old
FROM employees;

-- SQL Server: DATEDIFF(unit, start, end)
SELECT DATEDIFF(day, '2024-01-01', '2024-01-15');    -- 14 days
SELECT DATEDIFF(month, '2024-01-01', '2024-06-15');  -- 5 months
SELECT DATEDIFF(year, '2000-01-01', '2024-01-01');   -- 24 years

-- Find overdue orders (연체된 주문 찾기)
SELECT order_id, order_date,
       DATEDIFF(CURDATE(), order_date) AS days_since_order
FROM orders
WHERE status = 'Pending'
  AND DATEDIFF(CURDATE(), order_date) > 7;
```

### DATE_ADD and DATE_SUB - Date Arithmetic (날짜 연산)

```sql
-- MySQL DATE_ADD
SELECT DATE_ADD('2024-01-15', INTERVAL 10 DAY);     -- '2024-01-25'
SELECT DATE_ADD('2024-01-15', INTERVAL 2 MONTH);    -- '2024-03-15'
SELECT DATE_ADD('2024-01-15', INTERVAL 1 YEAR);     -- '2025-01-15'
SELECT DATE_ADD('2024-01-15', INTERVAL -7 DAY);     -- '2024-01-08'

-- DATE_SUB (subtract)
SELECT DATE_SUB('2024-01-15', INTERVAL 1 WEEK);     -- '2024-01-08'
SELECT DATE_SUB(NOW(), INTERVAL 30 DAY);            -- 30 days ago

-- Calculate due dates (마감일 계산)
SELECT
    order_id,
    order_date,
    DATE_ADD(order_date, INTERVAL 14 DAY) AS due_date
FROM orders;

-- SQL Server DATEADD
SELECT DATEADD(day, 10, '2024-01-15');
SELECT DATEADD(month, 2, GETDATE());

-- Find records from last 30 days
SELECT * FROM logs
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);
```

### YEAR, MONTH, DAY - Extract Date Parts (날짜 부분 추출)

```sql
SELECT YEAR('2024-01-15');    -- Result: 2024
SELECT MONTH('2024-01-15');   -- Result: 1
SELECT DAY('2024-01-15');     -- Result: 15

-- Also available
SELECT HOUR('14:30:25');      -- Result: 14
SELECT MINUTE('14:30:25');    -- Result: 30
SELECT SECOND('14:30:25');    -- Result: 25

-- DAYNAME and MONTHNAME
SELECT DAYNAME('2024-01-15'); -- 'Monday'
SELECT MONTHNAME('2024-01-15'); -- 'January'

-- Group sales by year and month
-- 연도와 월별로 매출 그룹화
SELECT
    YEAR(sale_date) AS year,
    MONTH(sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY YEAR(sale_date), MONTH(sale_date)
ORDER BY year, month;
```

### EXTRACT - Extract Components (구성 요소 추출)

```sql
-- Standard SQL EXTRACT
SELECT EXTRACT(YEAR FROM '2024-01-15');     -- 2024
SELECT EXTRACT(MONTH FROM '2024-01-15');    -- 1
SELECT EXTRACT(DAY FROM '2024-01-15');      -- 15
SELECT EXTRACT(HOUR FROM '14:30:25');       -- 14
SELECT EXTRACT(QUARTER FROM '2024-06-15');  -- 2

-- PostgreSQL additional extracts
SELECT EXTRACT(DOW FROM '2024-01-15');      -- Day of week (0-6)
SELECT EXTRACT(DOY FROM '2024-01-15');      -- Day of year
SELECT EXTRACT(WEEK FROM '2024-01-15');     -- Week number

-- Quarterly report
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(QUARTER FROM order_date) AS quarter,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(QUARTER FROM order_date);
```

---

## 5. Type Conversion Functions (형변환 함수)

### Conversion Functions Overview (형변환 함수 개요)

```
+------------------+----------------------------------------+
| Function         | Description (설명)                      |
+------------------+----------------------------------------+
| CAST             | Convert to specified type (타입 변환)   |
| CONVERT          | Convert with style (스타일 포함 변환)   |
| COALESCE         | First non-NULL value (첫 번째 비NULL값) |
| NULLIF           | NULL if equal (같으면 NULL)             |
| IFNULL/ISNULL    | Replace NULL (NULL 대체)                |
+------------------+----------------------------------------+
```

### CAST - Type Conversion (타입 변환)

```sql
-- Basic syntax: CAST(expression AS data_type)
-- 기본 구문: CAST(표현식 AS 데이터타입)

-- String to Integer
SELECT CAST('123' AS SIGNED);           -- Result: 123
SELECT CAST('123' AS INT);              -- SQL Server

-- Integer to String
SELECT CAST(123 AS CHAR(10));           -- Result: '123'
SELECT CAST(123 AS VARCHAR(10));        -- SQL Server

-- String to Date
SELECT CAST('2024-01-15' AS DATE);      -- Result: DATE '2024-01-15'

-- Float to Integer (truncates)
SELECT CAST(3.7 AS SIGNED);             -- Result: 3

-- Date to String
SELECT CAST(CURDATE() AS CHAR);         -- Result: '2024-01-15'

-- Decimal precision
SELECT CAST(10/3 AS DECIMAL(10, 2));    -- Result: 3.33
```

### CONVERT - Conversion with Style (스타일 포함 변환)

```sql
-- MySQL CONVERT
SELECT CONVERT('123', SIGNED);
SELECT CONVERT('2024-01-15', DATE);

-- SQL Server CONVERT with style
-- Date to String with various formats
SELECT CONVERT(VARCHAR, GETDATE(), 101);  -- '01/15/2024' (US)
SELECT CONVERT(VARCHAR, GETDATE(), 103);  -- '15/01/2024' (British)
SELECT CONVERT(VARCHAR, GETDATE(), 104);  -- '15.01.2024' (German)
SELECT CONVERT(VARCHAR, GETDATE(), 111);  -- '2024/01/15' (Japan)
SELECT CONVERT(VARCHAR, GETDATE(), 112);  -- '20240115' (ISO)

-- String to Date
SELECT CONVERT(DATE, '01/15/2024', 101);  -- US format
SELECT CONVERT(DATE, '15/01/2024', 103);  -- British format
```

### COALESCE - First Non-NULL Value (첫 번째 비NULL 값)

```sql
-- Returns first non-NULL value
-- 첫 번째 NULL이 아닌 값 반환
SELECT COALESCE(NULL, NULL, 'Hello', 'World');  -- Result: 'Hello'

-- Replace NULL with default (NULL을 기본값으로 대체)
SELECT
    product_name,
    COALESCE(discount_price, regular_price) AS final_price
FROM products;

-- Multiple fallbacks (다중 대체값)
SELECT
    customer_name,
    COALESCE(mobile_phone, home_phone, office_phone, 'No phone') AS contact_phone
FROM customers;

-- Handle NULL in calculations
SELECT
    order_id,
    COALESCE(shipping_fee, 0) + COALESCE(handling_fee, 0) AS extra_fees
FROM orders;
```

### NULLIF - Return NULL if Equal (같으면 NULL 반환)

```sql
-- Returns NULL if two expressions are equal
-- 두 표현식이 같으면 NULL 반환
SELECT NULLIF(10, 10);    -- Result: NULL
SELECT NULLIF(10, 20);    -- Result: 10

-- Avoid division by zero (0으로 나누기 방지)
SELECT
    total_sales,
    total_sales / NULLIF(total_orders, 0) AS avg_order_value
FROM sales_summary;
-- If total_orders is 0, returns NULL instead of error

-- Replace empty strings with NULL
SELECT NULLIF(TRIM(user_input), '') AS cleaned_input;
```

### IFNULL / ISNULL - NULL Replacement (NULL 대체)

```sql
-- MySQL IFNULL
SELECT IFNULL(NULL, 'Default');        -- Result: 'Default'
SELECT IFNULL('Value', 'Default');     -- Result: 'Value'

-- SQL Server ISNULL
SELECT ISNULL(NULL, 'Default');        -- Result: 'Default'

-- Practical examples
SELECT
    employee_name,
    IFNULL(commission, 0) AS commission,
    salary + IFNULL(commission, 0) AS total_pay
FROM employees;

-- Display 'N/A' for missing data
SELECT
    product_name,
    IFNULL(description, 'N/A') AS description
FROM products;
```

### Implicit vs Explicit Conversion (암시적 vs 명시적 변환)

```
+------------------+------------------------------------------+
| Implicit         | Explicit                                 |
| (암시적)          | (명시적)                                   |
+------------------+------------------------------------------+
| Automatic        | Using CAST/CONVERT                       |
| May cause errors | Controlled conversion                    |
| Less predictable | More predictable                         |
+------------------+------------------------------------------+
```

```sql
-- Implicit conversion (may cause issues)
-- 암시적 변환 (문제 발생 가능)
SELECT '10' + 5;           -- MySQL: 15 (string converted to int)
SELECT '10' + '5';         -- MySQL: 15 (both converted to int)

-- Explicit conversion (recommended)
-- 명시적 변환 (권장)
SELECT CAST('10' AS SIGNED) + 5;     -- Clearly: 15
SELECT CONCAT(CAST(10 AS CHAR), '5'); -- Clearly: '105'
```

### Conversion Best Practices (형변환 모범 사례)

```sql
-- 1. Always use explicit conversion in comparisons
-- 비교에서는 항상 명시적 변환 사용
SELECT * FROM orders
WHERE order_date = CAST('2024-01-15' AS DATE);

-- 2. Handle NULL before conversion
-- 변환 전 NULL 처리
SELECT CAST(COALESCE(quantity, 0) AS DECIMAL(10,2)) AS qty
FROM inventory;

-- 3. Use appropriate precision for decimals
-- 소수점에 적절한 정밀도 사용
SELECT CAST(price AS DECIMAL(10, 2)) AS formatted_price
FROM products;

-- 4. Validate data before conversion
-- 변환 전 데이터 검증
SELECT
    CASE
        WHEN value REGEXP '^[0-9]+$' THEN CAST(value AS SIGNED)
        ELSE NULL
    END AS numeric_value
FROM raw_data;
```

---

## 6. Summary (요약)

### Key Points (핵심 포인트)

| Feature | Key Takeaway |
|---------|--------------|
| **UNION** | Combines result sets; UNION removes duplicates, UNION ALL keeps them |
| **CASE WHEN** | Conditional logic in SQL; useful for data transformation |
| **String Functions** | CONCAT, SUBSTRING, TRIM for text manipulation |
| **Date Functions** | NOW, DATE_FORMAT, DATEDIFF for temporal operations |
| **Type Conversion** | CAST, CONVERT for explicit type changes; COALESCE for NULL handling |

### Quick Reference (빠른 참조)

```sql
-- UNION pattern
SELECT ... FROM table1
UNION [ALL]
SELECT ... FROM table2;

-- CASE pattern
CASE
    WHEN condition THEN result
    ELSE default
END

-- Common string operations
CONCAT(str1, str2)
SUBSTRING(str, start, len)
TRIM(str)
UPPER(str) / LOWER(str)

-- Common date operations
NOW() / CURDATE()
DATE_FORMAT(date, format)
DATEDIFF(date1, date2)
DATE_ADD(date, INTERVAL n unit)

-- Type conversion
CAST(expr AS type)
COALESCE(val1, val2, ...)
IFNULL(val, default)
```

### Practice Questions (연습 문제)

1. Write a query to combine customer lists from two regions and remove duplicates.
2. Use CASE WHEN to categorize products by price range.
3. Create a full name column by concatenating first and last names.
4. Calculate the number of days between order date and ship date.
5. Convert a string date to a DATE type and format it for display.

---

*End of Chapter 08*
