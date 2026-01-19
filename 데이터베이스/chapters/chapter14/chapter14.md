# Chapter 14. Transaction Basics (트랜잭션 기초)

## Overview (개요)

A transaction is a sequence of database operations that are treated as a single logical unit of work. Transactions ensure data integrity and consistency even in the face of system failures or concurrent access.

트랜잭션은 단일 논리적 작업 단위로 취급되는 데이터베이스 작업의 시퀀스입니다. 트랜잭션은 시스템 장애나 동시 접근 상황에서도 데이터 무결성과 일관성을 보장합니다.

---

## 1. Transaction Concept (트랜잭션 개념)

### 1.1 What is a Transaction? (트랜잭션이란?)

```
+-----------------------------------------------------------------------+
|                     TRANSACTION CONCEPT                                |
|                      트랜잭션 개념                                      |
+-----------------------------------------------------------------------+

  A Transaction is an "all or nothing" operation
  트랜잭션은 "전부 아니면 전무" 작업입니다

  Example: Bank Transfer (예시: 은행 이체)

  Account A: $1000                 Account B: $500
  계좌 A: $1000                    계좌 B: $500

  Transfer $200 from A to B
  A에서 B로 $200 이체

  +-------------------------------------------------------------------+
  |  BEGIN TRANSACTION                                                 |
  |  트랜잭션 시작                                                      |
  |                                                                    |
  |    Step 1: Deduct $200 from Account A                              |
  |    단계 1: 계좌 A에서 $200 차감                                      |
  |            A: $1000 -> $800                                        |
  |                                                                    |
  |    Step 2: Add $200 to Account B                                   |
  |    단계 2: 계좌 B에 $200 추가                                        |
  |            B: $500 -> $700                                         |
  |                                                                    |
  |  COMMIT (if both succeed) or ROLLBACK (if any fails)              |
  |  커밋 (둘 다 성공 시) 또는 롤백 (하나라도 실패 시)                     |
  +-------------------------------------------------------------------+

  Result: Either BOTH operations complete, or NEITHER does
  결과: 두 작업이 모두 완료되거나, 둘 다 완료되지 않음
```

### 1.2 Transaction Boundaries (트랜잭션 경계)

```sql
-- Explicit Transaction Example
-- 명시적 트랜잭션 예시

-- MySQL / MariaDB
START TRANSACTION;  -- or BEGIN

UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';

COMMIT;  -- Make changes permanent (변경사항 영구 적용)

-- PostgreSQL
BEGIN;

UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';

COMMIT;

-- SQL Server
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';

COMMIT TRANSACTION;
```

---

## 2. ACID Properties (ACID 속성)

The four fundamental properties that guarantee transaction reliability.
트랜잭션 신뢰성을 보장하는 네 가지 기본 속성입니다.

```
+-----------------------------------------------------------------------+
|                        ACID PROPERTIES                                 |
|                         ACID 속성                                      |
+-----------------------------------------------------------------------+

     A            C              I              D
     |            |              |              |
     v            v              v              v
  +------+    +--------+    +---------+    +----------+
  |  A   |    |   C    |    |    I    |    |    D     |
  |      |    |        |    |         |    |          |
  |Atom- |    |Consis- |    |Isola-   |    |Dura-     |
  |icity |    |tency   |    |tion     |    |bility    |
  |      |    |        |    |         |    |          |
  |원자성 |    |일관성   |    |격리성    |    |지속성     |
  +------+    +--------+    +---------+    +----------+
     |            |              |              |
     v            v              v              v
  All or      Valid         Concurrent    Committed
  Nothing     State         Isolation     = Permanent
  전부 또는   유효한         동시 실행     커밋된 것은
  전무       상태           격리          영구적
```

### 2.1 Atomicity (원자성)

All operations in a transaction succeed, or all fail together.
트랜잭션의 모든 작업이 성공하거나, 모두 함께 실패합니다.

