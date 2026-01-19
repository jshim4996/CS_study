# Chapter 16. Locking (락)

## Overview (개요)

Locking is a fundamental mechanism for controlling concurrent access to database resources. Locks prevent multiple transactions from modifying the same data simultaneously, ensuring data integrity.

락킹은 데이터베이스 리소스에 대한 동시 접근을 제어하는 기본 메커니즘입니다. 락은 여러 트랜잭션이 동시에 같은 데이터를 수정하는 것을 방지하여 데이터 무결성을 보장합니다.

---

## 1. Why Locking is Needed (락이 필요한 이유)

### 1.1 The Problem Without Locks (락 없이의 문제)

```
+-----------------------------------------------------------------------+
|                    WITHOUT LOCKING (락 없이)                           |
+-----------------------------------------------------------------------+

  Scenario: Two transactions updating the same account
  시나리오: 두 트랜잭션이 같은 계좌를 업데이트

  Initial balance: $1000
  초기 잔액: $1000

  Transaction 1                    Transaction 2
  (Withdraw $200)                  (Withdraw $300)
  ($200 출금)                      ($300 출금)

  Time
   |
   v
  T1  READ balance ($1000)
   |
  T2                               READ balance ($1000)
   |
  T3  Calculate: 1000 - 200 = 800
   |
  T4                               Calculate: 1000 - 300 = 700
   |
  T5  WRITE balance = $800
   |
  T6                               WRITE balance = $700  <-- OVERWRITES!
   |                                                         덮어씀!
  T7  COMMIT
   |
  T8                               COMMIT

  Final balance: $700 (WRONG!)
  최종 잔액: $700 (잘못됨!)

  Expected: $1000 - $200 - $300 = $500
  예상값: $1000 - $200 - $300 = $500

  Lost: $300 (the first transaction's update was lost)
  손실: $300 (첫 번째 트랜잭션의 업데이트가 손실됨)
```

### 1.2 The Solution with Locks (락을 사용한 해결)

```
+-----------------------------------------------------------------------+
|                     WITH LOCKING (락 사용)                             |
+-----------------------------------------------------------------------+

  Transaction 1                    Transaction 2
  (Withdraw $200)                  (Withdraw $300)

  Time
   |
   v
  T1  LOCK account (Exclusive)
   |  계좌 락 획득 (배타적)
   |
  T2  READ balance ($1000)
   |
  T3                               REQUEST LOCK on account
   |                               계좌에 락 요청
   |                               --> BLOCKED! Waiting...
   |                               --> 차단됨! 대기 중...
  T4  Calculate: 1000 - 200 = 800
   |
  T5  WRITE balance = $800
   |
  T6  COMMIT + RELEASE LOCK
   |  커밋 + 락 해제
   |
  T7                               --> LOCK acquired (락 획득)
   |
  T8                               READ balance ($800)
   |
  T9                               Calculate: 800 - 300 = 500
   |
  T10                              WRITE balance = $500
   |
  T11                              COMMIT + RELEASE LOCK

  Final balance: $500 (CORRECT!)
  최종 잔액: $500 (정확!)
```

---

## 2. Shared Lock vs Exclusive Lock (공유 락 vs 배타적 락)

### 2.1 Lock Types Overview (락 유형 개요)

```
+-----------------------------------------------------------------------+
|                         LOCK TYPES                                     |
|                          락 유형                                       |
+-----------------------------------------------------------------------+

  +------------------+------------------------------------------------+
  |  Lock Type       |  Description                                   |
  |  락 유형         |  설명                                          |
  +------------------+------------------------------------------------+
  |  Shared (S)      |  Multiple transactions can read simultaneously |
  |  공유 락         |  여러 트랜잭션이 동시에 읽기 가능                |
  |                  |  Also called "Read Lock" (읽기 락이라고도 함)   |
  +------------------+------------------------------------------------+
  |  Exclusive (X)   |  Only one transaction can read/write           |
  |  배타적 락       |  하나의 트랜잭션만 읽기/쓰기 가능               |
  |                  |  Also called "Write Lock" (쓰기 락이라고도 함)  |
  +------------------+------------------------------------------------+

  Compatibility Matrix:
  호환성 매트릭스:

           | Shared (S) | Exclusive (X) |
           | 공유       | 배타적        |
  ---------+------------+---------------+
  Shared   |     ✓      |      ✗        |
  공유     | Compatible | Conflict      |
           | 호환       | 충돌          |
  ---------+------------+---------------+
  Exclusive|     ✗      |      ✗        |
  배타적   | Conflict   | Conflict      |
           | 충돌       | 충돌          |
  ---------+------------+---------------+
```

