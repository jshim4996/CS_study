# Chapter 17. Index Basics (인덱스 기초)

## Overview (개요)

An index is a data structure that improves the speed of data retrieval operations on a database table. Like a book's index that helps you find topics quickly, database indexes help locate rows without scanning the entire table.

인덱스는 데이터베이스 테이블에서 데이터 검색 작업의 속도를 향상시키는 데이터 구조입니다. 책의 색인이 주제를 빠르게 찾는 데 도움을 주는 것처럼, 데이터베이스 인덱스는 전체 테이블을 스캔하지 않고 행을 찾는 데 도움을 줍니다.

---

## 1. Index Concept and Purpose (인덱스 개념과 목적)

### 1.1 Why Indexes are Needed (인덱스가 필요한 이유)

```
+-----------------------------------------------------------------------+
|                    WITHOUT INDEX (인덱스 없이)                          |
+-----------------------------------------------------------------------+

  Query: SELECT * FROM employees WHERE employee_id = 50000;

  Table: employees (100,000 rows)
  테이블: employees (100,000 행)

  +--------+----------+------------+
  | emp_id | name     | department |
  +--------+----------+------------+
  |   1    | Alice    | Sales      |  <-- Check (확인)
  |   2    | Bob      | IT         |  <-- Check (확인)
  |   3    | Carol    | HR         |  <-- Check (확인)
  |   ...  | ...      | ...        |  <-- Check all... (모두 확인...)
  | 50000  | Diana    | Marketing  |  <-- FOUND! (찾음!)
  |   ...  | ...      | ...        |  <-- Still checking... (계속 확인...)
  | 100000 | Zack     | Finance    |  <-- Done (완료)
  +--------+----------+------------+

  FULL TABLE SCAN: O(n) - Must check ALL 100,000 rows!
  전체 테이블 스캔: O(n) - 100,000 행 모두 확인해야 함!

  Time: ~1000ms (example)
  시간: ~1000ms (예시)
```

```
+-----------------------------------------------------------------------+
|                     WITH INDEX (인덱스와 함께)                          |
+-----------------------------------------------------------------------+

  Query: SELECT * FROM employees WHERE employee_id = 50000;

  Index on employee_id (B-Tree structure):
  employee_id에 대한 인덱스 (B-트리 구조):

                      [50000]
                     /       \
              [25000]         [75000]
             /      \         /      \
        [12500]  [37500]  [62500]  [87500]
           |        |        |        |
          ...      ...      ...      ...
           |
    [50000, ptr] -----> Row data (행 데이터)

  INDEX LOOKUP: O(log n) - Only ~17 comparisons for 100,000 rows!
  인덱스 조회: O(log n) - 100,000 행에 대해 약 17번 비교만!

  Time: ~10ms (example) - 100x faster!
  시간: ~10ms (예시) - 100배 빠름!
```

### 1.2 Index Purpose Summary (인덱스 목적 요약)

```
+-----------------------------------------------------------------------+
|                     INDEX PURPOSES                                     |
|                     인덱스 목적                                        |
+-----------------------------------------------------------------------+

  1. SPEED UP DATA RETRIEVAL (데이터 검색 속도 향상)
     - SELECT queries with WHERE clauses
       WHERE 절이 있는 SELECT 쿼리
     - JOIN operations
       JOIN 연산
     - ORDER BY sorting
       ORDER BY 정렬

  2. ENFORCE UNIQUENESS (고유성 강제)
     - PRIMARY KEY automatically creates unique index
       PRIMARY KEY가 자동으로 고유 인덱스 생성
     - UNIQUE constraints create unique indexes
       UNIQUE 제약조건이 고유 인덱스 생성

  3. EFFICIENT RANGE QUERIES (효율적인 범위 쿼리)
     - BETWEEN, <, >, <=, >=
     - Finding ranges of values quickly
       값의 범위를 빠르게 찾기

  Trade-off:
  트레이드오프:

  +------------------------+------------------------+
  |       BENEFIT          |         COST           |
  |       장점             |         비용           |
  +------------------------+------------------------+
  | Faster SELECT queries  | Slower INSERT/UPDATE   |
  | 빠른 SELECT 쿼리       | 느린 INSERT/UPDATE     |
  +------------------------+------------------------+
  | Faster JOINs           | Extra storage space    |
  | 빠른 JOIN              | 추가 저장 공간         |
  +------------------------+------------------------+
  | Faster ORDER BY        | Index maintenance      |
  | 빠른 ORDER BY          | 인덱스 유지보수        |
  +------------------------+------------------------+
```