```
+-----------------------------------------------------------------------+
|                          ATOMICITY                                     |
|                           원자성                                       |
+-----------------------------------------------------------------------+

  Transaction as an "Atom" - indivisible unit
  트랜잭션은 "원자" - 분할 불가능한 단위

  SCENARIO 1: Success                 SCENARIO 2: Failure
  시나리오 1: 성공                     시나리오 2: 실패

  BEGIN                               BEGIN
    |                                   |
    v                                   v
  [Deduct A] ---> OK                  [Deduct A] ---> OK
    |                                   |
    v                                   v
  [Add B] -----> OK                   [Add B] -----> ERROR!
    |                                   |
    v                                   v
  COMMIT                              ROLLBACK
    |                                   |
    v                                   v
  Changes Saved                       All Changes Undone
  변경사항 저장                        모든 변경사항 취소

  A: $800, B: $700                    A: $1000, B: $500
  (Transfer complete)                  (No partial state)
  (이체 완료)                          (부분 상태 없음)
```

```sql
-- Atomicity in Action
-- 원자성 실제 예시

START TRANSACTION;

-- Step 1: Check sufficient balance
-- 단계 1: 충분한 잔액 확인
SELECT balance INTO @balance FROM accounts WHERE account_id = 'A' FOR UPDATE;

-- Step 2: Conditional execution
-- 단계 2: 조건부 실행
IF @balance >= 200 THEN
    UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
    UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';
    COMMIT;
ELSE
    ROLLBACK;  -- Undo any partial changes
               -- 모든 부분 변경 취소
END IF;
```

### 2.2 Consistency (일관성)

A transaction brings the database from one valid state to another.
트랜잭션은 데이터베이스를 하나의 유효한 상태에서 다른 유효한 상태로 전환합니다.

```
+-----------------------------------------------------------------------+
|                         CONSISTENCY                                    |
|                           일관성                                       |
+-----------------------------------------------------------------------+

  Database maintains all defined rules (constraints)
  데이터베이스는 정의된 모든 규칙(제약조건)을 유지

  Before Transaction          After Transaction
  트랜잭션 전                  트랜잭션 후

  +------------------+        +------------------+
  | VALID STATE      |        | VALID STATE      |
  |------------------|        |------------------|
  | A + B = $1500    |  --->  | A + B = $1500    |
  | A >= 0           |        | A >= 0           |
  | B >= 0           |        | B >= 0           |
  | Constraints OK   |        | Constraints OK   |
  +------------------+        +------------------+
       $1000 + $500                $800 + $700

  Total money in system remains constant!
  시스템 내 총 금액은 일정하게 유지!
```

```sql
-- Consistency enforced by constraints
-- 제약조건으로 일관성 적용

CREATE TABLE accounts (
    account_id VARCHAR(10) PRIMARY KEY,
    balance DECIMAL(10,2) NOT NULL,
    -- Constraint: Balance cannot be negative
    -- 제약조건: 잔액은 음수가 될 수 없음
    CONSTRAINT chk_balance CHECK (balance >= 0)
);

-- This transaction maintains consistency
-- 이 트랜잭션은 일관성을 유지
START TRANSACTION;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';
COMMIT;

-- This would violate consistency (if A has only $100)
-- 이것은 일관성을 위반 (A가 $100만 있는 경우)
START TRANSACTION;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
-- ERROR: Check constraint 'chk_balance' is violated
-- 에러: 체크 제약조건 'chk_balance' 위반
ROLLBACK;
```

### 2.3 Isolation (격리성)

Concurrent transactions execute as if they were sequential.
동시 트랜잭션은 순차적으로 실행되는 것처럼 동작합니다.

```
+-----------------------------------------------------------------------+
|                          ISOLATION                                     |
|                           격리성                                       |
+-----------------------------------------------------------------------+

  Each transaction is isolated from others
  각 트랜잭션은 다른 트랜잭션으로부터 격리됨

  Without Isolation:                  With Isolation:
  격리 없이:                          격리와 함께:

  Transaction 1    Transaction 2      Transaction 1    Transaction 2
       |               |                   |               |
       v               v                   v               |
    Read A=$1000    Read A=$1000        Read A=$1000      | (waiting)
       |               |                   |               | (대기 중)
       v               v                   v               |
    A=$1000-200     A=$1000-200         A=$800            |
       |               |                   |               |
       v               v                   v               v
    Write A=$800    Write A=$800        COMMIT          Read A=$800
       |               |                   |               |
       v               v                                   v
    Lost Update!                                        A=$600
    업데이트 손실!                                        |
                                                          v
    A=$800 (wrong!)                                     COMMIT
    A=$800 (잘못됨!)
                                                        A=$600 (correct!)
                                                        A=$600 (정확!)
```