### 2.2 Shared Lock (공유 락)

```
+-----------------------------------------------------------------------+
|                        SHARED LOCK (S-LOCK)                            |
|                          공유 락                                       |
+-----------------------------------------------------------------------+

  Purpose: Allow multiple readers, block writers
  목적: 여러 읽기자 허용, 쓰기자 차단

  Visual:
  시각화:

  Transaction 1        Transaction 2        Transaction 3
       |                    |                    |
       v                    v                    v
  [S-LOCK row]         [S-LOCK row]        [X-LOCK row]
       |                    |                    |
       v                    v                    v
      OK ✓                 OK ✓               WAIT ✗
                                              (대기)
       |                    |                    |
       v                    v                    |
  [READ row]           [READ row]               |
       |                    |                    |
       v                    v                    |
  [RELEASE]            [RELEASE]                |
                                                v
                                           [NOW OK ✓]

  Multiple S-Locks can coexist on the same row
  여러 S-Lock이 같은 행에 공존 가능
```

```sql
-- Acquiring Shared Lock in SQL
-- SQL에서 공유 락 획득

-- MySQL
SELECT * FROM accounts WHERE account_id = 1 LOCK IN SHARE MODE;
-- or (MySQL 8.0+)
SELECT * FROM accounts WHERE account_id = 1 FOR SHARE;

-- PostgreSQL
SELECT * FROM accounts WHERE account_id = 1 FOR SHARE;

-- SQL Server
SELECT * FROM accounts WITH (HOLDLOCK) WHERE account_id = 1;

-- Example: Checking balance before other operations
-- 예시: 다른 작업 전 잔액 확인
START TRANSACTION;
SELECT balance FROM accounts
WHERE account_id = 1
FOR SHARE;  -- Others can read, but can't modify
            -- 다른 트랜잭션은 읽을 수 있지만 수정 불가

-- Do some validation... (검증 수행...)

COMMIT;  -- Release lock (락 해제)
```

### 2.3 Exclusive Lock (배타적 락)

```
+-----------------------------------------------------------------------+
|                      EXCLUSIVE LOCK (X-LOCK)                           |
|                          배타적 락                                     |
+-----------------------------------------------------------------------+

  Purpose: Block all other readers and writers
  목적: 모든 다른 읽기자와 쓰기자 차단

  Visual:
  시각화:

  Transaction 1        Transaction 2        Transaction 3
       |                    |                    |
       v                    |                    |
  [X-LOCK row]              |                    |
       |                    v                    v
       |               [S-LOCK row]         [X-LOCK row]
       |                    |                    |
       v                    v                    v
      OK ✓               WAIT ✗              WAIT ✗
                         (대기)              (대기)
       |
       v
  [WRITE row]
       |
       v
  [RELEASE]
       |
       +-------------------+------------------->
                           |                   |
                           v                   v
                      [NOW OK ✓]          [NOW OK ✓]
                      (One gets lock first)
                      (하나가 먼저 락 획득)

  Only one X-Lock can exist on a row
  하나의 X-Lock만 행에 존재 가능
```

```sql
-- Acquiring Exclusive Lock in SQL
-- SQL에서 배타적 락 획득

-- MySQL / PostgreSQL
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;

-- SQL Server
SELECT * FROM accounts WITH (UPDLOCK) WHERE account_id = 1;

-- Example: Transferring money with exclusive locks
-- 예시: 배타적 락으로 송금
START TRANSACTION;

-- Lock both accounts (order matters for deadlock prevention!)
-- 두 계좌 모두 잠금 (데드락 방지를 위해 순서 중요!)
SELECT * FROM accounts
WHERE account_id IN (1, 2)
ORDER BY account_id  -- Consistent ordering
FOR UPDATE;

-- Perform transfer
-- 이체 수행
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;  -- Release locks (락 해제)
```

### 2.4 Lock Escalation (락 에스컬레이션)