---

## 2. How Indexes Work (인덱스의 작동 방식)

### 2.1 Basic Index Structure (기본 인덱스 구조)

```
+-----------------------------------------------------------------------+
|                    INDEX STRUCTURE CONCEPT                             |
|                    인덱스 구조 개념                                     |
+-----------------------------------------------------------------------+

  Table: employees                      Index: idx_employee_name
  테이블: employees                      인덱스: idx_employee_name

  +-----+--------+--------+             +--------+-------------+
  | ID  | Name   | Salary |             | Name   | Row Pointer |
  +-----+--------+--------+             +--------+-------------+
  |  1  | Diana  | 50000  |             | Alice  | ---> Row 3  |
  |  2  | Bob    | 60000  |             | Bob    | ---> Row 2  |
  |  3  | Alice  | 55000  |             | Carol  | ---> Row 5  |
  |  4  | Eve    | 52000  |             | Diana  | ---> Row 1  |
  |  5  | Carol  | 58000  |             | Eve    | ---> Row 4  |
  +-----+--------+--------+             +--------+-------------+

  Table is in insertion order           Index is SORTED by Name
  테이블은 삽입 순서                     인덱스는 이름으로 정렬됨

  Query: SELECT * FROM employees WHERE name = 'Carol';

  WITHOUT INDEX:                        WITH INDEX:
  인덱스 없이:                           인덱스와 함께:
  Scan: 1 -> 2 -> 3 -> 4 -> 5           Binary search in index
  스캔: 1 -> 2 -> 3 -> 4 -> 5           인덱스에서 이진 검색
  (Check all 5 rows)                    Find 'Carol' -> Row 5
  (5개 행 모두 확인)                     'Carol' 찾기 -> 행 5
                                        (Only 2-3 comparisons)
                                        (2-3번 비교만)
```

### 2.2 Index Lookup Process (인덱스 조회 과정)

```
+-----------------------------------------------------------------------+
|                    INDEX LOOKUP PROCESS                                |
|                    인덱스 조회 과정                                     |
+-----------------------------------------------------------------------+

  Query: SELECT salary FROM employees WHERE name = 'Carol';

  Step 1: Query Parser identifies WHERE condition
  단계 1: 쿼리 파서가 WHERE 조건 식별

          SELECT salary
          FROM employees
          WHERE name = 'Carol'
                  ^
                  |
            Index exists on 'name'
            'name'에 인덱스 존재

  Step 2: Query Optimizer chooses index lookup
  단계 2: 쿼리 최적화기가 인덱스 조회 선택

          [Index Scan]  vs  [Table Scan]
               ✓ (faster)    ✗

  Step 3: Index lookup finds row pointer
  단계 3: 인덱스 조회로 행 포인터 찾기

          Index: idx_employee_name
          +--------+-------------+
          | ...    |             |
          | Carol  | ---> Row 5  |  <-- Found!
          | ...    |             |      찾음!
          +--------+-------------+

  Step 4: Fetch actual row data using pointer
  단계 4: 포인터를 사용하여 실제 행 데이터 가져오기

          Table: employees
          +-----+--------+--------+
          | ... | ...    | ...    |
          |  5  | Carol  | 58000  |  <-- Fetch this row
          | ... | ...    | ...    |      이 행 가져오기
          +-----+--------+--------+

  Step 5: Return result
  단계 5: 결과 반환

          Result: salary = 58000
```

### 2.3 Index Types Overview (인덱스 유형 개요)