```sql
-- Isolation with locking
-- 락을 사용한 격리

-- Transaction 1
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A' FOR UPDATE;
-- Row is now locked (행이 이제 잠김)
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
COMMIT;
-- Lock released (락 해제)

-- Transaction 2 (concurrent) - must wait for Transaction 1
-- 트랜잭션 2 (동시 실행) - 트랜잭션 1을 기다려야 함
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A' FOR UPDATE;
-- Waits until Transaction 1 commits... (트랜잭션 1이 커밋될 때까지 대기...)
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
COMMIT;
```

### 2.4 Durability (지속성)

Once committed, changes survive system failures.
일단 커밋되면, 변경사항은 시스템 장애에서도 유지됩니다.

```
+-----------------------------------------------------------------------+
|                          DURABILITY                                    |
|                           지속성                                       |
+-----------------------------------------------------------------------+

  Committed = Permanent (even after crash)
  커밋됨 = 영구적 (충돌 후에도)

  Timeline:
  타임라인:

  [Transaction]--->[COMMIT]--->[Success Response]
                      |
                      v
              [Write to Disk]
              [WAL/Redo Log]
                      |
                      |
       ~~~~ SYSTEM CRASH ~~~~
       ~~~~ 시스템 충돌 ~~~~
                      |
                      v
              [System Restart]
              [시스템 재시작]
                      |
                      v
              [Recovery Process]
              [복구 프로세스]
                      |
                      v
              [Replay from WAL]
              [WAL에서 재생]
                      |
                      v
              [Data Restored!]
              [데이터 복원됨!]

  WAL = Write-Ahead Logging
  WAL = 선행 기록 로깅
```

```sql
-- Durability is automatic after COMMIT
-- 지속성은 COMMIT 후 자동으로 적용

START TRANSACTION;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';
COMMIT;  -- At this point, changes are GUARANTEED to survive crashes
         -- 이 시점에서, 변경사항은 충돌에서도 유지가 보장됨

-- The database uses various mechanisms:
-- 데이터베이스는 다양한 메커니즘 사용:
-- - Write-Ahead Logging (WAL) - 선행 기록 로깅
-- - Transaction logs - 트랜잭션 로그
-- - Checkpoints - 체크포인트
-- - Redo logs - 리두 로그
```

---

## 3. Transaction States (트랜잭션 상태)

```
+-----------------------------------------------------------------------+
|                     TRANSACTION STATE DIAGRAM                          |
|                      트랜잭션 상태 다이어그램                            |
+-----------------------------------------------------------------------+

                            +----------+
                            |  Active  |
                            |  활성    |
                            +----+-----+
                                 |
                    +------------+------------+
                    |                         |
                    v                         v
           +----------------+        +----------------+
           | Partially      |        |   Failed       |
           | Committed      |        |   실패         |
           | 부분 커밋       |        +-------+--------+
           +-------+--------+                |
                   |                         |
                   v                         v
           +----------------+        +----------------+
           |  Committed     |        |   Aborted      |
           |  커밋됨        |        |   중단됨        |
           +----------------+        +----------------+


  State Descriptions:
  상태 설명:

  1. Active (활성)
     - Transaction is executing operations
     - 트랜잭션이 작업을 실행 중

  2. Partially Committed (부분 커밋)
     - All operations executed, waiting for commit
     - 모든 작업 실행 완료, 커밋 대기 중

  3. Committed (커밋됨)
     - Changes permanently saved
     - 변경사항이 영구적으로 저장됨

  4. Failed (실패)
     - Error occurred during execution
     - 실행 중 오류 발생

  5. Aborted (중단됨)
     - Transaction rolled back, database restored
     - 트랜잭션 롤백됨, 데이터베이스 복원됨
```

```sql
-- Transaction states in practice
-- 실제 트랜잭션 상태

-- State: Active (상태: 활성)
START TRANSACTION;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
-- Still active, changes not yet visible to others
-- 아직 활성, 변경사항이 아직 다른 트랜잭션에 보이지 않음

-- State: Partially Committed (상태: 부분 커밋)
UPDATE accounts SET balance = balance + 200 WHERE account_id = 'B';
-- All operations done, ready to commit
-- 모든 작업 완료, 커밋 준비됨

-- State: Committed (상태: 커밋됨)
COMMIT;
-- Changes are now permanent
-- 변경사항이 이제 영구적

-- Alternative: State transitions to Failed/Aborted
-- 대안: 실패/중단으로 상태 전환
START TRANSACTION;
UPDATE accounts SET balance = balance - 200 WHERE account_id = 'A';
-- Error occurs here (e.g., constraint violation)
-- 여기서 오류 발생 (예: 제약조건 위반)
-- State: Failed (상태: 실패)
ROLLBACK;
-- State: Aborted (상태: 중단됨)
```

