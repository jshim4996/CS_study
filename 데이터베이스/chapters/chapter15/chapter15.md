# Chapter 15. Concurrency Control (동시성 제어)

## Overview (개요)

Concurrency control manages simultaneous access to database resources by multiple transactions. Without proper concurrency control, data integrity can be compromised, leading to various anomalies.

동시성 제어는 여러 트랜잭션이 데이터베이스 리소스에 동시에 접근하는 것을 관리합니다. 적절한 동시성 제어가 없으면 데이터 무결성이 손상되어 다양한 이상 현상이 발생할 수 있습니다.

---

## 1. Concurrency Problems (동시성 문제)

### 1.1 Overview of Concurrency Issues (동시성 문제 개요)

```
+-----------------------------------------------------------------------+
|                    CONCURRENCY PROBLEMS OVERVIEW                       |
|                      동시성 문제 개요                                   |
+-----------------------------------------------------------------------+

  When multiple transactions run concurrently without proper control:
  적절한 제어 없이 여러 트랜잭션이 동시에 실행될 때:

  +------------------+--------------------------------------------------+
  |     Problem      |              Description                         |
  |     문제         |              설명                                 |
  +------------------+--------------------------------------------------+
  | Dirty Read       | Reading uncommitted data from another txn        |
  | 더티 리드        | 다른 트랜잭션의 커밋되지 않은 데이터 읽기           |
  +------------------+--------------------------------------------------+
  | Non-repeatable   | Same query returns different values              |
  | Read             | within same transaction                          |
  | 반복 불가능한    | 같은 트랜잭션 내에서 같은 쿼리가                   |
  | 읽기             | 다른 값 반환                                      |
  +------------------+--------------------------------------------------+
  | Phantom Read     | New rows appear/disappear between queries        |
  | 팬텀 리드        | 쿼리 사이에 새 행이 나타나거나 사라짐              |
  +------------------+--------------------------------------------------+
  | Lost Update      | One update overwrites another's changes          |
  | 갱신 손실        | 한 업데이트가 다른 업데이트의 변경을 덮어씀         |
  +------------------+--------------------------------------------------+
```

### 1.2 Dirty Read (더티 리드)

Reading data that has been modified by another transaction but not yet committed.
다른 트랜잭션에 의해 수정되었지만 아직 커밋되지 않은 데이터를 읽는 것입니다.

```
+-----------------------------------------------------------------------+
|                          DIRTY READ                                    |
|                           더티 리드                                    |
+-----------------------------------------------------------------------+

  Time    Transaction 1                Transaction 2
  시간    트랜잭션 1                    트랜잭션 2
  ----    -------------                -------------

   T1     BEGIN
          |
   T2     UPDATE accounts
          SET balance = 800
          WHERE id = 'A'
          (was 1000)
          |
   T3                                  BEGIN
                                       |
   T4                                  SELECT balance FROM accounts
                                       WHERE id = 'A'
                                       --> Returns 800  (DIRTY READ!)
                                       --> 800 반환 (더티 리드!)
   T5     ROLLBACK
          (balance back to 1000)
          (잔액이 다시 1000으로)
          |
   T6                                  -- Transaction 2 used invalid data!
                                       -- 트랜잭션 2가 무효한 데이터 사용!
                                       -- The $800 never really existed
                                       -- $800는 실제로 존재한 적 없음

  Problem: Transaction 2 read data that was never committed
  문제: 트랜잭션 2가 커밋되지 않은 데이터를 읽음
```

