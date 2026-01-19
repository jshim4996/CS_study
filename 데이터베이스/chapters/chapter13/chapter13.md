# Chapter 13. Denormalization (반정규화)

## Overview (개요)

Denormalization is the process of intentionally adding redundancy to a database schema to improve read performance. While normalization eliminates redundancy, denormalization strategically reintroduces it for practical performance gains.

반정규화는 읽기 성능을 향상시키기 위해 의도적으로 데이터베이스 스키마에 중복성을 추가하는 과정입니다. 정규화가 중복을 제거하는 반면, 반정규화는 실질적인 성능 향상을 위해 전략적으로 중복을 재도입합니다.

---

## 1. Why Denormalization is Needed (반정규화가 필요한 이유)

### 1.1 The Problem with Highly Normalized Databases (고도로 정규화된 데이터베이스의 문제점)

```
+-------------------+     +-------------------+     +-------------------+
|    customers      |     |     orders        |     |   order_items     |
+-------------------+     +-------------------+     +-------------------+
| customer_id (PK)  |<----| order_id (PK)     |<----| item_id (PK)      |
| name              |     | customer_id (FK)  |     | order_id (FK)     |
| email             |     | order_date        |     | product_id (FK)   |
| address           |     | status            |     | quantity          |
+-------------------+     +-------------------+     | unit_price        |
                                    |               +-------------------+
                                    |                        |
                                    v                        v
                          +-------------------+     +-------------------+
                          |    shipping       |     |    products       |
                          +-------------------+     +-------------------+
                          | shipping_id (PK)  |     | product_id (PK)   |
                          | order_id (FK)     |     | name              |
                          | address           |     | category_id (FK)  |
                          | carrier           |     | price             |
                          +-------------------+     +-------------------+
```

**Problems with too many JOINs:**
**너무 많은 JOIN의 문제점:**

```sql
-- Getting a complete order summary requires multiple JOINs
-- 완전한 주문 요약을 얻기 위해서는 여러 JOIN이 필요합니다

SELECT
    c.name AS customer_name,
    c.email,
    o.order_id,
    o.order_date,
    p.name AS product_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS line_total,
    s.carrier,
    cat.category_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
LEFT JOIN shipping s ON o.order_id = s.order_id
WHERE o.order_id = 12345;

-- This query requires 5-6 table JOINs!
-- 이 쿼리는 5-6개 테이블 JOIN이 필요합니다!
```

### 1.2 When to Consider Denormalization (반정규화를 고려해야 할 때)

```
+-----------------------------------------------------------------------+
|                 DENORMALIZATION DECISION MATRIX                        |
|                    반정규화 결정 매트릭스                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Consider Denormalization When:                                      |
|   반정규화를 고려해야 할 때:                                            |
|                                                                       |
|   [✓] Read operations significantly outnumber writes                  |
|       읽기 작업이 쓰기 작업보다 현저히 많을 때                           |
|                                                                       |
|   [✓] Complex JOINs are causing performance bottlenecks               |
|       복잡한 JOIN이 성능 병목을 일으킬 때                               |
|                                                                       |
|   [✓] Frequently accessed data spans multiple tables                  |
|       자주 접근하는 데이터가 여러 테이블에 걸쳐 있을 때                   |
|                                                                       |
|   [✓] Aggregation queries are run frequently                          |
|       집계 쿼리가 자주 실행될 때                                        |
|                                                                       |
|   [✓] Response time is critical (real-time applications)              |
|       응답 시간이 중요할 때 (실시간 애플리케이션)                         |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   Avoid Denormalization When:                                         |
|   반정규화를 피해야 할 때:                                              |
|                                                                       |
|   [✗] Data changes frequently                                         |
|       데이터가 자주 변경될 때                                           |
|                                                                       |
|   [✗] Data integrity is paramount                                     |
|       데이터 무결성이 가장 중요할 때                                     |
|                                                                       |
|   [✗] Storage space is limited                                        |
|       저장 공간이 제한적일 때                                           |
|                                                                       |
|   [✗] Application can tolerate slower reads                           |
|       애플리케이션이 느린 읽기를 허용할 수 있을 때                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. Denormalization Techniques (반정규화 기법)

### 2.1 Adding Redundant Columns (중복 컬럼 추가)

Store frequently accessed data from related tables directly in the main table.
관련 테이블에서 자주 접근하는 데이터를 메인 테이블에 직접 저장합니다.

```
BEFORE (Normalized):                      AFTER (Denormalized):
정규화된 상태:                             반정규화된 상태:

+------------------+                      +------------------------+
|     orders       |                      |        orders          |
+------------------+                      +------------------------+
| order_id (PK)    |                      | order_id (PK)          |
| customer_id (FK) |---> customers        | customer_id (FK)       |
| order_date       |                      | customer_name (RED)    | <-- Redundant
| status           |                      | customer_email (RED)   | <-- Redundant
+------------------+                      | order_date             |
                                          | status                 |
                                          +------------------------+
```

```sql
-- Before: Need JOIN to get customer info
-- 이전: 고객 정보를 얻기 위해 JOIN 필요
SELECT o.order_id, o.order_date, c.name, c.email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_id = 12345;

-- After: Direct access without JOIN
-- 이후: JOIN 없이 직접 접근
SELECT order_id, order_date, customer_name, customer_email
FROM orders
WHERE order_id = 12345;
```

**Implementation:**
**구현:**

```sql
-- Add redundant columns
-- 중복 컬럼 추가
ALTER TABLE orders
ADD COLUMN customer_name VARCHAR(100),
ADD COLUMN customer_email VARCHAR(100);

-- Populate existing data
-- 기존 데이터 채우기
UPDATE orders o
SET customer_name = (SELECT name FROM customers c WHERE c.customer_id = o.customer_id),
    customer_email = (SELECT email FROM customers c WHERE c.customer_id = o.customer_id);

-- Create trigger to maintain consistency
-- 일관성 유지를 위한 트리거 생성
CREATE TRIGGER update_order_customer_info
AFTER UPDATE ON customers
FOR EACH ROW
BEGIN
    UPDATE orders
    SET customer_name = NEW.name,
        customer_email = NEW.email
    WHERE customer_id = NEW.customer_id;
END;
```

### 2.2 Adding Derived/Calculated Columns (파생/계산 컬럼 추가)

Store pre-calculated values instead of computing them at query time.
쿼리 시점에 계산하는 대신 미리 계산된 값을 저장합니다.

```
+------------------------------------------------------------------+
|                      DERIVED COLUMNS EXAMPLE                      |
|                        파생 컬럼 예시                               |
+------------------------------------------------------------------+

BEFORE:                                 AFTER:
+-------------------+                   +------------------------+
|      orders       |                   |        orders          |
+-------------------+                   +------------------------+
| order_id          |                   | order_id               |
| customer_id       |                   | customer_id            |
| order_date        |                   | order_date             |
+-------------------+                   | total_amount (CALC)    | <-- Derived
        |                               | item_count (CALC)      | <-- Derived
        v                               +------------------------+
+-------------------+
|   order_items     |
+-------------------+
| item_id           |
| order_id          |
| quantity          |
| unit_price        |
+-------------------+
```

```sql
-- Before: Calculate total every time
-- 이전: 매번 총액 계산
SELECT
    o.order_id,
    SUM(oi.quantity * oi.unit_price) AS total_amount,
    COUNT(*) AS item_count
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = 12345
GROUP BY o.order_id;

-- After: Pre-calculated values
-- 이후: 미리 계산된 값
SELECT order_id, total_amount, item_count
FROM orders
WHERE order_id = 12345;
```

**Maintaining derived columns with triggers:**
**트리거로 파생 컬럼 유지:**

```sql
-- Trigger to update order totals when items change
-- 항목 변경 시 주문 총액을 업데이트하는 트리거

DELIMITER //

CREATE TRIGGER update_order_totals_insert
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE orders
    SET total_amount = (
            SELECT SUM(quantity * unit_price)
            FROM order_items
            WHERE order_id = NEW.order_id
        ),
        item_count = (
            SELECT COUNT(*)
            FROM order_items
            WHERE order_id = NEW.order_id
        )
    WHERE order_id = NEW.order_id;
END //