```
+-----------------------------------------------------------------------+
|                      LOCK ESCALATION                                   |
|                       락 에스컬레이션                                   |
+-----------------------------------------------------------------------+

  When too many row locks exist, the database may escalate to table lock
  너무 많은 행 락이 존재하면 데이터베이스가 테이블 락으로 에스컬레이션

  Row Locks                          Table Lock
  행 락                              테이블 락

  +-------+                         +---------------+
  | Row 1 | [X]                     |               |
  +-------+                         |               |
  | Row 2 | [X]                     |    Table      |
  +-------+              ==>        |     [X]       |
  | Row 3 | [X]                     |               |
  +-------+                         |               |
  | ...   | [X]                     |               |
  +-------+                         +---------------+
  | Row N | [X]
  +-------+

  Many individual locks              One table lock
  (high memory overhead)             (lower overhead,
  많은 개별 락                         but blocks entire table)
  (높은 메모리 오버헤드)               하나의 테이블 락
                                     (낮은 오버헤드,
                                     하지만 전체 테이블 차단)

  Thresholds vary by database:
  임계값은 데이터베이스마다 다름:
  - SQL Server: ~5000 row locks per table
  - MySQL: Generally avoids escalation, uses page locks instead
```

---

## 3. Two-Phase Locking Protocol (2단계 락킹 프로토콜)

### 3.1 Concept (개념)

```
+-----------------------------------------------------------------------+
|                  TWO-PHASE LOCKING (2PL)                               |
|                   2단계 락킹 프로토콜                                   |
+-----------------------------------------------------------------------+

  Rule: Transactions must acquire all locks before releasing any
  규칙: 트랜잭션은 락을 해제하기 전에 모든 락을 획득해야 함

  Two Phases:
  두 단계:

  1. Growing Phase (확장 단계)
     - Acquire locks (락 획득)
     - No locks released (락 해제 없음)

  2. Shrinking Phase (축소 단계)
     - Release locks (락 해제)
     - No new locks acquired (새 락 획득 없음)

  Visual Timeline:
  시각적 타임라인:

  Number of
  Locks
  락 수
      ^
      |           Lock Point
      |           락 포인트
      |              |
      |    /\        v
      |   /  \
      |  /    \
      | /      \
      |/        \
      +-------------------------> Time
      |         |                 시간
      Growing   Shrinking
      Phase     Phase
      확장 단계  축소 단계
```

### 3.2 2PL Example (2PL 예시)

```
+-----------------------------------------------------------------------+
|                        2PL EXAMPLE                                     |
|                         2PL 예시                                       |
+-----------------------------------------------------------------------+

  Transaction: Transfer $100 from Account A to Account B
  트랜잭션: 계좌 A에서 계좌 B로 $100 이체

  Time    Operation                    Phase
  시간    작업                         단계
  -----   ---------                    -----
  T1      BEGIN                        --
  T2      X-LOCK(Account A)            Growing (확장)
  T3      READ(Account A)              Growing
  T4      X-LOCK(Account B)            Growing  <-- Lock Point (락 포인트)
  T5      READ(Account B)              Shrinking (축소)
  T6      WRITE(Account A - 100)       Shrinking
  T7      WRITE(Account B + 100)       Shrinking
  T8      UNLOCK(Account A)            Shrinking
  T9      UNLOCK(Account B)            Shrinking
  T10     COMMIT                       --

  ✓ All locks acquired before any release (모든 락이 해제 전에 획득됨)
  ✓ This ensures serializability (이것이 직렬화 가능성 보장)
```

### 3.3 Strict 2PL (엄격한 2PL)

```
+-----------------------------------------------------------------------+
|                      STRICT TWO-PHASE LOCKING                          |
|                       엄격한 2단계 락킹                                 |
+-----------------------------------------------------------------------+

  Additional Rule: All locks held until COMMIT/ROLLBACK
  추가 규칙: 모든 락은 COMMIT/ROLLBACK까지 유지

  Benefits:
  장점:
  - Prevents cascading rollbacks (연쇄 롤백 방지)
  - Simpler recovery (더 간단한 복구)
  - Most databases use this variation (대부분의 데이터베이스가 사용)

  Visual Comparison:
  시각적 비교:

  Basic 2PL:                         Strict 2PL:
  기본 2PL:                           엄격한 2PL:

  Locks                              Locks
      ^                                  ^
      |    /\                            |    /|
      |   /  \                           |   / |
      |  /    \                          |  /  |
      | /      \                         | /   |
      |/        \                        |/    |
      +----------\----> Time             +-----+----> Time
                  \                            |
                   Release                    COMMIT
                   individually               then release all
                   개별 해제                   커밋 후 전체 해제
```