---

## 4. COMMIT and ROLLBACK (커밋과 롤백)

### 4.1 COMMIT (커밋)

Makes all changes in the current transaction permanent.
현재 트랜잭션의 모든 변경사항을 영구적으로 만듭니다.

```
+-----------------------------------------------------------------------+
|                            COMMIT                                      |
|                             커밋                                       |
+-----------------------------------------------------------------------+

  COMMIT Process:
  COMMIT 프로세스:

  +-------------+     +-------------+     +-------------+     +--------+
  |  Execute    | --> | Write to    | --> |  Flush to   | --> | Return |
  |  Operations |     | Transaction |     |    Disk     |     | Success|
  |  작업 실행   |     |    Log      |     |  디스크에   |     | 성공   |
  +-------------+     | 트랜잭션 로그 |     |   플러시    |     | 반환   |
                      +-------------+     +-------------+     +--------+

  After COMMIT:
  COMMIT 후:
  - Changes are visible to other transactions
    변경사항이 다른 트랜잭션에서 보임
  - Changes survive system failures
    변경사항은 시스템 장애에서도 유지
  - Cannot be undone (except by new transaction)
    취소 불가 (새 트랜잭션으로만 가능)
```

```sql
-- COMMIT examples
-- COMMIT 예시

-- Explicit COMMIT
-- 명시적 COMMIT
START TRANSACTION;
INSERT INTO orders (customer_id, total) VALUES (1, 99.99);
INSERT INTO order_items (order_id, product_id, qty) VALUES (LAST_INSERT_ID(), 5, 2);
COMMIT;  -- Both inserts are now permanent
         -- 두 삽입이 이제 영구적

-- Auto-commit mode (default in many databases)
-- 자동 커밋 모드 (많은 데이터베이스의 기본값)
SET autocommit = 1;  -- Each statement is auto-committed
                     -- 각 명령문이 자동 커밋됨

INSERT INTO products (name, price) VALUES ('Widget', 19.99);
-- Automatically committed! (자동으로 커밋됨!)

-- Disable auto-commit for explicit control
-- 명시적 제어를 위해 자동 커밋 비활성화
SET autocommit = 0;
INSERT INTO products (name, price) VALUES ('Gadget', 29.99);
-- Not committed yet (아직 커밋되지 않음)
COMMIT;  -- Now committed (이제 커밋됨)
```

### 4.2 ROLLBACK (롤백)

Undoes all changes in the current transaction.
현재 트랜잭션의 모든 변경사항을 취소합니다.

```
+-----------------------------------------------------------------------+
|                           ROLLBACK                                     |
|                            롤백                                        |
+-----------------------------------------------------------------------+

  ROLLBACK Process:
  ROLLBACK 프로세스:

  +-------------+     +-------------+     +-------------+     +--------+
  |  Error or   | --> |  Read Undo  | --> |  Restore    | --> | Return |
  |  Explicit   |     |    Log      |     |  Original   |     | to     |
  |  ROLLBACK   |     | 언두 로그    |     |   State     |     | Caller |
  |  오류 또는   |     |   읽기      |     |  원래 상태   |     | 호출자 |
  |명시적 롤백   |     |             |     |   복원      |     | 반환   |
  +-------------+     +-------------+     +-------------+     +--------+

  After ROLLBACK:
  ROLLBACK 후:
  - All changes in transaction are undone
    트랜잭션의 모든 변경사항 취소
  - Database returns to state before BEGIN
    데이터베이스가 BEGIN 이전 상태로 복원
  - Transaction ends
    트랜잭션 종료
```