```sql
-- Dirty Read Example
-- 더티 리드 예시

-- Session 1 (Transaction 1)
-- 세션 1 (트랜잭션 1)
START TRANSACTION;
UPDATE accounts SET balance = 800 WHERE account_id = 'A';
-- Don't commit yet... (아직 커밋하지 않음...)

-- Session 2 (Transaction 2) - With READ UNCOMMITTED
-- 세션 2 (트랜잭션 2) - READ UNCOMMITTED 사용
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';
-- Returns 800 (uncommitted value!) - 800 반환 (커밋되지 않은 값!)

-- Back to Session 1
-- 세션 1로 돌아가서
ROLLBACK;  -- Undo the change (변경 취소)

-- Session 2 now has used a value that never existed!
-- 세션 2는 이제 존재한 적 없는 값을 사용했음!
```

### 1.3 Non-repeatable Read (반복 불가능한 읽기)

The same query returns different results within the same transaction because another transaction modified the data.
다른 트랜잭션이 데이터를 수정했기 때문에 같은 트랜잭션 내에서 같은 쿼리가 다른 결과를 반환합니다.

```
+-----------------------------------------------------------------------+
|                      NON-REPEATABLE READ                               |
|                       반복 불가능한 읽기                                |
+-----------------------------------------------------------------------+

  Time    Transaction 1                Transaction 2
  시간    트랜잭션 1                    트랜잭션 2
  ----    -------------                -------------

   T1     BEGIN
          |
   T2     SELECT balance FROM accounts
          WHERE id = 'A'
          --> Returns 1000
          |
   T3                                  BEGIN
                                       |
   T4                                  UPDATE accounts
                                       SET balance = 500
                                       WHERE id = 'A'
                                       |
   T5                                  COMMIT
                                       |
   T6     SELECT balance FROM accounts
          WHERE id = 'A'
          --> Returns 500  (DIFFERENT!)
          --> 500 반환 (다름!)
          |
   T7     -- Same query, different result!
          -- 같은 쿼리, 다른 결과!
          COMMIT

  Problem: Txn 1 got inconsistent reads within same transaction
  문제: 트랜잭션 1이 같은 트랜잭션 내에서 일관되지 않은 읽기
```

```sql
-- Non-repeatable Read Example
-- 반복 불가능한 읽기 예시

-- Session 1 (Transaction 1) - With READ COMMITTED
-- 세션 1 (트랜잭션 1) - READ COMMITTED 사용
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';
-- Returns 1000 (1000 반환)

-- Session 2 (Transaction 2)
-- 세션 2 (트랜잭션 2)
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 'A';
COMMIT;

-- Back to Session 1
-- 세션 1로 돌아가서
SELECT balance FROM accounts WHERE account_id = 'A';
-- Returns 500 (different from first read!)
-- 500 반환 (첫 번째 읽기와 다름!)
COMMIT;
```

### 1.4 Phantom Read (팬텀 리드)

A range query returns different sets of rows because another transaction inserted or deleted rows.
다른 트랜잭션이 행을 삽입하거나 삭제했기 때문에 범위 쿼리가 다른 행 집합을 반환합니다.

```
+-----------------------------------------------------------------------+
|                         PHANTOM READ                                   |
|                          팬텀 리드                                     |
+-----------------------------------------------------------------------+

  Time    Transaction 1                Transaction 2
  시간    트랜잭션 1                    트랜잭션 2
  ----    -------------                -------------

   T1     BEGIN
          |
   T2     SELECT * FROM employees
          WHERE dept = 'Sales'
          --> Returns 3 rows
          --> 3개 행 반환
          (Alice, Bob, Carol)
          |
   T3                                  BEGIN
                                       |
   T4                                  INSERT INTO employees
                                       (name, dept) VALUES
                                       ('Dave', 'Sales')
                                       |
   T5                                  COMMIT
                                       |
   T6     SELECT * FROM employees
          WHERE dept = 'Sales'
          --> Returns 4 rows (PHANTOM!)
          --> 4개 행 반환 (팬텀!)
          (Alice, Bob, Carol, Dave)
          |
   T7     -- A "phantom" row appeared!
          -- "팬텀" 행이 나타남!
          COMMIT

  Problem: New row appeared between identical range queries
  문제: 동일한 범위 쿼리 사이에 새 행이 나타남
```

