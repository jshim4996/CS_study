# 데이터베이스 학습 계획

> 목표: 데이터베이스 설계, SQL 작성, 성능 최적화를 깊이 있게 이해하는 수준

---

## AI 학습 가이드

### 역할
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### 학습 진행 흐름 (5단계)

**1단계: 진행도 확인** - "오늘 어디 할 차례야"
**2단계: 설명** - "학습 내용 보여줘" → 해당 챕터 .md 읽기
**3단계: 예제** - "예제 보여줘" → examples/ 폴더 확인
**4단계: 과제** - "과제 줘"
**5단계: 평가** - [답안 제출] → 통과 = [x] 체크

### 평가 기준
□ 학습한 개념을 이해하고 적용했는가

---

## STEP 1. 데이터베이스 기초

### Chapter 01. 데이터베이스 개요
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] 데이터베이스란 무엇인가
- [ ] 파일 시스템 vs DBMS
- [ ] DBMS 종류: 관계형, NoSQL
- [ ] 관계형 데이터베이스 개념
- [ ] 스키마와 인스턴스

---

### Chapter 02. 관계형 모델
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] 릴레이션(테이블) 개념
- [ ] 속성, 튜플, 도메인
- [ ] 키 종류: 슈퍼키, 후보키, 기본키, 외래키
- [ ] 관계: 1:1, 1:N, N:M
- [ ] 무결성 제약조건

---

### Chapter 03. ERD - 개체-관계 다이어그램
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] 엔티티와 속성
- [ ] 관계와 카디널리티
- [ ] ERD 표기법: IE, Chen
- [ ] ERD 작성 실습

---

## STEP 2. SQL 기초

### Chapter 04. SQL 개요와 SELECT
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] SQL이란
- [ ] DDL, DML, DCL 구분
- [ ] SELECT 기본 구조
- [ ] WHERE 조건절
- [ ] 비교 연산자, 논리 연산자
- [ ] LIKE, IN, BETWEEN
- [ ] NULL 처리: IS NULL, IS NOT NULL

---

### Chapter 05. 정렬과 집계
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] ORDER BY (정렬)
- [ ] LIMIT, OFFSET (페이징)
- [ ] DISTINCT
- [ ] 집계 함수: COUNT, SUM, AVG, MAX, MIN
- [ ] GROUP BY
- [ ] HAVING

---

### Chapter 06. JOIN
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] JOIN 개념
- [ ] INNER JOIN
- [ ] LEFT JOIN, RIGHT JOIN
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] SELF JOIN
- [ ] 다중 테이블 JOIN

---

### Chapter 07. 서브쿼리
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] 서브쿼리 개념
- [ ] 스칼라 서브쿼리
- [ ] 인라인 뷰: FROM절 서브쿼리
- [ ] 중첩 서브쿼리: WHERE절
- [ ] EXISTS, NOT EXISTS
- [ ] IN vs EXISTS 차이

---

### Chapter 08. 기타 SQL
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] UNION, UNION ALL
- [ ] CASE WHEN
- [ ] 문자열 함수
- [ ] 날짜 함수
- [ ] 형변환 함수

---

## STEP 3. 데이터 조작과 정의

### Chapter 09. DDL - 데이터 정의어
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] CREATE TABLE
- [ ] 데이터 타입 선택
- [ ] 제약조건: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL
- [ ] ALTER TABLE
- [ ] DROP TABLE
- [ ] TRUNCATE vs DELETE

---

### Chapter 10. DML - 데이터 조작어
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] INSERT
- [ ] UPDATE
- [ ] DELETE
- [ ] INSERT ... SELECT
- [ ] UPSERT: INSERT ON DUPLICATE KEY UPDATE / MERGE

---

## STEP 4. 데이터베이스 설계

### Chapter 11. 함수적 종속성
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] 함수적 종속성 개념
- [ ] 완전 함수적 종속
- [ ] 부분 함수적 종속
- [ ] 이행적 함수적 종속

---

### Chapter 12. 정규화
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] 정규화 목적: 이상 현상 제거
- [ ] 제1정규형 (1NF)
- [ ] 제2정규형 (2NF)
- [ ] 제3정규형 (3NF)
- [ ] BCNF
- [ ] 정규화 실습

---

### Chapter 13. 반정규화
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] 반정규화 필요성
- [ ] 반정규화 기법
- [ ] 정규화 vs 반정규화 트레이드오프

---

## STEP 5. 트랜잭션

### Chapter 14. 트랜잭션 기초
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] 트랜잭션 개념
- [ ] ACID 속성
- [ ] 트랜잭션 상태
- [ ] COMMIT, ROLLBACK
- [ ] SAVEPOINT

---

### Chapter 15. 동시성 제어
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] 동시성 문제: Dirty Read, Non-repeatable Read, Phantom Read
- [ ] 트랜잭션 격리 수준
- [ ] Read Uncommitted, Read Committed, Repeatable Read, Serializable
- [ ] 격리 수준별 트레이드오프

---

### Chapter 16. 락
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] 락의 필요성
- [ ] 공유 락 vs 배타 락
- [ ] 2단계 잠금 프로토콜
- [ ] 데드락과 해결 방법
- [ ] 낙관적 락 vs 비관적 락

---

## STEP 6. 성능 최적화

### Chapter 17. 인덱스 기초
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] 인덱스 개념과 필요성
- [ ] 인덱스 동작 원리
- [ ] B-Tree 인덱스
- [ ] B+Tree 인덱스
- [ ] 클러스터드 vs 논클러스터드 인덱스

---

### Chapter 18. 인덱스 설계
> **doc**: `chapters/chapter18/chapter18.md`
> **examples**: `chapters/chapter18/examples/`

- [ ] CREATE INDEX (인덱스 생성)
- [ ] 복합 인덱스
- [ ] 인덱스 선택도
- [ ] 커버링 인덱스
- [ ] 인덱스 설계 전략

---

### Chapter 19. 쿼리 최적화
> **doc**: `chapters/chapter19/chapter19.md`
> **examples**: `chapters/chapter19/examples/`

- [ ] 실행 계획: EXPLAIN
- [ ] 실행 계획 읽는 방법
- [ ] 풀 테이블 스캔 vs 인덱스 스캔
- [ ] 쿼리 튜닝 기법
- [ ] N+1 문제와 해결

---

## STEP 7. 심화 주제

### Chapter 20. 뷰와 저장 프로시저
> **doc**: `chapters/chapter20/chapter20.md`
> **examples**: `chapters/chapter20/examples/`

- [ ] 뷰 개념
- [ ] 뷰 생성과 활용
- [ ] 저장 프로시저 개념
- [ ] 트리거 개념

---

### Chapter 21. NoSQL 개요
> **doc**: `chapters/chapter21/chapter21.md`
> **examples**: `chapters/chapter21/examples/`

- [ ] NoSQL 등장 배경
- [ ] NoSQL 유형: Key-Value, Document, Column, Graph
- [ ] CAP 정리
- [ ] RDBMS vs NoSQL 선택 기준

---

## 실무 연결 포인트

| DB 개념 | 실무 적용 |
|---------|----------|
| ERD/정규화 | 테이블 설계 |
| SQL | 데이터 조회/조작 |
| 트랜잭션 | 데이터 무결성 |
| 인덱스 | 쿼리 성능 |
| 실행 계획 | 슬로우 쿼리 디버깅 |

---

## 나중에 필요할 때

| 주제 | 언제 |
|------|------|
| 파티셔닝, 샤딩 | 대규모 데이터 |
| 레플리케이션 | 읽기 분산, 고가용성 |
| 분산 트랜잭션 | 마이크로서비스 |