```sql
-- Strict 2PL in Practice (default behavior in most databases)
-- 실제 엄격한 2PL (대부분 데이터베이스의 기본 동작)

START TRANSACTION;

-- Growing phase: Acquire all needed locks
-- 확장 단계: 필요한 모든 락 획득
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- X-Lock
SELECT * FROM accounts WHERE id = 2 FOR UPDATE;  -- X-Lock

-- Operations using locked data
-- 잠긴 데이터를 사용한 작업
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- All locks released here (모든 락이 여기서 해제)
         -- Shrinking phase is instantaneous
         -- 축소 단계가 즉시 발생
```

---

## 4. Deadlock and Solutions (데드락과 해결책)

### 4.1 What is Deadlock? (데드락이란?)

```
+-----------------------------------------------------------------------+
|                          DEADLOCK                                      |
|                           데드락                                       |
+-----------------------------------------------------------------------+

  Definition: Two or more transactions waiting for each other's locks
  정의: 둘 이상의 트랜잭션이 서로의 락을 기다리는 상태

  Classic Example:
  전형적인 예:

  Transaction 1                    Transaction 2
       |                                |
       v                                v
  [LOCK(A)] ✓                     [LOCK(B)] ✓
       |                                |
       v                                v
  [LOCK(B)] -----> WAIT           [LOCK(A)] -----> WAIT
       ^            |                  ^            |
       |            |                  |            |
       +------------+                  +------------+
                    |                               |
                    v                               v
               Waiting for B                  Waiting for A
               B를 기다림                      A를 기다림

                          DEADLOCK!
                          (Neither can proceed)
                          (어느 쪽도 진행 불가)

  Visual (Cycle Detection):
  시각화 (사이클 감지):

       +-------+         holds        +-------+
       |  T1   |--------------------->|   A   |
       +-------+                      +-------+
           |                              ^
           | wants                        | holds
           |                              |
           v                              |
       +-------+                      +-------+
       |   B   |<---------------------|  T2   |
       +-------+         wants        +-------+

       Cycle detected = Deadlock!
       사이클 감지됨 = 데드락!
```

### 4.2 Deadlock Detection (데드락 감지)

```
+-----------------------------------------------------------------------+
|                    DEADLOCK DETECTION                                  |
|                      데드락 감지                                       |
+-----------------------------------------------------------------------+

  Wait-For Graph:
  대기 그래프:

  Transactions are nodes, edges represent "waiting for"
  트랜잭션이 노드, 에지는 "대기 중"을 나타냄

  No Deadlock:                      Deadlock (Cycle):
  데드락 없음:                       데드락 (사이클):

       T1                               T1
       |                               /  \
       v                              v    \
       T2                            T2     |
       |                              |     |
       v                              v     |
       T3                            T3-----+

  (No cycle)                        (Cycle: T1->T2->T3->T1)
  (사이클 없음)                      (사이클 존재)

  Database periodically checks for cycles in wait-for graph
  데이터베이스가 대기 그래프에서 주기적으로 사이클 확인
```

```sql
-- Checking for deadlocks (database specific)
-- 데드락 확인 (데이터베이스별)

-- MySQL: View current locks and waits
-- MySQL: 현재 락과 대기 확인
SELECT * FROM performance_schema.data_locks;
SELECT * FROM performance_schema.data_lock_waits;

-- MySQL: See InnoDB status including deadlock info
-- MySQL: 데드락 정보를 포함한 InnoDB 상태 확인
SHOW ENGINE INNODB STATUS;

-- PostgreSQL: View locks
-- PostgreSQL: 락 확인
SELECT * FROM pg_locks;
SELECT * FROM pg_stat_activity WHERE wait_event_type = 'Lock';

-- SQL Server: View blocking
-- SQL Server: 차단 확인
SELECT * FROM sys.dm_exec_requests WHERE blocking_session_id <> 0;
```