CREATE TRIGGER update_order_totals_delete
AFTER DELETE ON order_items
FOR EACH ROW
BEGIN
    UPDATE orders
    SET total_amount = COALESCE((
            SELECT SUM(quantity * unit_price)
            FROM order_items
            WHERE order_id = OLD.order_id
        ), 0),
        item_count = (
            SELECT COUNT(*)
            FROM order_items
            WHERE order_id = OLD.order_id
        )
    WHERE order_id = OLD.order_id;
END //

DELIMITER ;
```

### 2.3 Table Merging (테이블 병합)

Combine frequently joined tables into a single table.
자주 조인되는 테이블들을 하나의 테이블로 병합합니다.

```
BEFORE (1:1 Relationship):              AFTER (Merged):
1:1 관계:                               병합 후:

+------------------+                    +---------------------------+
|    employees     |                    |        employees          |
+------------------+                    +---------------------------+
| emp_id (PK)      |                    | emp_id (PK)               |
| name             |                    | name                      |
| department_id    |                    | department_id             |
+------------------+                    | address                   |
        |                               | phone                     |
        | 1:1                           | emergency_contact         |
        |                               | bank_account              |
        v                               +---------------------------+
+------------------+
| employee_details |
+------------------+
| emp_id (PK/FK)   |
| address          |
| phone            |
| emergency_contact|
| bank_account     |
+------------------+
```

```sql
-- Before: Always need to JOIN for complete employee info
-- 이전: 완전한 직원 정보를 위해 항상 JOIN 필요
SELECT e.emp_id, e.name, ed.address, ed.phone
FROM employees e
JOIN employee_details ed ON e.emp_id = ed.emp_id
WHERE e.emp_id = 101;

-- After: Single table access
-- 이후: 단일 테이블 접근
SELECT emp_id, name, address, phone
FROM employees
WHERE emp_id = 101;
```

### 2.4 Adding Summary Tables (요약 테이블 추가)

Create pre-aggregated tables for reporting queries.
리포팅 쿼리를 위한 미리 집계된 테이블을 생성합니다.

```
+------------------------------------------------------------------+
|                    SUMMARY TABLE ARCHITECTURE                      |
|                      요약 테이블 아키텍처                            |
+------------------------------------------------------------------+

  Transactional Tables              Summary Tables (Denormalized)
  트랜잭션 테이블                    요약 테이블 (반정규화)

  +---------------+
  |    orders     |                 +-------------------------+
  +---------------+                 |   daily_sales_summary   |
  | order_id      |    ------->     +-------------------------+
  | customer_id   |    Aggregate    | summary_date            |
  | order_date    |                 | total_orders            |
  | status        |                 | total_revenue           |
  +---------------+                 | avg_order_value         |
         |                          | unique_customers        |
         v                          +-------------------------+
  +---------------+
  | order_items   |                 +-------------------------+
  +---------------+                 | monthly_product_summary |
  | item_id       |    ------->     +-------------------------+
  | order_id      |    Aggregate    | year_month              |
  | product_id    |                 | product_id              |
  | quantity      |                 | units_sold              |
  | unit_price    |                 | total_revenue           |
  +---------------+                 | return_count            |
                                    +-------------------------+