```sql
-- Phantom Read Example
-- 팬텀 리드 예시

-- Session 1 (Transaction 1) - With REPEATABLE READ
-- 세션 1 (트랜잭션 1) - REPEATABLE READ 사용
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT COUNT(*) FROM products WHERE category = 'Electronics';
-- Returns 10 (10 반환)

-- Session 2 (Transaction 2)
-- 세션 2 (트랜잭션 2)
START TRANSACTION;
INSERT INTO products (name, category) VALUES ('New Phone', 'Electronics');
COMMIT;

-- Back to Session 1
-- 세션 1로 돌아가서
SELECT COUNT(*) FROM products WHERE category = 'Electronics';
-- Returns 11 (phantom row appeared!)
-- 11 반환 (팬텀 행 나타남!)
-- Note: In MySQL InnoDB, this is actually prevented at REPEATABLE READ
-- 참고: MySQL InnoDB에서는 REPEATABLE READ에서도 이것이 방지됨
COMMIT;
```

### 1.5 Lost Update (갱신 손실)

Two transactions read the same data and update it, with one overwriting the other's changes.
두 트랜잭션이 같은 데이터를 읽고 업데이트하여 하나가 다른 하나의 변경을 덮어씁니다.

```
+-----------------------------------------------------------------------+
|                         LOST UPDATE                                    |
|                          갱신 손실                                     |
+-----------------------------------------------------------------------+

  Time    Transaction 1                Transaction 2
  시간    트랜잭션 1                    트랜잭션 2
  ----    -------------                -------------

   T1     BEGIN                        BEGIN
          |                            |
   T2     SELECT balance FROM accounts
          WHERE id = 'A'
          --> balance = 1000
          |
   T3                                  SELECT balance FROM accounts
                                       WHERE id = 'A'
                                       --> balance = 1000
          |                            |
   T4     -- Add $100
          UPDATE accounts
          SET balance = 1100
          WHERE id = 'A'
          |                            |
   T5                                  -- Add $200
                                       UPDATE accounts
                                       SET balance = 1200
                                       WHERE id = 'A'
          |                            |
   T6     COMMIT                       |
          |                            |
   T7                                  COMMIT

          Final balance: 1200 (WRONG!)
          최종 잔액: 1200 (잘못됨!)
          Expected: 1300 ($100 + $200 added)
          예상값: 1300 ($100 + $200 추가됨)
          Transaction 1's update was LOST!
          트랜잭션 1의 업데이트가 손실됨!
```

---

## 2. Transaction Isolation Levels (트랜잭션 격리 수준)

### 2.1 Overview (개요)

```
+-----------------------------------------------------------------------+
|               TRANSACTION ISOLATION LEVELS OVERVIEW                    |
|                트랜잭션 격리 수준 개요                                  |
+-----------------------------------------------------------------------+

  Isolation Level       Dirty    Non-repeatable   Phantom   Performance
  격리 수준             Read     Read             Read      성능
                        더티리드  반복불가읽기      팬텀리드
  -------------------------------------------------------------------
  READ UNCOMMITTED      YES      YES              YES       Highest
  (커밋되지 않은 읽기)   가능      가능             가능       최고

  READ COMMITTED        NO       YES              YES       High
  (커밋된 읽기)          방지      가능             가능       높음

  REPEATABLE READ       NO       NO               YES*      Medium
  (반복 가능한 읽기)     방지      방지             가능*      중간

  SERIALIZABLE          NO       NO               NO        Lowest
  (직렬화 가능)          방지      방지             방지       최저
  -------------------------------------------------------------------

  * MySQL InnoDB prevents phantoms at REPEATABLE READ using gap locks
  * MySQL InnoDB는 갭 락을 사용하여 REPEATABLE READ에서 팬텀 방지

  Trade-off: Higher isolation = Lower concurrency = Lower performance
  트레이드오프: 높은 격리 = 낮은 동시성 = 낮은 성능
```