```
+-----------------------------------------------------------------------+
|                      INDEX TYPES                                       |
|                      인덱스 유형                                        |
+-----------------------------------------------------------------------+

  +-------------------+-----------------------------------------------+
  |    Index Type     |    Description / Use Case                     |
  |    인덱스 유형    |    설명 / 사용 사례                            |
  +-------------------+-----------------------------------------------+
  | B-Tree / B+Tree   | Default, most common, good for ranges         |
  |                   | 기본값, 가장 일반적, 범위 검색에 적합           |
  +-------------------+-----------------------------------------------+
  | Hash              | Exact matches only, very fast O(1)            |
  |                   | 정확한 일치만, 매우 빠른 O(1)                   |
  +-------------------+-----------------------------------------------+
  | Full-Text         | Text searching, natural language              |
  |                   | 텍스트 검색, 자연어                            |
  +-------------------+-----------------------------------------------+
  | Spatial/GIS       | Geographic data, locations                    |
  |                   | 지리 데이터, 위치                              |
  +-------------------+-----------------------------------------------+
  | Bitmap            | Low cardinality columns (few distinct values) |
  |                   | 낮은 카디널리티 컬럼 (적은 고유 값)             |
  +-------------------+-----------------------------------------------+

  Most databases default to B-Tree or B+Tree indexes
  대부분의 데이터베이스는 B-Tree 또는 B+Tree 인덱스가 기본값
```

---

## 3. B-Tree Index (B-트리 인덱스)

### 3.1 B-Tree Structure (B-트리 구조)

```
+-----------------------------------------------------------------------+
|                      B-TREE STRUCTURE                                  |
|                       B-트리 구조                                       |
+-----------------------------------------------------------------------+

  B-Tree = Balanced Tree (균형 트리)
  - All leaf nodes at same depth (모든 리프 노드가 같은 깊이)
  - Nodes can have multiple keys (노드가 여러 키를 가질 수 있음)
  - Keys are sorted within nodes (키가 노드 내에서 정렬됨)

  Example: B-Tree with order 3 (max 2 keys per node)
  예시: 차수 3인 B-트리 (노드당 최대 2개 키)

                        +-------+
                        | 30|60 |
                        +-------+
                       /    |    \
                      /     |     \
                     v      v      v
              +-------+  +-------+  +-------+
              | 10|20 |  | 40|50 |  | 70|80 |
              +-------+  +-------+  +-------+
              /  |  \    /  |  \    /  |  \
             v   v   v  v   v   v  v   v   v
            [5][15][25][35][45][55][65][75][90]

  Each leaf node contains:
  각 리프 노드 포함:
  - Key value (키 값)
  - Pointer to actual data (실제 데이터 포인터)

  Properties:
  속성:
  - Height is O(log n) (높이는 O(log n))
  - Each node has at least ceil(m/2) - 1 keys (각 노드는 최소 ceil(m/2) - 1 키)
  - Each node has at most m - 1 keys (각 노드는 최대 m - 1 키)
```

### 3.2 B-Tree Search (B-트리 검색)

```
+-----------------------------------------------------------------------+
|                     B-TREE SEARCH EXAMPLE                              |
|                     B-트리 검색 예시                                    |
+-----------------------------------------------------------------------+

  Find: 45
  찾기: 45

  Step 1: Start at root (루트에서 시작)
                        +-------+
                        | 30|60 |
                        +-------+
                        30 < 45 < 60
                        Go to middle child
                        중간 자식으로 이동

  Step 2: Navigate to middle child (중간 자식으로 이동)
                        +-------+
                        | 40|50 |
                        +-------+
                        40 < 45 < 50
                        Go to middle child
                        중간 자식으로 이동

  Step 3: Find in leaf node (리프 노드에서 찾기)
                        [45] ✓ Found!
                              찾음!

  Comparisons: Only 3 (instead of scanning all data)
  비교: 3번만 (모든 데이터 스캔 대신)

  Time Complexity: O(log n)
  시간 복잡도: O(log n)
```

### 3.3 B-Tree Operations (B-트리 연산)

```sql
-- Creating B-Tree Index (most databases default to B-Tree)
-- B-트리 인덱스 생성 (대부분 데이터베이스에서 B-Tree가 기본값)

-- Single column index
-- 단일 컬럼 인덱스
CREATE INDEX idx_employee_name ON employees(name);

-- Composite index (multiple columns)
-- 복합 인덱스 (여러 컬럼)
CREATE INDEX idx_emp_dept_date ON employees(department, hire_date);

-- Unique index
-- 고유 인덱스
CREATE UNIQUE INDEX idx_employee_email ON employees(email);

-- Query examples that use B-Tree effectively
-- B-트리를 효과적으로 사용하는 쿼리 예시

-- Exact match (정확한 일치)
SELECT * FROM employees WHERE name = 'Alice';

-- Range query (범위 쿼리)
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 70000;

-- Prefix search (접두사 검색)
SELECT * FROM employees WHERE name LIKE 'A%';

-- Ordering (정렬)
SELECT * FROM employees ORDER BY name;

-- Note: These DON'T use index efficiently
-- 참고: 이것들은 인덱스를 효율적으로 사용하지 않음
SELECT * FROM employees WHERE name LIKE '%lice';  -- Suffix search (접미사 검색)
SELECT * FROM employees WHERE UPPER(name) = 'ALICE';  -- Function on column (컬럼에 함수)
```