```sql
-- ROLLBACK examples
-- ROLLBACK 예시

-- Manual ROLLBACK
-- 수동 ROLLBACK
START TRANSACTION;
DELETE FROM customers WHERE customer_id = 1;
-- Oops! Wrong customer! (이런! 잘못된 고객!)
ROLLBACK;  -- Undo the delete (삭제 취소)

-- ROLLBACK on error
-- 오류 시 ROLLBACK
START TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 'B';

-- Check if update was successful
-- 업데이트 성공 여부 확인
IF ROW_COUNT() = 0 THEN
    ROLLBACK;  -- Account B doesn't exist
               -- 계좌 B가 존재하지 않음
ELSE
    COMMIT;
END IF;

-- Using error handling (stored procedure)
-- 오류 처리 사용 (저장 프로시저)
DELIMITER //
CREATE PROCEDURE transfer_money(
    IN from_acc VARCHAR(10),
    IN to_acc VARCHAR(10),
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Transfer failed' AS result;
    END;

    START TRANSACTION;

    UPDATE accounts SET balance = balance - amount WHERE account_id = from_acc;
    UPDATE accounts SET balance = balance + amount WHERE account_id = to_acc;

    COMMIT;
    SELECT 'Transfer successful' AS result;
END //
DELIMITER ;
```

---

## 5. SAVEPOINT (세이브포인트)

### 5.1 Concept (개념)

Savepoints allow partial rollback within a transaction.
세이브포인트는 트랜잭션 내에서 부분적인 롤백을 가능하게 합니다.

```
+-----------------------------------------------------------------------+
|                          SAVEPOINT                                     |
|                         세이브포인트                                    |
+-----------------------------------------------------------------------+

  Transaction Timeline with Savepoints:
  세이브포인트가 있는 트랜잭션 타임라인:

  BEGIN
    |
    v
  [Operation 1] -----> Insert order
    |
    v
  SAVEPOINT sp1 -----> (checkpoint 1)
    |
    v
  [Operation 2] -----> Insert item 1
    |
    v
  SAVEPOINT sp2 -----> (checkpoint 2)
    |
    v
  [Operation 3] -----> Insert item 2 (ERROR!)
    |
    v
  ROLLBACK TO sp2 ---> Undo only operation 3
    |                  (작업 3만 취소)
    v
  [Operation 3b] ----> Insert item 2 (retry)
    |
    v
  COMMIT

  Result: Operations 1, 2, 3b saved; Operation 3 discarded
  결과: 작업 1, 2, 3b 저장됨; 작업 3 폐기됨
```

### 5.2 SAVEPOINT Syntax (SAVEPOINT 문법)

```sql
-- Create a savepoint
-- 세이브포인트 생성
SAVEPOINT savepoint_name;

-- Rollback to savepoint (undo changes after savepoint)
-- 세이브포인트로 롤백 (세이브포인트 이후 변경 취소)
ROLLBACK TO savepoint_name;
-- or
ROLLBACK TO SAVEPOINT savepoint_name;

-- Release savepoint (optional, frees resources)
-- 세이브포인트 해제 (선택적, 리소스 해제)
RELEASE SAVEPOINT savepoint_name;
```

### 5.3 Practical Example (실제 예시)

```sql
-- Complex order processing with savepoints
-- 세이브포인트를 사용한 복잡한 주문 처리

START TRANSACTION;

-- Step 1: Create order header
-- 단계 1: 주문 헤더 생성
INSERT INTO orders (customer_id, order_date, status)
VALUES (101, NOW(), 'pending');
SET @order_id = LAST_INSERT_ID();

SAVEPOINT after_order_created;

-- Step 2: Add first item
-- 단계 2: 첫 번째 항목 추가
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 1, 2, 29.99);

SAVEPOINT after_item1;

-- Step 3: Try to add second item (might fail due to stock)
-- 단계 3: 두 번째 항목 추가 시도 (재고로 인해 실패할 수 있음)
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 2, 1, 49.99);

-- If second item fails, rollback to after first item
-- 두 번째 항목 실패 시, 첫 번째 항목 이후로 롤백
-- ROLLBACK TO after_item1;

-- Continue with alternative item
-- 대체 항목으로 계속
-- INSERT INTO order_items (order_id, product_id, quantity, price)
-- VALUES (@order_id, 3, 1, 39.99);

SAVEPOINT after_items;

-- Step 4: Apply discount (might fail)
-- 단계 4: 할인 적용 (실패할 수 있음)
UPDATE orders SET discount = 10.00 WHERE order_id = @order_id;

-- If discount application fails, continue without discount
-- 할인 적용 실패 시, 할인 없이 계속
-- ROLLBACK TO after_items;

-- Final commit
-- 최종 커밋
COMMIT;

SELECT * FROM orders WHERE order_id = @order_id;
SELECT * FROM order_items WHERE order_id = @order_id;
```