### 2.2 READ UNCOMMITTED (커밋되지 않은 읽기)

The lowest isolation level. Transactions can see uncommitted changes from other transactions.
가장 낮은 격리 수준입니다. 트랜잭션이 다른 트랜잭션의 커밋되지 않은 변경을 볼 수 있습니다.

```
+-----------------------------------------------------------------------+
|                      READ UNCOMMITTED                                  |
|                     커밋되지 않은 읽기                                  |
+-----------------------------------------------------------------------+

  Characteristics:
  특성:
  - No locks for reading (읽기에 락 없음)
  - Highest concurrency (최고 동시성)
  - Data may be inconsistent (데이터가 일관되지 않을 수 있음)

  Use Cases:
  사용 사례:
  - Rough estimates, not critical data
    대략적인 추정, 중요하지 않은 데이터
  - Monitoring dashboards where approximate values are OK
    근사값이 괜찮은 모니터링 대시보드
  - Never use for financial transactions!
    금융 트랜잭션에는 절대 사용 금지!

  Visual:
  시각화:

  Transaction 1         Transaction 2
  [UPDATE A=100]
       |
       | (uncommitted)
       |--------------> [READ A] --> sees 100 (dirty!)
       |
  [ROLLBACK]
       |--------------> Still thinks A=100 (WRONG!)
```

```sql
-- Setting READ UNCOMMITTED
-- READ UNCOMMITTED 설정

-- MySQL
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- or for a single transaction
-- 또는 단일 트랜잭션에 대해
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
-- ... operations ...
COMMIT;

-- PostgreSQL (doesn't truly support, upgrades to READ COMMITTED)
-- PostgreSQL (진정한 지원 없음, READ COMMITTED로 업그레이드)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- SQL Server
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- or use table hint
-- 또는 테이블 힌트 사용
SELECT * FROM accounts WITH (NOLOCK);
```

### 2.3 READ COMMITTED (커밋된 읽기)

Transactions can only see data that has been committed. Default level for PostgreSQL and SQL Server.
트랜잭션은 커밋된 데이터만 볼 수 있습니다. PostgreSQL과 SQL Server의 기본 수준입니다.

```
+-----------------------------------------------------------------------+
|                       READ COMMITTED                                   |
|                        커밋된 읽기                                      |
+-----------------------------------------------------------------------+

  Characteristics:
  특성:
  - Prevents dirty reads (더티 리드 방지)
  - Each query sees latest committed data (각 쿼리가 최신 커밋 데이터 확인)
  - Non-repeatable reads still possible (반복 불가능한 읽기 여전히 가능)

  Visual:
  시각화:

  Transaction 1         Transaction 2
  [UPDATE A=100]
       |
       | (uncommitted)
       |--------------> [READ A] --> sees OLD value (1000)
       |                             원래 값 확인 (1000)
  [COMMIT]
       |
       |--------------> [READ A] --> sees 100 (committed)
                                     100 확인 (커밋됨)

  Problem: Two reads in Txn 2 return different values
  문제: Txn 2의 두 읽기가 다른 값 반환
```

```sql
-- READ COMMITTED Example
-- READ COMMITTED 예시

-- Session 1
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';  -- Returns 1000

-- Session 2
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 'A';
-- Session 1 reads again
SELECT balance FROM accounts WHERE account_id = 'A';  -- Still 1000 (not committed)

-- Session 2 commits
COMMIT;

-- Session 1 reads again
SELECT balance FROM accounts WHERE account_id = 'A';  -- Now 500 (committed)
-- Different from first read! (첫 번째 읽기와 다름!)
COMMIT;
```

### 2.4 REPEATABLE READ (반복 가능한 읽기)