---

## 4. B+Tree Index (B+트리 인덱스)

### 4.1 B+Tree vs B-Tree (B+트리 vs B-트리)

```
+-----------------------------------------------------------------------+
|                   B-TREE VS B+TREE COMPARISON                          |
|                    B-트리 vs B+트리 비교                                |
+-----------------------------------------------------------------------+

  B-TREE:
  --------
  - Data stored in ALL nodes (모든 노드에 데이터 저장)
  - No leaf level linking (리프 레벨 연결 없음)

                    [30|data30|60|data60]
                   /         |          \
           [10|data10]   [40|data40]   [70|data70]


  B+TREE:
  --------
  - Data stored ONLY in leaf nodes (리프 노드에만 데이터 저장)
  - Leaf nodes are linked (리프 노드가 연결됨)
  - Internal nodes only store keys (내부 노드는 키만 저장)

                        [30 | 60]           <- Keys only (키만)
                       /    |    \
                      /     |     \
               [10|20]   [40|50]   [70|80]   <- Keys only (키만)
                  |         |         |
                  v         v         v
  Leaf:    [10,d]->[20,d]->[30,d]->[40,d]->[50,d]->[60,d]->[70,d]->[80,d]
           ^                                                           ^
           |_____________________linked list___________________________|
                               연결 리스트

  'd' = data/row pointer (데이터/행 포인터)
```

### 4.2 B+Tree Advantages (B+트리 장점)

```
+-----------------------------------------------------------------------+
|                    B+TREE ADVANTAGES                                   |
|                      B+트리 장점                                        |
+-----------------------------------------------------------------------+

  1. BETTER FOR RANGE QUERIES (범위 쿼리에 더 좋음)
     --------------------------------------------------

     Query: SELECT * FROM products WHERE price BETWEEN 20 AND 50;

     B+Tree Leaf Level (linked):
     B+트리 리프 레벨 (연결됨):

     [10]->[15]->[20]->[25]->[30]->[35]->[40]->[45]->[50]->[55]->[60]
                   ^                                  ^
                   |__________________________________|
                   Sequential scan is fast!
                   순차 스캔이 빠름!

     Just follow the links (연결만 따라가면 됨)

  2. MORE KEYS PER NODE (노드당 더 많은 키)
     --------------------------------------------------

     Internal nodes don't store data, so they can fit more keys
     내부 노드가 데이터를 저장하지 않아 더 많은 키를 담을 수 있음

     B-Tree node:   [key1|data1|key2|data2|key3|data3]  (3 keys)
     B+Tree node:   [key1|key2|key3|key4|key5|key6]     (6 keys)

     Result: Shorter tree height, fewer disk I/O
     결과: 더 낮은 트리 높이, 적은 디스크 I/O

  3. ALL DATA AT LEAF LEVEL (모든 데이터가 리프 레벨에)
     --------------------------------------------------

     Consistent access time for all queries
     모든 쿼리에 대해 일관된 접근 시간

     B-Tree: Data might be found at any level (높이에 따라 다름)
     B+Tree: Always traverse to leaf level (항상 리프까지 탐색)
```

### 4.3 B+Tree in Practice (실제 B+트리)