### 5.4 Savepoint Visualization (세이브포인트 시각화)

```
+-----------------------------------------------------------------------+
|                    SAVEPOINT STACK BEHAVIOR                            |
|                    세이브포인트 스택 동작                               |
+-----------------------------------------------------------------------+

  Memory/Stack View:
  메모리/스택 뷰:

  BEGIN                      ROLLBACK TO sp2          After Rollback
  시작                       sp2로 롤백               롤백 후

  +-------------+            +-------------+          +-------------+
  |     sp3     | <-- top    |     sp3     | X        |             |
  +-------------+            +-------------+          +-------------+
  |     sp2     |            |     sp2     | <-- top  |     sp2     | <-- top
  +-------------+            +-------------+          +-------------+
  |     sp1     |            |     sp1     |          |     sp1     |
  +-------------+            +-------------+          +-------------+
  |   BEGIN     |            |   BEGIN     |          |   BEGIN     |
  +-------------+            +-------------+          +-------------+

  ROLLBACK TO sp2:
  - Removes sp3 and all changes after sp2
    sp3와 sp2 이후의 모든 변경 제거
  - sp2 becomes the current savepoint
    sp2가 현재 세이브포인트가 됨
  - sp1 and earlier work is preserved
    sp1과 그 이전 작업은 보존됨
```

---

## 6. Transaction Examples (트랜잭션 예시)

### 6.1 E-commerce Order Transaction (전자상거래 주문 트랜잭션)

```sql
-- Complete e-commerce transaction
-- 완전한 전자상거래 트랜잭션

DELIMITER //

CREATE PROCEDURE process_order(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_order_id INT,
    OUT p_message VARCHAR(100)
)
BEGIN
    DECLARE v_price DECIMAL(10,2);
    DECLARE v_stock INT;
    DECLARE v_total DECIMAL(10,2);

    -- Error handler
    -- 오류 처리기
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_order_id = NULL;
        SET p_message = 'Transaction failed - rolled back';
    END;

    START TRANSACTION;

    -- 1. Check and lock product stock
    -- 1. 제품 재고 확인 및 잠금
    SELECT price, stock INTO v_price, v_stock
    FROM products
    WHERE product_id = p_product_id
    FOR UPDATE;  -- Lock the row (행 잠금)

    -- 2. Verify sufficient stock
    -- 2. 충분한 재고 확인
    IF v_stock < p_quantity THEN
        ROLLBACK;
        SET p_order_id = NULL;
        SET p_message = 'Insufficient stock';
    ELSE
        -- 3. Calculate total
        -- 3. 총액 계산
        SET v_total = v_price * p_quantity;

        -- 4. Create order
        -- 4. 주문 생성
        INSERT INTO orders (customer_id, total_amount, status)
        VALUES (p_customer_id, v_total, 'confirmed');
        SET p_order_id = LAST_INSERT_ID();

        SAVEPOINT after_order;

        -- 5. Create order item
        -- 5. 주문 항목 생성
        INSERT INTO order_items (order_id, product_id, quantity, unit_price)
        VALUES (p_order_id, p_product_id, p_quantity, v_price);

        -- 6. Update stock
        -- 6. 재고 업데이트
        UPDATE products
        SET stock = stock - p_quantity
        WHERE product_id = p_product_id;

        -- 7. Commit if all successful
        -- 7. 모두 성공하면 커밋
        COMMIT;
        SET p_message = 'Order processed successfully';
    END IF;
END //

DELIMITER ;

-- Usage example
-- 사용 예시
CALL process_order(1, 101, 2, @order_id, @message);
SELECT @order_id AS order_id, @message AS message;
```

### 6.2 Bank Transfer with Full Error Handling (전체 오류 처리가 있는 은행 이체)