Guarantees that if a row is read twice within the same transaction, it will return the same values. Default level for MySQL.
같은 트랜잭션 내에서 행을 두 번 읽으면 같은 값을 반환하는 것을 보장합니다. MySQL의 기본 수준입니다.

```
+-----------------------------------------------------------------------+
|                      REPEATABLE READ                                   |
|                      반복 가능한 읽기                                   |
+-----------------------------------------------------------------------+

  Characteristics:
  특성:
  - Prevents dirty reads (더티 리드 방지)
  - Prevents non-repeatable reads (반복 불가능한 읽기 방지)
  - Uses snapshot of data at transaction start (트랜잭션 시작 시 스냅샷 사용)
  - Phantom reads may still occur (except MySQL InnoDB)
    팬텀 리드는 여전히 발생 가능 (MySQL InnoDB 제외)

  Visual:
  시각화:

  Transaction 1                   Transaction 2
  [BEGIN]
  [Snapshot created]
       |
  [READ A] --> 1000
       |
       |                          [UPDATE A = 500]
       |                          [COMMIT]
       |
  [READ A] --> 1000 (same!)
                    (같음!)
       |
  [COMMIT]

  MySQL InnoDB Behavior (Gap Locks prevent phantoms):
  MySQL InnoDB 동작 (갭 락이 팬텀 방지):

  Transaction 1                   Transaction 2
  [BEGIN]
  [SELECT * WHERE age > 20]
  --> Returns rows 1, 2, 3
       |
       |                          [INSERT age=25] <-- BLOCKED!
       |                                              차단됨!
       |
  [SELECT * WHERE age > 20]
  --> Still returns rows 1, 2, 3
```

```sql
-- REPEATABLE READ Example
-- REPEATABLE READ 예시

-- Session 1 (MySQL)
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;

SELECT balance FROM accounts WHERE account_id = 'A';  -- Returns 1000

-- Session 2
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 'A';
COMMIT;

-- Session 1 reads again
SELECT balance FROM accounts WHERE account_id = 'A';
-- Still returns 1000! (Snapshot isolation)
-- 여전히 1000 반환! (스냅샷 격리)

COMMIT;

-- After commit, new transaction sees updated value
-- 커밋 후, 새 트랜잭션은 업데이트된 값 확인
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';  -- Now returns 500
COMMIT;
```

### 2.5 SERIALIZABLE (직렬화 가능)

The highest isolation level. Transactions execute as if they were running one at a time.
가장 높은 격리 수준입니다. 트랜잭션이 한 번에 하나씩 실행되는 것처럼 동작합니다.

```
+-----------------------------------------------------------------------+
|                        SERIALIZABLE                                    |
|                        직렬화 가능                                      |
+-----------------------------------------------------------------------+

  Characteristics:
  특성:
  - Prevents all concurrency anomalies (모든 동시성 이상 방지)
  - Transactions appear to run sequentially (트랜잭션이 순차적으로 실행되는 것처럼 보임)
  - Highest data integrity (최고 데이터 무결성)
  - Lowest concurrency/performance (최저 동시성/성능)
  - May cause lock timeouts or deadlocks (락 타임아웃이나 데드락 발생 가능)

  Visual:
  시각화:

  Transaction 1                   Transaction 2
  [BEGIN]
  [SELECT * WHERE age > 20]
       |
       |                          [INSERT age=25]
       |                          +--> WAITS (or ERROR)
       |                          |    대기 (또는 오류)
  [More operations...]            |
       |                          |
  [COMMIT]                        |
                                  +--> Now proceeds
                                       이제 진행

  Implementation varies:
  구현 방식 다양:
  - Two-Phase Locking (2PL) - SQL Server
  - Serializable Snapshot Isolation (SSI) - PostgreSQL
  - Gap Locking - MySQL InnoDB
```