```
+-----------------------------------------------------------------------+
|                 B+TREE IN DATABASE SYSTEMS                             |
|                데이터베이스 시스템의 B+트리                              |
+-----------------------------------------------------------------------+

  Most relational databases use B+Tree:
  대부분의 관계형 데이터베이스가 B+Tree 사용:

  - MySQL InnoDB: B+Tree for all indexes
    MySQL InnoDB: 모든 인덱스에 B+Tree
  - PostgreSQL: B-Tree (similar to B+Tree)
    PostgreSQL: B-Tree (B+Tree와 유사)
  - SQL Server: B-Tree (B+Tree variant)
    SQL Server: B-Tree (B+Tree 변형)
  - Oracle: B-Tree indexes
    Oracle: B-Tree 인덱스

  Typical B+Tree in InnoDB:
  InnoDB의 일반적인 B+Tree:

  +--------------------+
  |   Page Size: 16KB  |
  +--------------------+
  | Keys per page:     |
  | ~500-1000          |
  +--------------------+
  | Tree depth:        |
  | Usually 3-4 levels |
  | 보통 3-4 레벨       |
  +--------------------+

  For 10 million rows:
  1천만 행의 경우:

  Level 1 (Root):     1 page       = 1000 keys
  Level 2:            1000 pages   = 1,000,000 keys
  Level 3 (Leaf):     1000 pages   = 1,000,000,000 potential entries

  Only 3 page reads to find any row!
  어떤 행이든 찾는데 3번의 페이지 읽기만 필요!
```

```sql
-- B+Tree index creation and usage
-- B+트리 인덱스 생성과 사용

-- MySQL: View index information
-- MySQL: 인덱스 정보 확인
SHOW INDEX FROM employees;

-- PostgreSQL: View index details
-- PostgreSQL: 인덱스 상세 정보 확인
SELECT * FROM pg_indexes WHERE tablename = 'employees';

-- Analyze query execution plan
-- 쿼리 실행 계획 분석
EXPLAIN SELECT * FROM employees WHERE employee_id = 12345;

-- MySQL output example:
-- MySQL 출력 예시:
-- +----+-------------+-----------+------+---------------+---------+---------+-------+------+-------+
-- | id | select_type | table     | type | possible_keys | key     | key_len | ref   | rows | Extra |
-- +----+-------------+-----------+------+---------------+---------+---------+-------+------+-------+
-- |  1 | SIMPLE      | employees | ref  | idx_emp_id    | idx_emp | 4       | const |    1 |       |
-- +----+-------------+-----------+------+---------------+---------+---------+-------+------+-------+

-- type=ref means index is being used
-- type=ref는 인덱스가 사용되고 있음을 의미
```

---

## 5. Clustered vs Non-clustered Index (클러스터드 vs 비클러스터드 인덱스)

### 5.1 Clustered Index (클러스터드 인덱스)

```
+-----------------------------------------------------------------------+
|                     CLUSTERED INDEX                                    |
|                     클러스터드 인덱스                                   |
+-----------------------------------------------------------------------+

  Definition: The index that determines the physical order of data
  정의: 데이터의 물리적 순서를 결정하는 인덱스

  - Only ONE clustered index per table (테이블당 하나의 클러스터드 인덱스만)
  - Leaf nodes contain actual data rows (리프 노드가 실제 데이터 행 포함)
  - Data is sorted by clustered index key (데이터가 클러스터드 인덱스 키로 정렬됨)

                   Clustered Index (B+Tree)
                   클러스터드 인덱스 (B+트리)

                        [50 | 100]
                       /     |     \
                      /      |      \
               [25|40]    [70|85]    [120|150]
                  |          |           |
                  v          v           v
        +------------------+------------------+------------------+
        | ID=25, data...   | ID=70, data...   | ID=120, data...  |
        | ID=40, data...   | ID=85, data...   | ID=150, data...  |
        | (sorted by ID)   | (sorted by ID)   | (sorted by ID)   |
        +------------------+------------------+------------------+
              Leaf pages contain complete rows
              리프 페이지가 완전한 행 포함

  In MySQL InnoDB:
  MySQL InnoDB에서:
  - PRIMARY KEY is the clustered index (PRIMARY KEY가 클러스터드 인덱스)
  - If no PK, first UNIQUE NOT NULL index is used
    PK가 없으면 첫 번째 UNIQUE NOT NULL 인덱스 사용
  - If neither exists, InnoDB creates hidden row ID
    둘 다 없으면 InnoDB가 숨겨진 행 ID 생성
```

### 5.2 Non-clustered Index (비클러스터드 인덱스)