```sql
-- Bank transfer with comprehensive error handling
-- 포괄적인 오류 처리가 있는 은행 이체

DELIMITER //

CREATE PROCEDURE bank_transfer(
    IN from_account VARCHAR(20),
    IN to_account VARCHAR(20),
    IN transfer_amount DECIMAL(15,2),
    OUT result_message VARCHAR(200)
)
BEGIN
    DECLARE from_balance DECIMAL(15,2);
    DECLARE to_exists INT;

    -- Error handler for any SQL exception
    -- 모든 SQL 예외에 대한 오류 처리기
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1
            result_message = MESSAGE_TEXT;
        ROLLBACK;
        SET result_message = CONCAT('ERROR: ', result_message);
    END;

    -- Validate transfer amount
    -- 이체 금액 검증
    IF transfer_amount <= 0 THEN
        SET result_message = 'ERROR: Transfer amount must be positive';
    ELSE
        START TRANSACTION;

        -- Lock and check source account
        -- 출금 계좌 잠금 및 확인
        SELECT balance INTO from_balance
        FROM accounts
        WHERE account_number = from_account
        FOR UPDATE;

        IF from_balance IS NULL THEN
            ROLLBACK;
            SET result_message = 'ERROR: Source account not found';
        ELSEIF from_balance < transfer_amount THEN
            ROLLBACK;
            SET result_message = 'ERROR: Insufficient funds';
        ELSE
            -- Check destination account exists
            -- 입금 계좌 존재 확인
            SELECT COUNT(*) INTO to_exists
            FROM accounts
            WHERE account_number = to_account
            FOR UPDATE;

            IF to_exists = 0 THEN
                ROLLBACK;
                SET result_message = 'ERROR: Destination account not found';
            ELSE
                -- Perform transfer
                -- 이체 수행
                UPDATE accounts
                SET balance = balance - transfer_amount
                WHERE account_number = from_account;

                UPDATE accounts
                SET balance = balance + transfer_amount
                WHERE account_number = to_account;

                -- Log the transaction
                -- 트랜잭션 기록
                INSERT INTO transaction_log
                    (from_account, to_account, amount, trans_date)
                VALUES
                    (from_account, to_account, transfer_amount, NOW());

                COMMIT;
                SET result_message = CONCAT('SUCCESS: Transferred $',
                    transfer_amount, ' from ', from_account, ' to ', to_account);
            END IF;
        END IF;
    END IF;
END //

DELIMITER ;

-- Test the procedure
-- 프로시저 테스트
CALL bank_transfer('ACC001', 'ACC002', 500.00, @result);
SELECT @result;
```

---

## Summary (요약)

```
+-----------------------------------------------------------------------+
|                        CHAPTER 14 SUMMARY                              |
|                          제14장 요약                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|  KEY CONCEPTS (핵심 개념):                                             |
|                                                                       |
|  1. Transaction (트랜잭션)                                             |
|     - Logical unit of work (논리적 작업 단위)                           |
|     - All-or-nothing execution (전부 또는 전무 실행)                    |
|                                                                       |
|  2. ACID Properties (ACID 속성)                                        |
|     +-----------+----------------------------------------+            |
|     | Atomicity | All operations succeed or all fail     |            |
|     | 원자성    | 모든 작업이 성공하거나 모두 실패          |            |
|     +-----------+----------------------------------------+            |
|     | Consistency| Database remains in valid state       |            |
|     | 일관성     | 데이터베이스가 유효한 상태 유지          |            |
|     +-----------+----------------------------------------+            |
|     | Isolation | Transactions don't interfere           |            |
|     | 격리성    | 트랜잭션이 서로 간섭하지 않음            |            |
|     +-----------+----------------------------------------+            |
|     | Durability| Committed changes survive failures     |            |
|     | 지속성    | 커밋된 변경은 장애에서도 유지            |            |
|     +-----------+----------------------------------------+            |
|                                                                       |
|  3. Transaction Control (트랜잭션 제어)                                 |
|     - BEGIN/START TRANSACTION - Start transaction                     |
|     - COMMIT - Make changes permanent                                 |
|     - ROLLBACK - Undo all changes                                     |
|     - SAVEPOINT - Create partial rollback point                       |
|                                                                       |
|  4. Transaction States (트랜잭션 상태)                                  |
|     Active -> Partially Committed -> Committed                        |
|        |                                                              |
|        +-> Failed -> Aborted                                          |
|                                                                       |
+-----------------------------------------------------------------------+

  REMEMBER: "A transaction is a promise of data integrity"
  기억하세요: "트랜잭션은 데이터 무결성의 약속입니다"
```