```sql
-- SERIALIZABLE Example
-- SERIALIZABLE 예시

-- Session 1
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;

SELECT SUM(balance) FROM accounts WHERE type = 'savings';
-- Returns total: $10000

-- Session 2
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
INSERT INTO accounts (type, balance) VALUES ('savings', 500);
-- This may BLOCK until Session 1 commits
-- 세션 1이 커밋할 때까지 차단될 수 있음
-- Or may receive a serialization error
-- 또는 직렬화 오류를 받을 수 있음
COMMIT;

-- Session 1
SELECT SUM(balance) FROM accounts WHERE type = 'savings';
-- Still returns $10000 (no phantom)
-- 여전히 $10000 반환 (팬텀 없음)
COMMIT;

-- After Session 1 commits, Session 2 can proceed
-- 세션 1 커밋 후, 세션 2 진행 가능
```

---

## 3. Isolation Level Comparison (격리 수준 비교)

### 3.1 Detailed Comparison Matrix (상세 비교 매트릭스)

```
+-----------------------------------------------------------------------+
|              ISOLATION LEVELS DETAILED COMPARISON                      |
|                  격리 수준 상세 비교                                    |
+-----------------------------------------------------------------------+

  Level              Lock Type        Read Behavior      Concurrent Writes
  수준               락 유형          읽기 동작          동시 쓰기
  -----------------------------------------------------------------------
  READ               No locks         No blocks          Full concurrency
  UNCOMMITTED        락 없음          차단 없음          완전 동시성
  -----------------------------------------------------------------------
  READ               Short read       Blocks dirty       High concurrency
  COMMITTED          locks            reads only         높은 동시성
                     짧은 읽기 락     더티 리드만 차단
  -----------------------------------------------------------------------
  REPEATABLE         Long read        Snapshot or        Medium concurrency
  READ               locks            long locks         중간 동시성
                     긴 읽기 락       스냅샷/긴 락
  -----------------------------------------------------------------------
  SERIALIZABLE       Range locks/     Full isolation     Low concurrency
                     SSI                                 낮은 동시성
                     범위 락/SSI      완전 격리
  -----------------------------------------------------------------------

  Performance Impact (성능 영향):

  READ UNCOMMITTED  ████████████████████  100% throughput
  READ COMMITTED    ██████████████████    90% throughput
  REPEATABLE READ   ████████████████      80% throughput
  SERIALIZABLE      ████████████          60% throughput
```

### 3.2 Choosing the Right Level (적절한 수준 선택)

```
+-----------------------------------------------------------------------+
|                 ISOLATION LEVEL SELECTION GUIDE                        |
|                   격리 수준 선택 가이드                                 |
+-----------------------------------------------------------------------+

  +----------------------+--------------------------------------------+
  |    Use Case          |    Recommended Level                       |
  |    사용 사례         |    권장 수준                                |
  +----------------------+--------------------------------------------+
  | Reporting queries    | READ UNCOMMITTED or READ COMMITTED         |
  | 리포팅 쿼리          |                                            |
  +----------------------+--------------------------------------------+
  | General OLTP         | READ COMMITTED (PostgreSQL default)        |
  | 일반 OLTP            | REPEATABLE READ (MySQL default)            |
  +----------------------+--------------------------------------------+
  | Financial txns       | REPEATABLE READ or SERIALIZABLE            |
  | 금융 트랜잭션        |                                            |
  +----------------------+--------------------------------------------+
  | Inventory management | SERIALIZABLE (prevent overselling)         |
  | 재고 관리            | (과잉 판매 방지)                            |
  +----------------------+--------------------------------------------+
  | Bank transfers       | SERIALIZABLE                               |
  | 은행 이체            |                                            |
  +----------------------+--------------------------------------------+
  | Booking systems      | SERIALIZABLE (prevent double booking)      |
  | 예약 시스템          | (중복 예약 방지)                            |
  +----------------------+--------------------------------------------+

  Decision Flow:
  결정 흐름:

              Need accurate data?
              정확한 데이터 필요?
                /           \
               NO           YES
               |             |
               v             v
        READ UNCOMMITTED   Need repeatable reads?
                           반복 가능한 읽기 필요?
                            /           \
                           NO           YES
                           |             |
                           v             v
                    READ COMMITTED    Need to prevent phantoms?
                                     팬텀 방지 필요?
                                      /           \
                                     NO           YES
                                     |             |
                                     v             v
                              REPEATABLE READ  SERIALIZABLE
```