```
+-----------------------------------------------------------------------+
|                   NON-CLUSTERED INDEX                                  |
|                   비클러스터드 인덱스                                   |
+-----------------------------------------------------------------------+

  Definition: A separate index structure that points to data rows
  정의: 데이터 행을 가리키는 별도의 인덱스 구조

  - Multiple non-clustered indexes per table (테이블당 여러 개 가능)
  - Leaf nodes contain index keys + pointers to data
    리프 노드가 인덱스 키 + 데이터 포인터 포함
  - Also called "Secondary Index" (보조 인덱스라고도 함)

         Non-clustered Index                    Clustered Index/Table
         비클러스터드 인덱스                     클러스터드 인덱스/테이블

              [M | S]                                    ...
             /   |   \                            +-------------+
            /    |    \                           | ID=3, Bob   |
     [A|D|J] [M|P|R] [S|T|Z]                      +-------------+
         |      |      |                          | ID=7, Alice |
         v      v      v                          +-------------+
    +---------+---------+---------+               | ID=12, Zoe  |
    | Alice:7 | Mike:15 | Sam:22  |               +-------------+
    | David:9 | Pat:18  | Tom:25  |               | ID=15, Mike |
    | John:3  | Rose:20 | Zoe:12  |               +-------------+
    +---------+---------+---------+                     ...
         |
         |  Pointer (lookup)
         |  포인터 (조회)
         v
    Go to clustered index to get full row
    전체 행을 얻기 위해 클러스터드 인덱스로 이동

  In MySQL InnoDB:
  MySQL InnoDB에서:
  - Non-clustered index leaf contains: indexed columns + PRIMARY KEY
    비클러스터드 인덱스 리프 포함: 인덱스된 컬럼 + PRIMARY KEY
  - Uses PK to lookup in clustered index (PK로 클러스터드 인덱스 조회)
```

### 5.3 Comparison (비교)

```
+-----------------------------------------------------------------------+
|           CLUSTERED VS NON-CLUSTERED COMPARISON                        |
|             클러스터드 vs 비클러스터드 비교                              |
+-----------------------------------------------------------------------+

  +-------------------------+------------------+--------------------+
  |        Aspect           |    Clustered     |   Non-clustered    |
  |        측면             |    클러스터드     |   비클러스터드      |
  +-------------------------+------------------+--------------------+
  | Number per table        |    Only 1        |   Multiple         |
  | 테이블당 개수           |    1개만          |   여러 개          |
  +-------------------------+------------------+--------------------+
  | Leaf node contains      |   Complete row   |   Key + Pointer    |
  | 리프 노드 포함          |   완전한 행      |   키 + 포인터      |
  +-------------------------+------------------+--------------------+
  | Data access             |   Direct         |   Indirect         |
  | 데이터 접근             |   직접           |   간접 (2단계)      |
  +-------------------------+------------------+--------------------+
  | Range queries           |   Very efficient |   Less efficient   |
  | 범위 쿼리               |   매우 효율적    |   덜 효율적         |
  +-------------------------+------------------+--------------------+
  | Insert/Update cost      |   Higher         |   Lower            |
  | 삽입/업데이트 비용      |   높음           |   낮음              |
  +-------------------------+------------------+--------------------+
  | Storage                 |   No extra       |   Extra space      |
  | 저장 공간               |   추가 없음      |   추가 공간 필요    |
  +-------------------------+------------------+--------------------+

  Visual: Query using Non-clustered Index
  시각화: 비클러스터드 인덱스를 사용한 쿼리

  SELECT * FROM employees WHERE name = 'Alice';

  Step 1: Search non-clustered index (name)
  단계 1: 비클러스터드 인덱스 검색 (name)
          -> Find 'Alice' -> Get PK = 7

  Step 2: Use PK to search clustered index
  단계 2: PK로 클러스터드 인덱스 검색
          -> Find row where ID = 7 -> Return full row

  This two-step process is called "Bookmark Lookup" or "Key Lookup"
  이 2단계 과정을 "북마크 조회" 또는 "키 조회"라고 함
```

### 5.4 Covering Index (커버링 인덱스)

