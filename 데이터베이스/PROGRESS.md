# Database Study Plan (데이터베이스 학습 계획)

> Goal: Deeply understand database design, SQL, and performance optimization
> 목표: 데이터베이스 설계, SQL 작성, 성능 최적화를 깊이 있게 이해하는 수준

---

## AI Study Guide (AI 학습 가이드)

### Role (역할)
- AI is a **senior developer and professor** teaching this curriculum
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### Language Rule (언어 규칙)
- **Default: Explain in English (기본: 영어로 설명)**
- When user says "번역" or "translate" → Provide Korean translation

### Study Flow (학습 진행 흐름) - 5 Steps

**Step 1: Check Progress** - "What's next?"
**Step 2: Explanation** - "Explain this" → Read chapter .md
**Step 3: Example** - "Show example" → Check examples/ folder
**Step 4: Assignment** - "Give me an assignment"
**Step 5: Evaluation** - [Submit answer/SQL] → Pass = [x] check

### Evaluation Criteria (평가 기준)
□ Did you understand and apply the concept? (학습한 개념을 이해하고 적용했는가)

---

## STEP 1. Database Fundamentals (데이터베이스 기초)

### Chapter 01. Introduction to Databases (데이터베이스 개요)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] What is a Database (데이터베이스란 무엇인가)
- [ ] File System vs DBMS (파일 시스템 vs DBMS)
- [ ] DBMS Types: Relational, NoSQL (DBMS 종류)
- [ ] Relational Database Concept (관계형 데이터베이스 개념)
- [ ] Schema and Instance (스키마와 인스턴스)

---

### Chapter 02. Relational Model (관계형 모델)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Relation (Table) Concept (릴레이션 개념)
- [ ] Attributes, Tuples, Domains (속성, 튜플, 도메인)
- [ ] Key Types: Super, Candidate, Primary, Foreign (키 종류)
- [ ] Relationships: 1:1, 1:N, N:M (관계)
- [ ] Integrity Constraints (무결성 제약조건)

---

### Chapter 03. ERD - Entity-Relationship Diagram
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] Entities and Attributes (엔티티와 속성)
- [ ] Relationships and Cardinality (관계와 카디널리티)
- [ ] ERD Notations: IE, Chen (ERD 표기법)
- [ ] ERD Practice (ERD 작성 실습)

---

## STEP 2. SQL Basics (SQL 기초)

### Chapter 04. SQL Overview and SELECT
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] What is SQL (SQL이란)
- [ ] DDL, DML, DCL Classification (DDL, DML, DCL 구분)
- [ ] SELECT Basic Structure (SELECT 기본 구조)
- [ ] WHERE Clause (WHERE 조건절)
- [ ] Comparison and Logical Operators (비교 연산자, 논리 연산자)
- [ ] LIKE, IN, BETWEEN
- [ ] NULL Handling: IS NULL, IS NOT NULL (NULL 처리)

---

### Chapter 05. Sorting and Aggregation (정렬과 집계)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] ORDER BY (정렬)
- [ ] LIMIT, OFFSET (페이징)
- [ ] DISTINCT
- [ ] Aggregate Functions: COUNT, SUM, AVG, MAX, MIN (집계 함수)
- [ ] GROUP BY
- [ ] HAVING

---

### Chapter 06. JOIN
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] JOIN Concept (JOIN 개념)
- [ ] INNER JOIN
- [ ] LEFT JOIN, RIGHT JOIN
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] SELF JOIN
- [ ] Multiple Table JOIN (다중 테이블 JOIN)

---

### Chapter 07. Subqueries (서브쿼리)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] Subquery Concept (서브쿼리 개념)
- [ ] Scalar Subquery (스칼라 서브쿼리)
- [ ] Inline View: FROM clause subquery (인라인 뷰)
- [ ] Nested Subquery: WHERE clause (중첩 서브쿼리)
- [ ] EXISTS, NOT EXISTS
- [ ] IN vs EXISTS Difference (IN vs EXISTS 차이)

---

### Chapter 08. Other SQL Features (기타 SQL)
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] UNION, UNION ALL
- [ ] CASE WHEN
- [ ] String Functions (문자열 함수)
- [ ] Date Functions (날짜 함수)
- [ ] Type Conversion Functions (형변환 함수)

---

## STEP 3. Data Manipulation and Definition (데이터 조작과 정의)

### Chapter 09. DDL - Data Definition Language
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] CREATE TABLE
- [ ] Data Type Selection (데이터 타입 선택)
- [ ] Constraints: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL (제약조건)
- [ ] ALTER TABLE
- [ ] DROP TABLE
- [ ] TRUNCATE vs DELETE

---

### Chapter 10. DML - Data Manipulation Language
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] INSERT
- [ ] UPDATE
- [ ] DELETE
- [ ] INSERT ... SELECT
- [ ] UPSERT: INSERT ON DUPLICATE KEY UPDATE / MERGE

---

## STEP 4. Database Design (데이터베이스 설계)

