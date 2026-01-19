# Chapter 12. Normalization (정규화)

## Table of Contents (목차)
1. [Normalization Purpose](#1-normalization-purpose-정규화의-목적)
2. [First Normal Form (1NF)](#2-first-normal-form-1nf-제1정규형)
3. [Second Normal Form (2NF)](#3-second-normal-form-2nf-제2정규형)
4. [Third Normal Form (3NF)](#4-third-normal-form-3nf-제3정규형)
5. [Boyce-Codd Normal Form (BCNF)](#5-boyce-codd-normal-form-bcnf)
6. [Normalization Practice](#6-normalization-practice-정규화-실습)
7. [Summary](#7-summary-요약)

---

## 1. Normalization Purpose (정규화의 목적)

### What is Normalization? (정규화란?)

**Normalization** is the process of organizing data in a database to reduce redundancy and improve data integrity.

**정규화**는 데이터베이스에서 중복을 줄이고 데이터 무결성을 향상시키기 위해 데이터를 조직화하는 과정입니다.

```
Normalization Process (정규화 과정)

+-------------------+
| Unnormalized Data |     Redundancy
| (비정규화 데이터)   |     Anomalies
+-------------------+
         |
         v
+-------------------+
|       1NF         |     Atomic values
| (제1정규형)        |     원자값
+-------------------+
         |
         v
+-------------------+
|       2NF         |     No partial FD
| (제2정규형)        |     부분종속 제거
+-------------------+
         |
         v
+-------------------+
|       3NF         |     No transitive FD
| (제3정규형)        |     이행종속 제거
+-------------------+
         |
         v
+-------------------+
|       BCNF        |     Every determinant
| (보이스코드정규형)  |     is candidate key
+-------------------+
```

### Data Anomalies (데이터 이상현상)

Anomalies are problems that occur in poorly designed databases.
이상현상은 잘못 설계된 데이터베이스에서 발생하는 문제입니다.

```
+------------------+----------------------------------------+
| Anomaly Type     | Description (설명)                      |
+------------------+----------------------------------------+
| Insertion        | Cannot insert data without other data  |
| (삽입 이상)       | 다른 데이터 없이 삽입 불가                |
+------------------+----------------------------------------+
| Update           | Need to update multiple rows           |
| (갱신 이상)       | 여러 행을 수정해야 함                    |
+------------------+----------------------------------------+
| Deletion         | Deleting data causes loss of other data|
| (삭제 이상)       | 삭제 시 다른 데이터도 소실               |
+------------------+----------------------------------------+
```

### Example of Anomalies (이상현상 예제)

```
Unnormalized Student_Course Table (비정규화된 학생_수강 테이블)

+------------+---------------+---------------+-----------+-----------+
| student_id | student_name  | student_phone | course_id | course_name|
+------------+---------------+---------------+-----------+-----------+
| S001       | Kim           | 010-1111-1111 | CS101     | Database  |
| S001       | Kim           | 010-1111-1111 | CS102     | Algorithm |
| S002       | Lee           | 010-2222-2222 | CS101     | Database  |
| S003       | Park          | 010-3333-3333 | CS103     | Network   |
+------------+---------------+---------------+-----------+-----------+


1. INSERTION ANOMALY (삽입 이상)
   Problem: Cannot add a new course without a student
   문제: 학생 없이 새 과목 추가 불가

   Example: Want to add "CS104 - Security" but no students yet
   예: "CS104 - 보안" 과목을 추가하고 싶지만 아직 수강생이 없음


2. UPDATE ANOMALY (갱신 이상)
   Problem: Changing Kim's phone requires multiple updates
   문제: Kim의 전화번호 변경 시 여러 행 수정 필요

   Before: Kim's phone is stored twice (S001 rows)
   전: Kim의 전화번호가 두 번 저장됨

   Risk: Inconsistent data if one row not updated
   위험: 한 행만 수정되면 데이터 불일치


3. DELETION ANOMALY (삭제 이상)
   Problem: Deleting Park loses the Network course info
   문제: Park 삭제 시 Network 과목 정보 소실

   S003 is the only student in CS103
   S003이 CS103의 유일한 학생

   If we delete S003, we lose the fact that CS103 exists
   S003 삭제 시 CS103 존재 사실도 소실됨
```

### Benefits of Normalization (정규화의 이점)

```
+------------------+----------------------------------------+
| Benefit          | Description                            |
+------------------+----------------------------------------+
| Reduced          | Each fact stored once                  |
| Redundancy       | 각 사실이 한 번만 저장됨                 |
+------------------+----------------------------------------+
| Data Integrity   | Consistent data across tables          |
|                  | 테이블 간 일관된 데이터                  |
+------------------+----------------------------------------+
| Easier Updates   | Change data in one place               |
|                  | 한 곳에서만 데이터 변경                  |
+------------------+----------------------------------------+
| Smaller Tables   | More efficient storage                 |
|                  | 효율적인 저장공간                        |
+------------------+----------------------------------------+
```

---

## 2. First Normal Form (1NF) (제1정규형)

### 1NF Definition (1NF 정의)

A table is in **First Normal Form (1NF)** if:
테이블이 **제1정규형 (1NF)**이 되려면:

1. All attributes contain only atomic (indivisible) values
   모든 속성이 원자적(분할 불가) 값만 포함
2. Each column contains values of a single type
   각 열은 단일 유형의 값만 포함
3. Each row is unique (has a primary key)
   각 행이 고유함 (기본키 존재)
4. No repeating groups
   반복 그룹 없음

```
1NF Rule: Atomic Values Only
1NF 규칙: 원자값만 허용

+-------------------+          +-------------------+
| Non-Atomic        |          | Atomic            |
| (비원자적)         |    =>    | (원자적)           |
+-------------------+          +-------------------+
| "010-1111,        |          | 010-1111-1111     |
|  010-2222"        |          | (one value)       |
| (multiple values) |          |                   |
+-------------------+          +-------------------+
```

### Before 1NF: Unnormalized Table (1NF 이전: 비정규화 테이블)

```
UNNORMALIZED: Student Table with Multiple Values
비정규화: 다중 값을 가진 학생 테이블

+------------+---------------+------------------------+-----------------+
| student_id | student_name  | phone_numbers          | courses         |
+------------+---------------+------------------------+-----------------+
| S001       | Kim           | 010-1111, 010-2222     | CS101, CS102    |
| S002       | Lee           | 010-3333               | CS101           |
| S003       | Park          | 010-4444, 010-5555,    | CS102, CS103,   |
|            |               | 010-6666               | CS104           |
+------------+---------------+------------------------+-----------------+

Problems (문제점):
1. Multiple values in single cell (한 셀에 여러 값)
2. Difficult to query specific phone/course (특정 전화/과목 조회 어려움)
3. Varying number of values per row (행마다 값 개수가 다름)
```

### Converting to 1NF (1NF로 변환)

#### Method 1: Separate Rows (행 분리)

```
Option A: Create rows for each combination
옵션 A: 각 조합에 대해 행 생성

+------------+---------------+---------------+-----------+
| student_id | student_name  | phone         | course    |
+------------+---------------+---------------+-----------+
| S001       | Kim           | 010-1111      | CS101     |
| S001       | Kim           | 010-1111      | CS102     |
| S001       | Kim           | 010-2222      | CS101     |
| S001       | Kim           | 010-2222      | CS102     |
| S002       | Lee           | 010-3333      | CS101     |
| ...        |               |               |           |
+------------+---------------+---------------+-----------+

Problem: High redundancy!
문제: 높은 중복!
```

#### Method 2: Separate Tables (테이블 분리 - 권장)

```
Option B: Decompose into multiple tables (RECOMMENDED)
옵션 B: 여러 테이블로 분해 (권장)

Table 1: Students (학생)
+------------+---------------+
| student_id | student_name  |
+------------+---------------+
| S001       | Kim           |
| S002       | Lee           |
| S003       | Park          |
+------------+---------------+

Table 2: Student_Phones (학생_전화번호)
+------------+---------------+
| student_id | phone         |
+------------+---------------+
| S001       | 010-1111      |
| S001       | 010-2222      |
| S002       | 010-3333      |
| S003       | 010-4444      |
| S003       | 010-5555      |
| S003       | 010-6666      |
+------------+---------------+

Table 3: Enrollments (수강)
+------------+-----------+
| student_id | course_id |
+------------+-----------+
| S001       | CS101     |
| S001       | CS102     |
| S002       | CS101     |
| S003       | CS102     |
| S003       | CS103     |
| S003       | CS104     |
+------------+-----------+
```

### 1NF Conversion Process (1NF 변환 과정)

```
1NF Conversion Steps (1NF 변환 단계)

Step 1: Identify repeating groups or multi-valued attributes
        반복 그룹 또는 다중값 속성 식별

Step 2: Create separate tables for repeating data
        반복 데이터를 위한 별도 테이블 생성

Step 3: Establish primary keys for all tables
        모든 테이블에 기본키 설정

Step 4: Use foreign keys to maintain relationships
        관계 유지를 위해 외래키 사용


Visualization:

Before 1NF:                     After 1NF:
+-------------------+           +---------------+
| student_id        |           | Students      |
| student_name      |    =>     +---------------+
| phones (multi)    |           | student_id PK |
| courses (multi)   |           | student_name  |
+-------------------+           +---------------+
                                       |
                           +-----------+-----------+
                           |                       |
                    +---------------+       +---------------+
                    | Student_Phones|       | Enrollments   |
                    +---------------+       +---------------+
                    | student_id FK |       | student_id FK |
                    | phone PK      |       | course_id FK  |
                    +---------------+       +---------------+
```

---

## 3. Second Normal Form (2NF) (제2정규형)

### 2NF Definition (2NF 정의)

A table is in **Second Normal Form (2NF)** if:
테이블이 **제2정규형 (2NF)**이 되려면:

1. It is already in 1NF
   이미 1NF임
2. No partial functional dependencies exist
   부분 함수적 종속이 없음
   (Non-key attributes depend on the ENTIRE primary key)
   (비키 속성이 전체 기본키에 종속)

```
2NF Rule: No Partial Dependencies
2NF 규칙: 부분 종속 없음

Partial Dependency:
+-------+-------+              +---------+
|   A   |   B   |     but      |    C    |
| (PK part) (PK)|     A alone  |         |
+-------+-------+     determines+---------+
                           C

In 2NF: Every non-key attribute must depend
        on the WHOLE primary key
2NF에서: 모든 비키 속성은 전체 기본키에 종속해야 함
```

### When Does 2NF Apply? (2NF가 적용되는 경우)

```
2NF is relevant ONLY when:
2NF가 관련되는 경우는 오직:

1. Table has a COMPOSITE primary key (복합 기본키가 있을 때)
   PK = {A, B}

2. Non-key attributes exist (비키 속성이 존재할 때)

If PK is a single attribute, table is automatically in 2NF
(assuming it's in 1NF)
PK가 단일 속성이면 자동으로 2NF임 (1NF 가정 시)
```

### Before 2NF Example (2NF 이전 예제)

```
Student_Course Table (학생_수강 테이블)
+------------+-----------+---------------+-------------+-------+
| student_id | course_id | student_name  | course_name | grade |
+------------+-----------+---------------+-------------+-------+
| S001       | CS101     | Kim           | Database    | A     |
| S001       | CS102     | Kim           | Algorithm   | B     |
| S002       | CS101     | Lee           | Database    | A     |
| S002       | CS103     | Lee           | Network     | B     |
| S003       | CS102     | Park          | Algorithm   | A     |
+------------+-----------+---------------+-------------+-------+

Primary Key: {student_id, course_id}

Functional Dependencies:
1. {student_id, course_id} → grade        (Full FD - OK)
2. student_id → student_name              (Partial FD - Problem!)
3. course_id → course_name                (Partial FD - Problem!)

FD Diagram:

student_id ----+----> student_name (Partial!)
               |
               +----> grade (Full)
               |
course_id -----+----> course_name (Partial!)
```

### Converting to 2NF (2NF로 변환)

```
Remove Partial Dependencies by Decomposition
분해를 통해 부분 종속 제거

STEP 1: Identify partial FDs
        부분 함수적 종속 식별

        student_id → student_name
        course_id → course_name

STEP 2: Create separate tables for each partial FD
        각 부분 FD에 대해 별도 테이블 생성

STEP 3: Keep full FD in original table
        전체 FD는 원래 테이블에 유지


Result: Three Tables (결과: 세 개의 테이블)

Table 1: Students (학생)
+------------+---------------+
| student_id | student_name  |
+------------+---------------+
| S001       | Kim           |
| S002       | Lee           |
| S003       | Park          |
+------------+---------------+
PK: student_id
FD: student_id → student_name (Direct)


Table 2: Courses (과목)
+-----------+-------------+
| course_id | course_name |
+-----------+-------------+
| CS101     | Database    |
| CS102     | Algorithm   |
| CS103     | Network     |
+-----------+-------------+
PK: course_id
FD: course_id → course_name (Direct)


Table 3: Enrollments (수강)
+------------+-----------+-------+
| student_id | course_id | grade |
+------------+-----------+-------+
| S001       | CS101     | A     |
| S001       | CS102     | B     |
| S002       | CS101     | A     |
| S002       | CS103     | B     |
| S003       | CS102     | A     |
+------------+-----------+-------+
PK: {student_id, course_id}
FD: {student_id, course_id} → grade (Full FD)
```

### 2NF Conversion Diagram (2NF 변환 다이어그램)

```
Before 2NF:
+---------------------------------------------------------+
| student_id | course_id | student_name | course_name | grade |
+---------------------------------------------------------+
      |           |            |              |          |
      +-----------+            |              |          |
            |                  |              |          |
        Partial FD         Partial FD      Full FD
                |                |              |
                v                v              |
                                               |
After 2NF:                                     |
                                               |
+------------------+    +----------------+     |
| Students         |    | Courses        |     |
+------------------+    +----------------+     |
| student_id (PK)  |    | course_id (PK) |     |
| student_name     |    | course_name    |     |
+------------------+    +----------------+     |
        |                       |              |
        |     +-----------------+              |
        |     |                                |
        v     v                                v
+----------------------------------------+
| Enrollments                            |
+----------------------------------------+
| student_id (PK, FK)                    |
| course_id (PK, FK)                     |
| grade                                  |
+----------------------------------------+
```

### SQL for 2NF Conversion (2NF 변환 SQL)

```sql
-- Original problematic table
-- 원본 문제 테이블
CREATE TABLE student_course_old (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    student_name VARCHAR(50),
    course_name VARCHAR(100),
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id)
);

-- Convert to 2NF
-- 2NF로 변환

-- Table 1: Students
CREATE TABLE students (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50) NOT NULL
);

-- Table 2: Courses
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL
);

-- Table 3: Enrollments
CREATE TABLE enrollments (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

-- Migrate data
-- 데이터 마이그레이션
INSERT INTO students (student_id, student_name)
SELECT DISTINCT student_id, student_name
FROM student_course_old;

INSERT INTO courses (course_id, course_name)
SELECT DISTINCT course_id, course_name
FROM student_course_old;

INSERT INTO enrollments (student_id, course_id, grade)
SELECT student_id, course_id, grade
FROM student_course_old;
```

---

## 4. Third Normal Form (3NF) (제3정규형)

### 3NF Definition (3NF 정의)

A table is in **Third Normal Form (3NF)** if:
테이블이 **제3정규형 (3NF)**이 되려면:

1. It is already in 2NF
   이미 2NF임
2. No transitive functional dependencies exist
   이행적 함수 종속이 없음
   (Non-key attributes depend ONLY on the primary key, not on other non-key attributes)
   (비키 속성이 다른 비키 속성이 아닌 기본키에만 종속)

```
3NF Rule: No Transitive Dependencies
3NF 규칙: 이행적 종속 없음

Transitive Dependency:
+-------+    direct    +-------+   determines   +-------+
|  PK   |  ========>   |   A   |   ========>    |   B   |
+-------+              +-------+                +-------+
 (Key)                (Non-key)                (Non-key)

PK → A → B means PK determines B THROUGH A
PK → A → B는 PK가 A를 통해 B를 결정함을 의미

In 3NF: Non-key must NOT determine another non-key
3NF에서: 비키가 다른 비키를 결정하면 안 됨
```

### Before 3NF Example (3NF 이전 예제)

```
Employee Table (직원 테이블)
+-------------+---------------+---------------+---------------+
| employee_id | employee_name | department_id | dept_name     |
+-------------+---------------+---------------+---------------+
| E001        | Kim           | D10           | Engineering   |
| E002        | Lee           | D10           | Engineering   |
| E003        | Park          | D20           | Marketing     |
| E004        | Choi          | D20           | Marketing     |
| E005        | Jung          | D30           | Sales         |
+-------------+---------------+---------------+---------------+

Primary Key: employee_id

Functional Dependencies:
1. employee_id → employee_name     (Direct - OK)
2. employee_id → department_id     (Direct - OK)
3. department_id → dept_name       (Non-key → Non-key!)
4. employee_id → dept_name         (Transitive! Through department_id)

FD Diagram:

employee_id ----+----> employee_name (Direct)
    (PK)        |
               +----> department_id -----> dept_name
                           |         (Transitive!)
                      (Non-key)
```

### Converting to 3NF (3NF로 변환)

```
Remove Transitive Dependencies by Decomposition
분해를 통해 이행적 종속 제거

STEP 1: Identify transitive FD
        이행적 함수 종속 식별

        employee_id → department_id → dept_name

STEP 2: Create separate table for the transitive part
        이행적 부분을 위한 별도 테이블 생성

        department_id → dept_name
        becomes the Departments table

STEP 3: Keep foreign key reference in original table
        원본 테이블에 외래키 참조 유지


Result: Two Tables (결과: 두 개의 테이블)

Table 1: Employees (직원)
+-------------+---------------+---------------+
| employee_id | employee_name | department_id |
+-------------+---------------+---------------+
| E001        | Kim           | D10           |
| E002        | Lee           | D10           |
| E003        | Park          | D20           |
| E004        | Choi          | D20           |
| E005        | Jung          | D30           |
+-------------+---------------+---------------+
PK: employee_id
FK: department_id → Departments


Table 2: Departments (부서)
+---------------+---------------+
| department_id | dept_name     |
+---------------+---------------+
| D10           | Engineering   |
| D20           | Marketing     |
| D30           | Sales         |
+---------------+---------------+
PK: department_id
```

### 3NF Conversion Diagram (3NF 변환 다이어그램)

```
Before 3NF:
+----------------------------------------------------------+
| employee_id | employee_name | department_id | dept_name   |
+----------------------------------------------------------+
      |              |               |              |
      |     Direct   |    Direct     |  Transitive  |
      +------------->|              |<-------------|
                              |              |
                              +------+-------+
                                     |
                                Transitive FD
                                     |
                                     v

After 3NF:

+----------------------------------+
| Employees                        |
+----------------------------------+
| employee_id (PK)                 |
| employee_name                    |
| department_id (FK) --------------+
+----------------------------------+      |
                                         |
                                         v
                              +-------------------+
                              | Departments       |
                              +-------------------+
                              | department_id (PK)|
                              | dept_name         |
                              +-------------------+
```

### Another 3NF Example (또 다른 3NF 예제)

```
Order Table Before 3NF (3NF 이전 주문 테이블)
+----------+-------------+---------------+---------------+
| order_id | customer_id | customer_name | customer_city |
+----------+-------------+---------------+---------------+
| O001     | C001        | Kim Corp      | Seoul         |
| O002     | C001        | Kim Corp      | Seoul         |
| O003     | C002        | Lee Inc       | Busan         |
+----------+-------------+---------------+---------------+

Transitive FDs:
order_id → customer_id → customer_name
order_id → customer_id → customer_city


After 3NF (3NF 이후):

Orders (주문):
+----------+-------------+
| order_id | customer_id |
+----------+-------------+
| O001     | C001        |
| O002     | C001        |
| O003     | C002        |
+----------+-------------+

Customers (고객):
+-------------+---------------+---------------+
| customer_id | customer_name | customer_city |
+-------------+---------------+---------------+
| C001        | Kim Corp      | Seoul         |
| C002        | Lee Inc       | Busan         |
+-------------+---------------+---------------+
```

### SQL for 3NF Conversion (3NF 변환 SQL)

```sql
-- Original table with transitive dependency
-- 이행적 종속이 있는 원본 테이블
CREATE TABLE employees_old (
    employee_id VARCHAR(10) PRIMARY KEY,
    employee_name VARCHAR(50),
    department_id VARCHAR(10),
    dept_name VARCHAR(100)
);

-- Convert to 3NF
-- 3NF로 변환

-- Create Departments table first
CREATE TABLE departments (
    department_id VARCHAR(10) PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL UNIQUE
);

-- Create Employees table with FK
CREATE TABLE employees (
    employee_id VARCHAR(10) PRIMARY KEY,
    employee_name VARCHAR(50) NOT NULL,
    department_id VARCHAR(10),
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- Migrate data
-- 데이터 마이그레이션
INSERT INTO departments (department_id, dept_name)
SELECT DISTINCT department_id, dept_name
FROM employees_old;

INSERT INTO employees (employee_id, employee_name, department_id)
SELECT employee_id, employee_name, department_id
FROM employees_old;
```

---

## 5. Boyce-Codd Normal Form (BCNF)

### BCNF Definition (BCNF 정의)

A table is in **Boyce-Codd Normal Form (BCNF)** if:
테이블이 **보이스-코드 정규형 (BCNF)**이 되려면:

1. It is already in 3NF
   이미 3NF임
2. For every functional dependency X → Y, X must be a superkey
   모든 함수적 종속 X → Y에서 X가 슈퍼키여야 함

```
BCNF Rule: Every Determinant is a Superkey
BCNF 규칙: 모든 결정자가 슈퍼키

In BCNF:
For EVERY dependency X → Y in the table:
X must be a candidate key (or superkey)

테이블의 모든 종속 X → Y에 대해:
X가 후보키(또는 슈퍼키)여야 함

3NF vs BCNF:
- 3NF allows: A → B where A is non-key but B is part of key
- BCNF: NO exception, determinant MUST be superkey
```

### When 3NF is Not Enough (3NF가 충분하지 않은 경우)

```
A table can be in 3NF but NOT in BCNF when:
다음과 같은 경우 3NF이지만 BCNF가 아닐 수 있음:

1. There are multiple candidate keys
   여러 후보키가 있을 때
2. Candidate keys overlap (share attributes)
   후보키가 중복(속성 공유)될 때
3. A non-prime attribute is a determinant
   비주요 속성이 결정자일 때

Example Scenario:
Course assignment where:
- One course can have multiple teachers
- One teacher teaches only one course per time slot
- One room hosts only one course per time slot
```

### BCNF Example (BCNF 예제)

```
Course_Schedule Table
+----------+----------+---------+------+
| course   | teacher  | room    | time |
+----------+----------+---------+------+
| CS101    | Kim      | R101    | 9AM  |
| CS101    | Lee      | R102    | 10AM |
| CS102    | Park     | R101    | 10AM |
| CS102    | Kim      | R103    | 11AM |
+----------+----------+---------+------+

Candidate Keys:
- {course, teacher}
- {course, room, time}
- {teacher, time}
- {room, time}

Functional Dependencies:
1. {teacher, time} → room      (teacher teaches in one room at a time)
2. {room, time} → course       (one course per room per time)
3. {course, teacher} → room, time
4. teacher → course            (teacher teaches one course - Problem!)

FD #4: teacher → course
       teacher is NOT a superkey!

This violates BCNF!
이것은 BCNF를 위반함!
```

### Converting to BCNF (BCNF로 변환)

```
BCNF Violation: teacher → course
(teacher is not a superkey)

Decomposition:

Table 1: Teacher_Course (교사_과목)
+----------+----------+
| teacher  | course   |
+----------+----------+
| Kim      | CS101    |
| Lee      | CS101    |
| Park     | CS102    |
+----------+----------+
PK: teacher
FD: teacher → course (Now teacher IS a key!)


Table 2: Schedule (일정)
+----------+---------+------+
| teacher  | room    | time |
+----------+---------+------+
| Kim      | R101    | 9AM  |
| Lee      | R102    | 10AM |
| Park     | R101    | 10AM |
| Kim      | R103    | 11AM |
+----------+---------+------+
PK: {teacher, time}
FD: {teacher, time} → room (teacher, time IS a key!)


Both tables are now in BCNF:
- Every determinant is a superkey
- 모든 결정자가 슈퍼키임
```

### 3NF vs BCNF Comparison (3NF와 BCNF 비교)

```
+------------------+------------------------------------------+
| Aspect           | 3NF                    | BCNF            |
+------------------+------------------------------------------+
| Transitive FD    | Not allowed            | Not allowed     |
| (이행적 종속)     | (허용 안 됨)            | (허용 안 됨)     |
+------------------+------------------------------------------+
| Non-key → Key    | Allowed                | NOT allowed     |
| part dependency  | (허용됨)                | (허용 안 됨)     |
+------------------+------------------------------------------+
| Every determinant| Not required           | Required        |
| is superkey      | (필수 아님)             | (필수)          |
+------------------+------------------------------------------+
| Dependency       | Always possible        | Sometimes       |
| preservation     | (항상 가능)             | impossible      |
|                  |                        | (때때로 불가능)  |
+------------------+------------------------------------------+
| Common in        | Most databases         | Strict cases    |
| practice         | (대부분의 DB)           | (엄격한 경우)    |
+------------------+------------------------------------------+
```

### When to Use BCNF (BCNF 사용 시점)

```
Use BCNF when:
BCNF를 사용하는 경우:

1. Data integrity is critical
   데이터 무결성이 중요할 때

2. Multiple overlapping candidate keys exist
   여러 중복되는 후보키가 있을 때

3. Update anomalies still occur in 3NF
   3NF에서도 갱신 이상이 발생할 때

Use 3NF when:
3NF를 사용하는 경우:

1. Dependency preservation is important
   종속성 보존이 중요할 때

2. Query performance is priority
   쿼리 성능이 우선일 때

3. BCNF decomposition loses too many joins
   BCNF 분해로 조인이 너무 많아질 때
```

---

## 6. Normalization Practice (정규화 실습)

### Practice Example 1: E-commerce Order (실습 예제 1: 전자상거래 주문)

```
Original Unnormalized Table:
원본 비정규화 테이블:

Orders_Raw
+----------+-------------+---------------+-------------+------------+
| order_id | customer_id | customer_name | products    | order_date |
+----------+-------------+---------------+-------------+------------+
| O001     | C01         | Kim Corp      | P1:3, P2:1  | 2024-01-15 |
| O002     | C02         | Lee Inc       | P1:2        | 2024-01-16 |
| O003     | C01         | Kim Corp      | P3:5, P2:2  | 2024-01-17 |
+----------+-------------+---------------+-------------+------------+

Problems:
1. Multi-valued attribute: products (P1:3 means product P1, quantity 3)
2. Redundant customer info
```

#### Step 1: Convert to 1NF (1NF로 변환)

```
Eliminate multi-valued attributes:
다중값 속성 제거:

Orders_1NF
+----------+-------------+---------------+------------+------------+----------+
| order_id | customer_id | customer_name | product_id | quantity   | order_date|
+----------+-------------+---------------+------------+------------+----------+
| O001     | C01         | Kim Corp      | P1         | 3          | 2024-01-15|
| O001     | C01         | Kim Corp      | P2         | 1          | 2024-01-15|
| O002     | C02         | Lee Inc       | P1         | 2          | 2024-01-16|
| O003     | C01         | Kim Corp      | P3         | 5          | 2024-01-17|
| O003     | C01         | Kim Corp      | P2         | 2          | 2024-01-17|
+----------+-------------+---------------+------------+------------+----------+

PK: {order_id, product_id}

Now in 1NF ✓
- All values are atomic
- No repeating groups
```

#### Step 2: Convert to 2NF (2NF로 변환)

```
Identify Partial FDs:
부분 종속 식별:

- order_id → customer_id, customer_name, order_date (Partial!)
- product_id alone doesn't determine anything here
- {order_id, product_id} → quantity (Full FD)

Remove partial dependencies:
부분 종속 제거:

Table: Orders (주문)
+----------+-------------+---------------+------------+
| order_id | customer_id | customer_name | order_date |
+----------+-------------+---------------+------------+
| O001     | C01         | Kim Corp      | 2024-01-15 |
| O002     | C02         | Lee Inc       | 2024-01-16 |
| O003     | C01         | Kim Corp      | 2024-01-17 |
+----------+-------------+---------------+------------+
PK: order_id

Table: Order_Items (주문항목)
+----------+------------+----------+
| order_id | product_id | quantity |
+----------+------------+----------+
| O001     | P1         | 3        |
| O001     | P2         | 1        |
| O002     | P1         | 2        |
| O003     | P3         | 5        |
| O003     | P2         | 2        |
+----------+------------+----------+
PK: {order_id, product_id}

Now in 2NF ✓
```

#### Step 3: Convert to 3NF (3NF로 변환)

```
Identify Transitive FDs in Orders table:
Orders 테이블에서 이행적 종속 식별:

order_id → customer_id → customer_name (Transitive!)

Remove transitive dependency:
이행적 종속 제거:

Table: Customers (고객)
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| C01         | Kim Corp      |
| C02         | Lee Inc       |
+-------------+---------------+
PK: customer_id

Table: Orders (주문)
+----------+-------------+------------+
| order_id | customer_id | order_date |
+----------+-------------+------------+
| O001     | C01         | 2024-01-15 |
| O002     | C02         | 2024-01-16 |
| O003     | C01         | 2024-01-17 |
+----------+-------------+------------+
PK: order_id
FK: customer_id → Customers

Table: Order_Items (주문항목) - unchanged
+----------+------------+----------+
| order_id | product_id | quantity |
+----------+------------+----------+

Now in 3NF ✓
```

### Final Schema (최종 스키마)

```sql
-- Final 3NF Schema
-- 최종 3NF 스키마

CREATE TABLE customers (
    customer_id VARCHAR(10) PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    order_id VARCHAR(10) PRIMARY KEY,
    customer_id VARCHAR(10) NOT NULL,
    order_date DATE NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE products (
    product_id VARCHAR(10) PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE order_items (
    order_id VARCHAR(10),
    product_id VARCHAR(10),
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### Practice Example 2: Student Registration (실습 예제 2: 학생 등록)

```
Original Table:
+------------+-------------+---------------+-------------+------------+
| student_id | course_id   | student_name  | student_addr| course_name|
| instructor | room        | grade         | credits     | dept_name  |
+------------+-------------+---------------+-------------+------------+

Step-by-step normalization:

1NF: Already atomic (assuming)

2NF: Remove partial dependencies
     PK = {student_id, course_id}
     - student_id → student_name, student_addr
     - course_id → course_name, instructor, room, credits, dept_name

3NF: Remove transitive dependencies
     - course_id → instructor → (instructor info)
     - course_id → dept_name → (dept info)

Final Tables:
- Students (student_id, student_name, student_addr)
- Courses (course_id, course_name, credits, instructor_id, dept_id)
- Departments (dept_id, dept_name)
- Instructors (instructor_id, instructor_name, ...)
- Enrollments (student_id, course_id, room, grade)
```

---

## 7. Summary (요약)

### Normalization Forms Hierarchy (정규형 계층)

```
+------------------+
|  Unnormalized    |
+------------------+
        |
        v (Atomic values / 원자값)
+------------------+
|      1NF         |
+------------------+
        |
        v (No partial FD / 부분종속 제거)
+------------------+
|      2NF         |
+------------------+
        |
        v (No transitive FD / 이행종속 제거)
+------------------+
|      3NF         |
+------------------+
        |
        v (Every determinant is superkey / 모든 결정자가 슈퍼키)
+------------------+
|     BCNF         |
+------------------+
```

### Quick Reference Table (빠른 참조표)

| Normal Form | Requirement | Eliminates |
|-------------|-------------|------------|
| **1NF** | Atomic values, no repeating groups | Multi-valued attributes |
| **2NF** | 1NF + No partial FD | Partial dependencies |
| **3NF** | 2NF + No transitive FD | Transitive dependencies |
| **BCNF** | Every determinant is superkey | All FD anomalies |

### Normalization Process Summary (정규화 과정 요약)

```
Step 1: Analyze the table
        테이블 분석
        - Identify PK
        - List all FDs

Step 2: Check 1NF
        1NF 확인
        - Are all values atomic?
        - Split multi-valued attributes

Step 3: Check 2NF (if composite PK)
        2NF 확인 (복합 PK인 경우)
        - Any partial FDs?
        - Decompose partial dependencies

Step 4: Check 3NF
        3NF 확인
        - Any transitive FDs?
        - Decompose transitive dependencies

Step 5: Check BCNF (if needed)
        BCNF 확인 (필요시)
        - Are all determinants superkeys?
        - Decompose non-key determinants
```

### Key Points (핵심 포인트)

1. **Normalization reduces redundancy** but may require more joins
   - 정규화는 중복을 줄이지만 더 많은 조인이 필요할 수 있음

2. **1NF**: Atomic values, unique rows
   - 원자값, 고유 행

3. **2NF**: Full dependency on entire key
   - 전체 키에 대한 완전 종속

4. **3NF**: No non-key depending on non-key
   - 비키가 비키에 종속하면 안 됨

5. **BCNF**: Every determinant must be a key
   - 모든 결정자가 키여야 함

6. **In practice, 3NF is usually sufficient**
   - 실제로 3NF면 대부분 충분함

### When to Denormalize (비정규화 시점)

```
Consider denormalization when:
비정규화를 고려하는 경우:

1. Read performance is critical
   읽기 성능이 중요할 때

2. Joins are too expensive
   조인 비용이 너무 클 때

3. Data rarely changes
   데이터 변경이 거의 없을 때

4. Reporting/Analytics use cases
   보고서/분석 용도

Always balance:
항상 균형을 맞추세요:
- Data integrity vs Performance
- Normalization vs Query complexity
```

---

*End of Chapter 12*