```
+-----------------------------------------------------------------------+
|                      COVERING INDEX                                    |
|                      커버링 인덱스                                      |
+-----------------------------------------------------------------------+

  Definition: An index that contains all columns needed for a query
  정의: 쿼리에 필요한 모든 컬럼을 포함하는 인덱스

  Benefit: Avoids going to clustered index (no bookmark lookup)
  장점: 클러스터드 인덱스로 갈 필요 없음 (북마크 조회 없음)

  Example:
  예시:

  -- Create covering index
  -- 커버링 인덱스 생성
  CREATE INDEX idx_cover ON employees(name, department, salary);

  -- This query is "covered" by the index
  -- 이 쿼리는 인덱스로 "커버"됨
  SELECT name, department, salary
  FROM employees
  WHERE name = 'Alice';

  Query execution:
  쿼리 실행:

         Non-clustered Index (covering)
         비클러스터드 인덱스 (커버링)

              [name | dept | salary | PK]
                        |
                        v
              Alice | Sales | 50000 | 7  <- All data here!
                                          모든 데이터가 여기!

              No need to access clustered index!
              클러스터드 인덱스 접근 불필요!

  EXPLAIN shows "Using index" or "Index Only Scan"
  EXPLAIN이 "Using index" 또는 "Index Only Scan" 표시
```

```sql
-- Covering Index Examples
-- 커버링 인덱스 예시

-- Create index for specific query pattern
-- 특정 쿼리 패턴을 위한 인덱스 생성
CREATE INDEX idx_order_cover
ON orders(customer_id, order_date, total_amount);

-- This query uses index only (covered)
-- 이 쿼리는 인덱스만 사용 (커버됨)
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123
  AND order_date >= '2024-01-01';

-- Check if covering index is used
-- 커버링 인덱스 사용 여부 확인
EXPLAIN SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123;

-- MySQL: Shows "Using index" in Extra column
-- MySQL: Extra 컬럼에 "Using index" 표시
-- PostgreSQL: Shows "Index Only Scan"
-- PostgreSQL: "Index Only Scan" 표시

-- Include columns in index (SQL Server, PostgreSQL)
-- 인덱스에 컬럼 포함 (SQL Server, PostgreSQL)
-- SQL Server:
CREATE INDEX idx_cover ON orders(customer_id)
INCLUDE (order_date, total_amount);

-- PostgreSQL:
CREATE INDEX idx_cover ON orders(customer_id)
INCLUDE (order_date, total_amount);
```

---

## 6. Index Best Practices (인덱스 모범 사례)

### 6.1 When to Create Indexes (인덱스를 생성해야 할 때)

```
+-----------------------------------------------------------------------+
|                   INDEX CREATION GUIDELINES                            |
|                   인덱스 생성 지침                                      |
+-----------------------------------------------------------------------+

  CREATE INDEX WHEN:
  인덱스 생성 시점:

  ✓ Column is frequently used in WHERE clauses
    컬럼이 WHERE 절에서 자주 사용될 때

  ✓ Column is used in JOIN conditions
    컬럼이 JOIN 조건에서 사용될 때

  ✓ Column is used in ORDER BY or GROUP BY
    컬럼이 ORDER BY나 GROUP BY에서 사용될 때

  ✓ Column has high selectivity (many unique values)
    컬럼이 높은 선택성을 가질 때 (많은 고유 값)

  ✓ Table has many rows and queries return small subset
    테이블에 많은 행이 있고 쿼리가 작은 부분집합을 반환할 때


  AVOID INDEX WHEN:
  인덱스를 피해야 할 때:

  ✗ Table is small (full scan may be faster)
    테이블이 작을 때 (전체 스캔이 더 빠를 수 있음)

  ✗ Column has low selectivity (e.g., gender, status)
    컬럼이 낮은 선택성을 가질 때 (예: 성별, 상태)

  ✗ Column is frequently updated
    컬럼이 자주 업데이트될 때

  ✗ Queries return large portion of table
    쿼리가 테이블의 큰 부분을 반환할 때

  ✗ Table has heavy INSERT operations
    테이블에 많은 INSERT 작업이 있을 때
```

### 6.2 Index Selectivity (인덱스 선택성)