### 4.3 Deadlock Solutions (데드락 해결책)

```
+-----------------------------------------------------------------------+
|                    DEADLOCK SOLUTIONS                                  |
|                      데드락 해결책                                      |
+-----------------------------------------------------------------------+

  1. DEADLOCK PREVENTION (데드락 방지)
     Avoid conditions that lead to deadlock
     데드락으로 이어지는 조건 회피

  2. DEADLOCK DETECTION + RECOVERY (데드락 감지 + 복구)
     Detect deadlocks and resolve by aborting a transaction
     데드락 감지 후 트랜잭션 중단으로 해결

  3. TIMEOUT (타임아웃)
     Abort transaction if waiting too long for a lock
     락 대기가 너무 길면 트랜잭션 중단
```

**Solution 1: Lock Ordering (락 순서 지정)**

```
+-----------------------------------------------------------------------+
|                   PREVENTION: LOCK ORDERING                            |
|                   방지: 락 순서 지정                                    |
+-----------------------------------------------------------------------+

  Rule: Always acquire locks in a consistent order (e.g., by ID)
  규칙: 항상 일관된 순서로 락 획득 (예: ID 순)

  WITHOUT ordering:                  WITH ordering:
  순서 없이:                         순서와 함께:

  Transaction 1    Transaction 2     Transaction 1    Transaction 2
  LOCK(A) ✓        LOCK(B) ✓         LOCK(A) ✓        LOCK(A) WAIT
  LOCK(B) WAIT     LOCK(A) WAIT      LOCK(B) ✓        ...waiting...
       |               |                  |
       v               v                  v
      DEADLOCK!                      [Complete work]
                                     UNLOCK(A,B)
                                          |
                                          v
                                     Transaction 2 proceeds
```

```sql
-- Lock Ordering Example
-- 락 순서 지정 예시

-- WRONG: Inconsistent order can cause deadlock
-- 잘못됨: 일관되지 않은 순서는 데드락 유발 가능
-- Session 1:
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
SELECT * FROM accounts WHERE id = 2 FOR UPDATE;  -- May deadlock if Session 2 did opposite
                                                  -- 세션 2가 반대로 하면 데드락 가능

-- RIGHT: Always order by account_id
-- 옳음: 항상 account_id로 정렬
START TRANSACTION;
SELECT * FROM accounts
WHERE id IN (1, 2)
ORDER BY id  -- Ensures consistent order (일관된 순서 보장)
FOR UPDATE;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

**Solution 2: Deadlock Detection and Victim Selection (데드락 감지 및 희생자 선택)**

```
+-----------------------------------------------------------------------+
|             DETECTION: VICTIM SELECTION                                |
|             감지: 희생자 선택                                           |
+-----------------------------------------------------------------------+

  When deadlock detected, one transaction must be aborted (victim)
  데드락 감지 시, 하나의 트랜잭션을 중단해야 함 (희생자)

  Victim Selection Criteria:
  희생자 선택 기준:

  +------------------+--------------------------------------------+
  |  Factor          |  Description                               |
  |  요인            |  설명                                       |
  +------------------+--------------------------------------------+
  | Transaction age  | Abort younger transaction                  |
  | 트랜잭션 나이    | 더 젊은 트랜잭션 중단                        |
  +------------------+--------------------------------------------+
  | Work done        | Abort transaction with less work           |
  | 수행한 작업량    | 작업량이 적은 트랜잭션 중단                   |
  +------------------+--------------------------------------------+
  | Locks held       | Abort transaction with fewer locks         |
  | 보유 락 수       | 락을 적게 가진 트랜잭션 중단                  |
  +------------------+--------------------------------------------+
  | Priority         | Abort lower priority transaction           |
  | 우선순위         | 낮은 우선순위 트랜잭션 중단                   |
  +------------------+--------------------------------------------+

  After abort, the victim transaction is ROLLED BACK
  중단 후, 희생자 트랜잭션은 롤백됨
```

```sql
-- Handling deadlock errors (application code)
-- 데드락 오류 처리 (애플리케이션 코드)

-- MySQL error code 1213: Deadlock found
-- MySQL 오류 코드 1213: 데드락 발견
-- PostgreSQL error code 40P01: deadlock_detected
-- PostgreSQL 오류 코드 40P01: 데드락 감지됨