### Chapter 11. Functional Dependencies (함수적 종속성)
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] Functional Dependency Concept (함수적 종속성 개념)
- [ ] Full Functional Dependency (완전 함수적 종속)
- [ ] Partial Functional Dependency (부분 함수적 종속)
- [ ] Transitive Functional Dependency (이행적 함수적 종속)

---

### Chapter 12. Normalization (정규화)
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] Normalization Purpose: Eliminating Anomalies (정규화 목적)
- [ ] First Normal Form - 1NF (제1정규형)
- [ ] Second Normal Form - 2NF (제2정규형)
- [ ] Third Normal Form - 3NF (제3정규형)
- [ ] BCNF
- [ ] Normalization Practice (정규화 실습)

---

### Chapter 13. Denormalization (반정규화)
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] Why Denormalization (반정규화 필요성)
- [ ] Denormalization Techniques (반정규화 기법)
- [ ] Normalization vs Denormalization Trade-offs (트레이드오프)

---

## STEP 5. Transactions (트랜잭션)

### Chapter 14. Transaction Basics (트랜잭션 기초)
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] Transaction Concept (트랜잭션 개념)
- [ ] ACID Properties (ACID 속성)
- [ ] Transaction States (트랜잭션 상태)
- [ ] COMMIT, ROLLBACK
- [ ] SAVEPOINT

---

### Chapter 15. Concurrency Control (동시성 제어)
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] Concurrency Problems: Dirty Read, Non-repeatable Read, Phantom Read (동시성 문제)
- [ ] Transaction Isolation Levels (트랜잭션 격리 수준)
- [ ] Read Uncommitted, Read Committed, Repeatable Read, Serializable
- [ ] Isolation Level Trade-offs (격리 수준별 트레이드오프)

---

### Chapter 16. Locking (락)
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] Why Locking (락의 필요성)
- [ ] Shared Lock vs Exclusive Lock (공유 락 vs 배타 락)
- [ ] Two-Phase Locking Protocol (2단계 잠금 프로토콜)
- [ ] Deadlock and Solutions (데드락과 해결 방법)
- [ ] Optimistic vs Pessimistic Locking (낙관적 락 vs 비관적 락)

---

## STEP 6. Performance Optimization (성능 최적화)

### Chapter 17. Index Basics (인덱스 기초)
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] Index Concept and Purpose (인덱스 개념과 필요성)
- [ ] How Indexes Work (인덱스 동작 원리)
- [ ] B-Tree Index
- [ ] B+Tree Index
- [ ] Clustered vs Non-clustered Index (클러스터드 vs 논클러스터드)

---

### Chapter 18. Index Design (인덱스 설계)
> **doc**: `chapters/chapter18/chapter18.md`
> **examples**: `chapters/chapter18/examples/`

- [ ] CREATE INDEX (인덱스 생성)
- [ ] Composite Index (복합 인덱스)
- [ ] Index Selectivity (인덱스 선택도)
- [ ] Covering Index (커버링 인덱스)
- [ ] Index Design Strategies (인덱스 설계 전략)

---

### Chapter 19. Query Optimization (쿼리 최적화)
> **doc**: `chapters/chapter19/chapter19.md`
> **examples**: `chapters/chapter19/examples/`

- [ ] Execution Plan: EXPLAIN (실행 계획)
- [ ] Reading Execution Plans (실행 계획 읽는 방법)
- [ ] Full Table Scan vs Index Scan (풀 테이블 스캔 vs 인덱스 스캔)
- [ ] Query Tuning Techniques (쿼리 튜닝 기법)
- [ ] N+1 Problem and Solutions (N+1 문제와 해결)

---

## STEP 7. Advanced Topics (심화 주제)

### Chapter 20. Views and Stored Procedures (뷰와 저장 프로시저)
> **doc**: `chapters/chapter20/chapter20.md`
> **examples**: `chapters/chapter20/examples/`

- [ ] View Concept (뷰 개념)
- [ ] Creating and Using Views (뷰 생성과 활용)
- [ ] Stored Procedure Concept (저장 프로시저 개념)
- [ ] Trigger Concept (트리거 개념)

---

### Chapter 21. NoSQL Overview
> **doc**: `chapters/chapter21/chapter21.md`
> **examples**: `chapters/chapter21/examples/`

- [ ] Why NoSQL (NoSQL 등장 배경)
- [ ] NoSQL Types: Key-Value, Document, Column, Graph (NoSQL 유형)
- [ ] CAP Theorem (CAP 정리)
- [ ] RDBMS vs NoSQL: When to Choose (선택 기준)

---

## Practical Connections (실무 연결 포인트)

| DB Concept | Practical Application |
|------------|----------------------|
| ERD/Normalization | Table design |
| SQL | Data querying/manipulation |
| Transactions | Data integrity |
| Indexes | Query performance |
| Execution Plans | Slow query debugging |

---

## For Later (나중에 필요할 때)

| Topic | When |
|-------|------|
| Partitioning, Sharding | Large-scale data |
| Replication | Read distribution, HA |
| Distributed Transactions | Microservices |