---

## 4. Isolation Levels in Different Databases (다양한 데이터베이스의 격리 수준)

### 4.1 MySQL

```sql
-- Check current isolation level
-- 현재 격리 수준 확인
SELECT @@transaction_isolation;  -- MySQL 8.0+
SELECT @@tx_isolation;           -- MySQL 5.x

-- Set isolation level (session)
-- 격리 수준 설정 (세션)
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;  -- Default
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set isolation level (global - requires SUPER privilege)
-- 격리 수준 설정 (전역 - SUPER 권한 필요)
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- MySQL InnoDB specific: Gap Locking at REPEATABLE READ
-- MySQL InnoDB 특정: REPEATABLE READ에서 갭 락킹
-- This prevents phantom reads even at REPEATABLE READ
-- 이것은 REPEATABLE READ에서도 팬텀 리드 방지
```

### 4.2 PostgreSQL

```sql
-- Check current isolation level
-- 현재 격리 수준 확인
SHOW transaction_isolation;

-- Set isolation level
-- 격리 수준 설정
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- Default

-- For current transaction only
-- 현재 트랜잭션만
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- ... operations ...
COMMIT;

-- PostgreSQL note: Uses SSI (Serializable Snapshot Isolation)
-- PostgreSQL 참고: SSI (직렬화 가능한 스냅샷 격리) 사용
-- More efficient than traditional locking
-- 전통적인 락킹보다 효율적
```

### 4.3 SQL Server

```sql
-- Check current isolation level
-- 현재 격리 수준 확인
DBCC USEROPTIONS;

-- Set isolation level
-- 격리 수준 설정
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- Default
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- SQL Server specific: SNAPSHOT isolation
-- SQL Server 특정: SNAPSHOT 격리
ALTER DATABASE MyDB SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

-- Table hints for specific queries
-- 특정 쿼리에 대한 테이블 힌트
SELECT * FROM accounts WITH (NOLOCK);        -- READ UNCOMMITTED
SELECT * FROM accounts WITH (HOLDLOCK);      -- SERIALIZABLE
SELECT * FROM accounts WITH (REPEATABLEREAD); -- REPEATABLE READ
```

---

## 5. Trade-offs (트레이드오프)

### 5.1 Isolation vs Performance (격리 vs 성능)

```
+-----------------------------------------------------------------------+
|                  ISOLATION VS PERFORMANCE TRADE-OFF                    |
|                      격리 vs 성능 트레이드오프                          |
+-----------------------------------------------------------------------+

  Isolation Level
  격리 수준
        ^
        |
  HIGH  | SERIALIZABLE --------> Highest Integrity
        |       |                최고 무결성
        |       |                But: Low throughput, possible deadlocks
        |       |                단점: 낮은 처리량, 데드락 가능
        |       v
        | REPEATABLE READ -----> Good balance for most OLTP
        |       |                대부분 OLTP에 좋은 균형
        |       |                But: Still some locking overhead
        |       |                단점: 여전히 일부 락킹 오버헤드
        |       v
        | READ COMMITTED ------> Good for high-concurrency systems
        |       |                높은 동시성 시스템에 적합
        |       |                But: Non-repeatable reads possible
        |       |                단점: 반복 불가능한 읽기 가능
        |       v
  LOW   | READ UNCOMMITTED ----> Maximum performance
        |                        최대 성능
        |                        But: Data may be inaccurate
        |                        단점: 데이터가 부정확할 수 있음
        +-------------------------------------->
                                            Performance
                                            성능

  The 80/20 Rule:
  80/20 규칙:
  - 80% of applications work fine with READ COMMITTED
    80%의 애플리케이션은 READ COMMITTED로 충분
  - 20% need higher isolation for specific operations
    20%는 특정 작업에 더 높은 격리 필요
```