-- Pseudo-code for retry logic
-- 재시도 로직 의사 코드
/*
max_retries = 3
for attempt in range(max_retries):
    try:
        START TRANSACTION
        -- business logic --
        COMMIT
        break  -- Success, exit loop
    except DeadlockError:
        ROLLBACK
        if attempt == max_retries - 1:
            raise  -- Final attempt failed
        sleep(random(0.1, 0.5))  -- Random backoff
*/

-- MySQL: Set deadlock detection interval
-- MySQL: 데드락 감지 간격 설정
-- Default is 1 second; can be configured
SET GLOBAL innodb_deadlock_detect = ON;
```

**Solution 3: Timeout (타임아웃)**

```sql
-- Setting lock wait timeout
-- 락 대기 타임아웃 설정

-- MySQL: Lock wait timeout (default 50 seconds)
-- MySQL: 락 대기 타임아웃 (기본 50초)
SET innodb_lock_wait_timeout = 10;  -- 10 seconds

-- PostgreSQL: Lock timeout
-- PostgreSQL: 락 타임아웃
SET lock_timeout = '10s';

-- SQL Server: Lock timeout (milliseconds)
-- SQL Server: 락 타임아웃 (밀리초)
SET LOCK_TIMEOUT 10000;  -- 10 seconds

-- Example with timeout
-- 타임아웃 예시
SET innodb_lock_wait_timeout = 5;  -- 5 seconds

START TRANSACTION;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- If another transaction holds the lock for > 5 seconds:
-- 다른 트랜잭션이 5초 이상 락을 보유하면:
-- ERROR 1205: Lock wait timeout exceeded
-- 오류 1205: 락 대기 타임아웃 초과
COMMIT;
```

---

## 5. Optimistic vs Pessimistic Locking (낙관적 vs 비관적 락킹)

### 5.1 Comparison Overview (비교 개요)

```
+-----------------------------------------------------------------------+
|              OPTIMISTIC VS PESSIMISTIC LOCKING                         |
|                낙관적 vs 비관적 락킹                                    |
+-----------------------------------------------------------------------+

  PESSIMISTIC LOCKING                OPTIMISTIC LOCKING
  비관적 락킹                         낙관적 락킹

  "Assume conflicts will happen"     "Assume conflicts are rare"
  "충돌이 발생할 것으로 가정"          "충돌이 드물 것으로 가정"

  +----------------------------+    +----------------------------+
  |  Lock data BEFORE access   |    |  No locks during read      |
  |  접근 전에 데이터 잠금      |    |  읽기 중 락 없음            |
  +----------------------------+    +----------------------------+
  |  Block other transactions  |    |  Check for conflicts at    |
  |  다른 트랜잭션 차단         |    |  commit time               |
  +----------------------------+    |  커밋 시 충돌 확인          |
  |  High contention           |    +----------------------------+
  |  environments              |    |  Low contention            |
  |  높은 경합 환경             |    |  environments              |
  +----------------------------+    |  낮은 경합 환경             |
  |  Database-level locks      |    +----------------------------+
  |  데이터베이스 수준 락       |    |  Application-level version |
  +----------------------------+    |  애플리케이션 수준 버전     |
                                    +----------------------------+

  Use When:                         Use When:
  사용 시점:                         사용 시점:
  - High write contention           - Read-heavy workloads
    높은 쓰기 경합                    읽기 중심 워크로드
  - Short transactions              - Long-running operations
    짧은 트랜잭션                     오래 실행되는 작업
  - Data integrity is critical      - Web applications
    데이터 무결성이 중요               웹 애플리케이션
```

### 5.2 Pessimistic Locking (비관적 락킹)

```
+-----------------------------------------------------------------------+
|                    PESSIMISTIC LOCKING                                 |
|                       비관적 락킹                                       |
+-----------------------------------------------------------------------+

  Flow:
  흐름:

  Transaction 1                    Transaction 2
       |                                |
       v                                |
  [BEGIN]                               |
       |                                |
       v                                |
  [SELECT FOR UPDATE]                   |
  [Lock acquired] ✓                     |
       |                                v
       |                           [BEGIN]
       |                                |
       v                                v
  [Read & Modify]                  [SELECT FOR UPDATE]
       |                                |
       |                                v
       |                           [BLOCKED - Waiting]
       |                           [차단됨 - 대기 중]
       v                                |
  [COMMIT]                              |
  [Lock released]                       |
       |                                v
       +--------------------------->[Lock acquired] ✓
                                        |
                                        v
                                   [Read & Modify]
                                        |
                                        v
                                   [COMMIT]