```

```sql
-- Create summary table
-- 요약 테이블 생성
CREATE TABLE daily_sales_summary (
    summary_date DATE PRIMARY KEY,
    total_orders INT DEFAULT 0,
    total_revenue DECIMAL(15,2) DEFAULT 0,
    avg_order_value DECIMAL(10,2) DEFAULT 0,
    unique_customers INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Populate summary table (run daily via scheduled job)
-- 요약 테이블 채우기 (예약된 작업으로 매일 실행)
INSERT INTO daily_sales_summary (summary_date, total_orders, total_revenue, avg_order_value, unique_customers)
SELECT
    DATE(order_date) AS summary_date,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue,
    AVG(total_amount) AS avg_order_value,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
WHERE DATE(order_date) = CURDATE() - INTERVAL 1 DAY
GROUP BY DATE(order_date)
ON DUPLICATE KEY UPDATE
    total_orders = VALUES(total_orders),
    total_revenue = VALUES(total_revenue),
    avg_order_value = VALUES(avg_order_value),
    unique_customers = VALUES(unique_customers);

-- Fast reporting query
-- 빠른 리포팅 쿼리
SELECT * FROM daily_sales_summary
WHERE summary_date BETWEEN '2024-01-01' AND '2024-01-31';
```

### 2.5 Table Partitioning/Splitting (테이블 분할)

Split large tables based on access patterns.
접근 패턴에 따라 큰 테이블을 분할합니다.

```
+------------------------------------------------------------------+
|                  HORIZONTAL PARTITIONING (Sharding)                |
|                     수평 분할 (샤딩)                                |
+------------------------------------------------------------------+

              Original Table                    Partitioned Tables
              원본 테이블                        분할된 테이블

        +-------------------+           +-------------------+
        |   transactions    |           | transactions_2023 |
        +-------------------+           +-------------------+
        | trans_id          |           | (data from 2023)  |
        | amount            |           +-------------------+
        | trans_date        |   --->
        | customer_id       |           +-------------------+
        | (millions of rows)|           | transactions_2024 |
        +-------------------+           +-------------------+
                                        | (data from 2024)  |
                                        +-------------------+

                                        +-------------------+
                                        | transactions_2025 |
                                        +-------------------+
                                        | (data from 2025)  |
                                        +-------------------+
```

```sql
-- MySQL Partitioning Example
-- MySQL 파티셔닝 예시
CREATE TABLE transactions (
    trans_id BIGINT AUTO_INCREMENT,
    amount DECIMAL(15,2),
    trans_date DATE,
    customer_id INT,
    description VARCHAR(255),
    PRIMARY KEY (trans_id, trans_date)
)
PARTITION BY RANGE (YEAR(trans_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Query automatically uses relevant partition
-- 쿼리가 자동으로 관련 파티션 사용
SELECT * FROM transactions
WHERE trans_date BETWEEN '2024-01-01' AND '2024-12-31';
-- Only scans p2024 partition!
-- p2024 파티션만 스캔!
```

```
+------------------------------------------------------------------+
|                    VERTICAL PARTITIONING                           |
|                        수직 분할                                    |
+------------------------------------------------------------------+

      Original Table                      Split Tables
      원본 테이블                         분할된 테이블

  +--------------------+            +------------------+
  |      products      |            |  products_core   |
  +--------------------+            +------------------+
  | product_id         |            | product_id (PK)  |
  | name               |   --->     | name             |
  | price              |            | price            |
  | category_id        |            | category_id      |
  | description (TEXT) |            +------------------+
  | specifications     |
  | images (BLOB)      |            +------------------+
  | manual (BLOB)      |            | products_content |
  +--------------------+            +------------------+
                                    | product_id (FK)  |
  Frequently accessed               | description      |
  + Rarely accessed                 | specifications   |
                                    | images           |
                                    | manual           |
                                    +------------------+

                                    Infrequently accessed
```

---

## 3. Normalization vs Denormalization Trade-offs (정규화 vs 반정규화 트레이드오프)

### 3.1 Comparison Table (비교 표)

```
+-----------------------------------------------------------------------+
|           NORMALIZATION vs DENORMALIZATION COMPARISON                  |
|              정규화 vs 반정규화 비교                                    |
+-----------------------------------------------------------------------+
|  Aspect            |  Normalization        |  Denormalization          |
|  측면              |  정규화               |  반정규화                  |
+-----------------------------------------------------------------------+
|  Data Redundancy   |  Minimal (최소)       |  Intentional (의도적)      |
|  데이터 중복       |                       |                           |
+--------------------+-----------------------+---------------------------+
|  Storage Space     |  Less (적음)          |  More (많음)              |
|  저장 공간         |                       |                           |
+--------------------+-----------------------+---------------------------+
|  Read Performance  |  Slower (느림)        |  Faster (빠름)            |
|  읽기 성능         |  (due to JOINs)       |  (fewer JOINs)            |
+--------------------+-----------------------+---------------------------+
|  Write Performance |  Faster (빠름)        |  Slower (느림)            |
|  쓰기 성능         |  (single update)      |  (multiple updates)       |
+--------------------+-----------------------+---------------------------+
|  Data Integrity    |  Higher (높음)        |  Lower (낮음)             |
|  데이터 무결성     |  (no anomalies)       |  (possible anomalies)     |
+--------------------+-----------------------+---------------------------+
|  Query Complexity  |  Complex (복잡)       |  Simple (단순)            |
|  쿼리 복잡성       |  (many JOINs)         |  (fewer JOINs)            |
+--------------------+-----------------------+---------------------------+
|  Maintenance       |  Easier (쉬움)        |  Harder (어려움)          |
|  유지보수          |  (single source)      |  (sync required)          |
+--------------------+-----------------------+---------------------------+
|  Best For          |  OLTP Systems         |  OLAP/Reporting           |
|  적합한 용도       |  트랜잭션 처리        |  분석/리포팅               |
+-----------------------------------------------------------------------+
```

### 3.2 Read vs Write Trade-off Visualization (읽기 vs 쓰기 트레이드오프 시각화)

```
Performance
성능
    ^
    |
    |    Normalized              Denormalized
    |    정규화                   반정규화
    |
100%|    ████                    ░░░░
    |    ████  Write             ░░░░  Write
    |    ████  Performance       ░░░░  Performance
    |    ████                    ░░░░
 50%|----████--------------------░░░░----------------
    |    ░░░░                    ████
    |    ░░░░  Read              ████  Read
    |    ░░░░  Performance       ████  Performance
    |    ░░░░                    ████
  0%+-------------------------------------->
         OLTP                    OLAP
         (Many Writes)           (Many Reads)
```

### 3.3 Decision Framework (의사 결정 프레임워크)

```
+-----------------------------------------------------------------------+
|                    DENORMALIZATION DECISION FLOW                       |
|                      반정규화 결정 흐름도                               |
+-----------------------------------------------------------------------+

                          Start
                          시작
                            |
                            v
              +---------------------------+
              | Is read performance       |
              | a critical issue?         |
              | 읽기 성능이 중요한 문제인가? |
              +---------------------------+
                     |           |
                    YES         NO
                     |           |
                     v           v
        +-------------------+   Keep Normalized
        | Read:Write ratio  |   정규화 유지
        | > 10:1?           |
        | 읽기:쓰기 비율     |
        | > 10:1?           |
        +-------------------+
               |        |
              YES      NO
               |        |
               v        v
    +----------------+  Consider Index
    | Can you afford |  Optimization First
    | data sync      |  인덱스 최적화를
    | complexity?    |  먼저 고려
    | 데이터 동기화   |
    | 복잡성 감당?   |
    +----------------+
          |      |
         YES    NO
          |      |
          v      v
    Denormalize   Use Caching
    반정규화      캐싱 사용
```

### 3.4 Hybrid Approach Example (하이브리드 접근법 예시)

```sql
-- Practical Example: E-commerce Order System
-- 실제 예시: 전자상거래 주문 시스템

-- Normalized tables for data integrity (OLTP)
-- 데이터 무결성을 위한 정규화 테이블 (OLTP)
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered') DEFAULT 'pending',
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

-- Denormalized columns for frequent reads (Hybrid)
-- 자주 읽기 위한 반정규화 컬럼 (하이브리드)
ALTER TABLE orders
ADD COLUMN customer_name VARCHAR(100),
ADD COLUMN total_amount DECIMAL(12,2) DEFAULT 0,
ADD COLUMN item_count INT DEFAULT 0;

-- Denormalized summary table for reporting (OLAP)
-- 리포팅을 위한 반정규화 요약 테이블 (OLAP)
CREATE TABLE order_analytics (
    date DATE PRIMARY KEY,
    total_orders INT,
    total_revenue DECIMAL(15,2),
    avg_order_value DECIMAL(10,2),
    top_product_id INT,
    new_customers INT,
    returning_customers INT
);

-- Application determines which to query based on use case
-- 애플리케이션이 사용 사례에 따라 어떤 것을 쿼리할지 결정

-- For order creation (use normalized)
-- 주문 생성 시 (정규화 사용)
INSERT INTO orders (customer_id) VALUES (123);

-- For order display (use denormalized columns)
-- 주문 표시 시 (반정규화 컬럼 사용)
SELECT order_id, customer_name, total_amount, item_count
FROM orders WHERE order_id = 456;

-- For reporting dashboard (use summary table)
-- 리포팅 대시보드 시 (요약 테이블 사용)
SELECT * FROM order_analytics
WHERE date BETWEEN '2024-01-01' AND '2024-01-31';
```

---

## 4. Best Practices (모범 사례)

### 4.1 Guidelines for Denormalization (반정규화 지침)

```
+-----------------------------------------------------------------------+
|                   DENORMALIZATION BEST PRACTICES                       |
|                     반정규화 모범 사례                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. START NORMALIZED                                                  |
|     정규화에서 시작하기                                                 |
|     - Always begin with a properly normalized schema                  |
|     - 항상 적절히 정규화된 스키마로 시작                                 |
|                                                                       |
|  2. MEASURE FIRST                                                     |
|     먼저 측정하기                                                      |
|     - Identify actual performance bottlenecks                         |
|     - 실제 성능 병목 지점 식별                                          |
|     - Don't denormalize prematurely                                   |
|     - 조기 반정규화 금지                                                |
|                                                                       |
|  3. DOCUMENT REDUNDANCY                                               |
|     중복성 문서화                                                      |
|     - Keep track of what data is duplicated where                     |
|     - 어떤 데이터가 어디에 중복되어 있는지 추적                          |
|                                                                       |
|  4. AUTOMATE SYNC                                                     |
|     동기화 자동화                                                      |
|     - Use triggers, stored procedures, or application logic           |
|     - 트리거, 저장 프로시저 또는 애플리케이션 로직 사용                   |
|                                                                       |
|  5. MONITOR DATA CONSISTENCY                                          |
|     데이터 일관성 모니터링                                              |
|     - Regularly verify denormalized data matches source               |
|     - 반정규화 데이터가 소스와 일치하는지 정기적으로 확인                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### 4.2 Data Consistency Verification (데이터 일관성 검증)

```sql
-- Periodic consistency check query
-- 주기적 일관성 검사 쿼리
SELECT
    o.order_id,
    o.customer_name AS denormalized_name,
    c.name AS actual_name,
    CASE WHEN o.customer_name = c.name THEN 'OK' ELSE 'MISMATCH' END AS status
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.customer_name != c.name;

-- Fix inconsistencies
-- 불일치 수정
UPDATE orders o
JOIN customers c ON o.customer_id = c.customer_id
SET o.customer_name = c.name
WHERE o.customer_name != c.name;
```

---

## Summary (요약)

```
+-----------------------------------------------------------------------+
|                         CHAPTER 13 SUMMARY                             |
|                          제13장 요약                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|  KEY CONCEPTS (핵심 개념):                                             |
|                                                                       |
|  1. Denormalization Purpose (반정규화 목적)                            |
|     - Trade data redundancy for read performance                      |
|     - 읽기 성능을 위해 데이터 중복성 교환                                |
|                                                                       |
|  2. Techniques (기법)                                                  |
|     - Redundant columns (중복 컬럼)                                    |
|     - Derived/calculated columns (파생/계산 컬럼)                       |
|     - Table merging (테이블 병합)                                       |
|     - Summary tables (요약 테이블)                                      |
|     - Table partitioning (테이블 분할)                                  |
|                                                                       |
|  3. Trade-offs (트레이드오프)                                           |
|     - Faster reads vs Slower writes                                   |
|     - 빠른 읽기 vs 느린 쓰기                                            |
|     - More storage vs Better performance                              |
|     - 더 많은 저장공간 vs 더 나은 성능                                   |
|     - Simpler queries vs Complex maintenance                          |
|     - 단순한 쿼리 vs 복잡한 유지보수                                     |
|                                                                       |
|  4. Best Practice (모범 사례)                                          |
|     - Start normalized, denormalize selectively based on data         |
|     - 정규화에서 시작, 데이터 기반으로 선택적 반정규화                    |
|                                                                       |
+-----------------------------------------------------------------------+

  REMEMBER: "Normalize until it hurts, denormalize until it works"
  기억하세요: "아플 때까지 정규화하고, 작동할 때까지 반정규화하세요"
```