```
+-----------------------------------------------------------------------+
|                     INDEX SELECTIVITY                                  |
|                     인덱스 선택성                                       |
+-----------------------------------------------------------------------+

  Selectivity = Number of distinct values / Total number of rows
  선택성 = 고유 값 수 / 총 행 수

  High Selectivity (Good for index):
  높은 선택성 (인덱스에 좋음):

  +------------------+------------------+--------------+
  | Column           | Distinct Values  | Selectivity  |
  | 컬럼             | 고유 값 수       | 선택성       |
  +------------------+------------------+--------------+
  | employee_id (PK) | 100,000         | 1.0 (100%)   |
  | email            | 99,000          | 0.99 (99%)   |
  | phone            | 95,000          | 0.95 (95%)   |
  +------------------+------------------+--------------+

  Low Selectivity (Poor for index):
  낮은 선택성 (인덱스에 좋지 않음):

  +------------------+------------------+--------------+
  | Column           | Distinct Values  | Selectivity  |
  | 컬럼             | 고유 값 수       | 선택성       |
  +------------------+------------------+--------------+
  | gender           | 2               | 0.00002      |
  | status           | 5               | 0.00005      |
  | is_active        | 2               | 0.00002      |
  +------------------+------------------+--------------+

  Rule of thumb: Selectivity > 10-20% is usually good for indexing
  경험 법칙: 선택성 > 10-20%가 일반적으로 인덱싱에 좋음
```

```sql
-- Check column selectivity
-- 컬럼 선택성 확인

-- MySQL
SELECT
    COUNT(DISTINCT column_name) AS distinct_values,
    COUNT(*) AS total_rows,
    COUNT(DISTINCT column_name) / COUNT(*) AS selectivity
FROM table_name;

-- Example output:
-- 예시 출력:
-- email: selectivity = 0.99 (good for index)
-- gender: selectivity = 0.00002 (poor for index)

-- Composite index selectivity
-- 복합 인덱스 선택성
SELECT
    COUNT(DISTINCT CONCAT(col1, '-', col2)) / COUNT(*) AS combined_selectivity
FROM table_name;
```

---

## Summary (요약)

```
+-----------------------------------------------------------------------+
|                        CHAPTER 17 SUMMARY                              |
|                          제17장 요약                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|  INDEX PURPOSE (인덱스 목적):                                          |
|  - Speed up data retrieval: O(n) -> O(log n)                          |
|    데이터 검색 속도 향상: O(n) -> O(log n)                              |
|  - Enforce uniqueness (고유성 강제)                                    |
|  - Efficient range queries (효율적인 범위 쿼리)                         |
|                                                                       |
|  B-TREE vs B+TREE:                                                    |
|  +---------------+--------------------+------------------------+      |
|  | Feature       | B-Tree             | B+Tree                 |      |
|  | 특징          |                    |                        |      |
|  +---------------+--------------------+------------------------+      |
|  | Data location | All nodes          | Leaf nodes only        |      |
|  | 데이터 위치   | 모든 노드          | 리프 노드만             |      |
|  +---------------+--------------------+------------------------+      |
|  | Leaf linking  | No                 | Yes (linked list)      |      |
|  | 리프 연결     | 없음               | 있음 (연결 리스트)      |      |
|  +---------------+--------------------+------------------------+      |
|  | Range queries | Less efficient     | Very efficient         |      |
|  | 범위 쿼리     | 덜 효율적          | 매우 효율적             |      |
|  +---------------+--------------------+------------------------+      |
|                                                                       |
|  CLUSTERED vs NON-CLUSTERED:                                          |
|  +----------------+------------------+--------------------+           |
|  | Feature        | Clustered        | Non-clustered      |           |
|  | 특징           | 클러스터드        | 비클러스터드        |           |
|  +----------------+------------------+--------------------+           |
|  | Per table      | Only 1           | Multiple           |           |
|  | 테이블당       | 1개만            | 여러 개             |           |
|  +----------------+------------------+--------------------+           |
|  | Contains       | Full row data    | Key + Pointer      |           |
|  | 포함 내용      | 전체 행 데이터   | 키 + 포인터         |           |
|  +----------------+------------------+--------------------+           |
|  | Access         | Direct           | Indirect (2-step)  |           |
|  | 접근 방식      | 직접             | 간접 (2단계)        |           |
|  +----------------+------------------+--------------------+           |
|                                                                       |
|  BEST PRACTICES (모범 사례):                                           |
|  - Index high-selectivity columns (높은 선택성 컬럼에 인덱스)           |
|  - Consider covering indexes (커버링 인덱스 고려)                       |
|  - Don't over-index (과도한 인덱스 지양)                                |
|  - Use EXPLAIN to verify index usage (EXPLAIN으로 인덱스 사용 확인)     |
|                                                                       |
+-----------------------------------------------------------------------+

  REMEMBER: "The right index in the right place makes all the difference"
  기억하세요: "적절한 위치의 올바른 인덱스가 모든 차이를 만듭니다"
```