```

```sql
-- Pessimistic Locking Examples
-- 비관적 락킹 예시

-- MySQL: SELECT FOR UPDATE (Exclusive Lock)
-- MySQL: SELECT FOR UPDATE (배타적 락)
START TRANSACTION;

SELECT * FROM products
WHERE product_id = 1
FOR UPDATE;  -- Lock the row (행 잠금)

-- Check stock and update
-- 재고 확인 및 업데이트
UPDATE products
SET stock = stock - 1
WHERE product_id = 1;

COMMIT;  -- Release lock (락 해제)

-- MySQL: SELECT FOR SHARE (Shared Lock)
-- MySQL: SELECT FOR SHARE (공유 락)
START TRANSACTION;

SELECT * FROM products
WHERE product_id = 1
FOR SHARE;  -- Shared lock - others can read but not write
            -- 공유 락 - 다른 트랜잭션은 읽을 수 있지만 쓰기 불가

-- Validate data...
-- 데이터 검증...

COMMIT;

-- With NOWAIT option (don't wait for lock)
-- NOWAIT 옵션 (락을 기다리지 않음)
SELECT * FROM products
WHERE product_id = 1
FOR UPDATE NOWAIT;  -- Fails immediately if locked
                    -- 잠겨 있으면 즉시 실패

-- With SKIP LOCKED option (skip locked rows)
-- SKIP LOCKED 옵션 (잠긴 행 건너뛰기)
SELECT * FROM products
WHERE category = 'electronics'
FOR UPDATE SKIP LOCKED  -- Skip rows that are locked
LIMIT 1;                -- 잠긴 행 건너뛰기
```

### 5.3 Optimistic Locking (낙관적 락킹)

```
+-----------------------------------------------------------------------+
|                    OPTIMISTIC LOCKING                                  |
|                       낙관적 락킹                                       |
+-----------------------------------------------------------------------+

  Flow:
  흐름:

  Transaction 1                    Transaction 2
       |                                |
       v                                v
  [Read row + version]             [Read row + version]
  version = 1                      version = 1
       |                                |
       v                                |
  [Modify locally]                      |
       |                                v
       v                           [Modify locally]
  [UPDATE WHERE version=1]              |
  [Set version=2]                       |
  [1 row affected] ✓                    |
       |                                v
       v                           [UPDATE WHERE version=1]
  [COMMIT]                         [0 rows affected] ✗
                                   (Version changed!)
                                   (버전이 변경됨!)
                                        |
                                        v
                                   [Retry or Error]
                                   [재시도 또는 오류]
```

```sql
-- Optimistic Locking Implementation
-- 낙관적 락킹 구현

-- Table with version column
-- 버전 컬럼이 있는 테이블
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    stock INT,
    version INT DEFAULT 1  -- Version for optimistic locking
                           -- 낙관적 락킹을 위한 버전
);

-- Read with version
-- 버전과 함께 읽기
SELECT product_id, name, stock, version
FROM products
WHERE product_id = 1;
-- Result: product_id=1, stock=10, version=5

-- Update with version check
-- 버전 확인과 함께 업데이트
UPDATE products
SET stock = stock - 1,
    version = version + 1
WHERE product_id = 1
  AND version = 5;  -- Only update if version matches
                    -- 버전이 일치할 때만 업데이트

-- Check if update succeeded
-- 업데이트 성공 여부 확인
-- If affected_rows = 0, someone else modified the data
-- affected_rows = 0이면 다른 사람이 데이터를 수정함
-- Retry the operation!
-- 작업 재시도!

-- Alternative: Using timestamp
-- 대안: 타임스탬프 사용
CREATE TABLE products_v2 (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    stock INT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
               ON UPDATE CURRENT_TIMESTAMP
);

UPDATE products_v2
SET stock = stock - 1
WHERE product_id = 1
  AND updated_at = '2024-01-15 10:30:00';  -- Version check with timestamp
                                           -- 타임스탬프로 버전 확인