### 5.2 Practical Guidelines (실용적 지침)

```sql
-- Strategy: Use appropriate level per operation
-- 전략: 작업별로 적절한 수준 사용

-- For read-heavy reporting (읽기 위주 리포팅)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT COUNT(*), SUM(amount) FROM transactions
WHERE date > '2024-01-01';
COMMIT;

-- For critical financial operations (중요한 금융 작업)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
-- Check balance
SELECT balance INTO @balance FROM accounts
WHERE account_id = 'A' FOR UPDATE;
-- Transfer if sufficient
IF @balance >= 1000 THEN
    UPDATE accounts SET balance = balance - 1000 WHERE account_id = 'A';
    UPDATE accounts SET balance = balance + 1000 WHERE account_id = 'B';
END IF;
COMMIT;

-- For inventory reservation (재고 예약)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
SELECT quantity FROM products WHERE product_id = 1 FOR UPDATE;
-- Reserve if available
UPDATE products SET quantity = quantity - 1
WHERE product_id = 1 AND quantity >= 1;
-- Check if update succeeded
IF ROW_COUNT() = 0 THEN
    ROLLBACK;  -- Item not available
ELSE
    INSERT INTO reservations (product_id, customer_id) VALUES (1, @cust_id);
    COMMIT;
END IF;
```

---

## Summary (요약)

```
+-----------------------------------------------------------------------+
|                        CHAPTER 15 SUMMARY                              |
|                          제15장 요약                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|  CONCURRENCY PROBLEMS (동시성 문제):                                   |
|                                                                       |
|  +------------------+---------------------------------------------+   |
|  | Dirty Read       | Reading uncommitted data                    |   |
|  | 더티 리드        | 커밋되지 않은 데이터 읽기                     |   |
|  +------------------+---------------------------------------------+   |
|  | Non-repeatable   | Same row returns different values           |   |
|  | 반복 불가능 읽기  | 같은 행이 다른 값 반환                        |   |
|  +------------------+---------------------------------------------+   |
|  | Phantom Read     | New rows appear in range query              |   |
|  | 팬텀 리드        | 범위 쿼리에 새 행 나타남                       |   |
|  +------------------+---------------------------------------------+   |
|  | Lost Update      | One update overwrites another               |   |
|  | 갱신 손실        | 한 업데이트가 다른 것을 덮어씀                 |   |
|  +------------------+---------------------------------------------+   |
|                                                                       |
|  ISOLATION LEVELS (격리 수준):                                         |
|                                                                       |
|  Level               Dirty   Non-rep  Phantom   Typical Use           |
|  ------------------------------------------------------------------   |
|  READ UNCOMMITTED    Yes     Yes      Yes       Rough estimates       |
|  READ COMMITTED      No      Yes      Yes       General apps (PG)     |
|  REPEATABLE READ     No      No       Yes*      General apps (MySQL)  |
|  SERIALIZABLE        No      No       No        Critical txns         |
|                                                                       |
|  * MySQL InnoDB prevents phantoms at REPEATABLE READ                  |
|                                                                       |
|  KEY TRADE-OFF (핵심 트레이드오프):                                     |
|  Higher Isolation = Lower Concurrency = Lower Throughput              |
|  높은 격리 = 낮은 동시성 = 낮은 처리량                                  |
|                                                                       |
+-----------------------------------------------------------------------+

  REMEMBER: "Choose the lowest isolation level that meets your needs"
  기억하세요: "필요를 충족하는 가장 낮은 격리 수준을 선택하세요"
```