```

### 5.4 When to Use Each (각각의 사용 시점)

```
+-----------------------------------------------------------------------+
|                  CHOOSING LOCKING STRATEGY                             |
|                    락킹 전략 선택                                       |
+-----------------------------------------------------------------------+

  Decision Flow:
  결정 흐름:

             High conflict likelihood?
             충돌 가능성 높음?
                    |
          +--------+---------+
          |                  |
         YES                 NO
          |                  |
          v                  v
    PESSIMISTIC         Is response time
    비관적               critical?
                        응답 시간 중요?
                              |
                    +--------+--------+
                    |                 |
                   YES               NO
                    |                 |
                    v                 v
              OPTIMISTIC        Either works
              낙관적            둘 다 가능

  +------------------+------------------+------------------+
  |    Scenario      |   Pessimistic    |   Optimistic     |
  |    시나리오      |   비관적          |   낙관적         |
  +------------------+------------------+------------------+
  | Bank transfer    |       ✓          |                  |
  | 은행 이체        |                  |                  |
  +------------------+------------------+------------------+
  | Inventory mgmt   |       ✓          |       △         |
  | 재고 관리        |                  |   (low volume)   |
  +------------------+------------------+------------------+
  | Shopping cart    |                  |       ✓          |
  | 장바구니         |                  |                  |
  +------------------+------------------+------------------+
  | User profile     |                  |       ✓          |
  | 사용자 프로필    |                  |                  |
  +------------------+------------------+------------------+
  | Seat booking     |       ✓          |                  |
  | 좌석 예약        |                  |                  |
  +------------------+------------------+------------------+
  | Blog comments    |                  |       ✓          |
  | 블로그 댓글      |                  |                  |
  +------------------+------------------+------------------+
```

---

## Summary (요약)

```
+-----------------------------------------------------------------------+
|                        CHAPTER 16 SUMMARY                              |
|                          제16장 요약                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|  LOCK TYPES (락 유형):                                                 |
|  +------------------+--------------------------------------------+    |
|  | Shared (S)       | Multiple readers allowed, writers blocked  |    |
|  | 공유 락          | 여러 읽기자 허용, 쓰기자 차단                 |    |
|  +------------------+--------------------------------------------+    |
|  | Exclusive (X)    | Only one transaction, all others blocked   |    |
|  | 배타적 락        | 하나의 트랜잭션만, 다른 모든 트랜잭션 차단    |    |
|  +------------------+--------------------------------------------+    |
|                                                                       |
|  TWO-PHASE LOCKING (2단계 락킹):                                       |
|  - Growing Phase: Acquire locks (확장 단계: 락 획득)                   |
|  - Shrinking Phase: Release locks (축소 단계: 락 해제)                 |
|  - Strict 2PL: Hold all locks until commit (커밋까지 모든 락 유지)     |
|                                                                       |
|  DEADLOCK (데드락):                                                    |
|  - Circular wait for locks (락에 대한 순환 대기)                        |
|  - Solutions: Prevention, Detection, Timeout                          |
|              방지, 감지, 타임아웃                                       |
|  - Best Practice: Consistent lock ordering                            |
|                   일관된 락 순서 지정                                   |
|                                                                       |
|  PESSIMISTIC VS OPTIMISTIC (비관적 vs 낙관적):                          |
|  +------------------+------------------+--------------------+          |
|  |                  | Pessimistic      | Optimistic         |          |
|  +------------------+------------------+--------------------+          |
|  | When             | High conflict    | Low conflict       |          |
|  | 언제             | 높은 충돌        | 낮은 충돌           |          |
|  +------------------+------------------+--------------------+          |
|  | How              | DB locks         | Version column     |          |
|  | 어떻게           | DB 락            | 버전 컬럼           |          |
|  +------------------+------------------+--------------------+          |
|  | Tradeoff         | Less concurrency | Retry on conflict  |          |
|  | 트레이드오프     | 낮은 동시성      | 충돌 시 재시도      |          |
|  +------------------+------------------+--------------------+          |
|                                                                       |
+-----------------------------------------------------------------------+

  REMEMBER: "Choose the right lock for the right situation"
  기억하세요: "상황에 맞는 올바른 락을 선택하세요"
```
