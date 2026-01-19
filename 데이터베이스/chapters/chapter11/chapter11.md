# Chapter 11. Functional Dependencies (함수적 종속성)

## Table of Contents (목차)
1. [Functional Dependency Concept](#1-functional-dependency-concept-함수적-종속성-개념)
2. [Full Functional Dependency](#2-full-functional-dependency-완전-함수적-종속)
3. [Partial Functional Dependency](#3-partial-functional-dependency-부분-함수적-종속)
4. [Transitive Functional Dependency](#4-transitive-functional-dependency-이행적-함수-종속)
5. [Functional Dependency Diagrams](#5-functional-dependency-diagrams-함수적-종속-다이어그램)
6. [Summary](#6-summary-요약)

---

## 1. Functional Dependency Concept (함수적 종속성 개념)

### Definition (정의)

A **functional dependency** (FD) exists when one attribute (or set of attributes) uniquely determines another attribute.

**함수적 종속성**은 하나의 속성(또는 속성 집합)이 다른 속성을 유일하게 결정할 때 존재합니다.

**Notation (표기법):** X → Y

This means: "X functionally determines Y" or "Y is functionally dependent on X"
이는 다음을 의미합니다: "X가 Y를 함수적으로 결정한다" 또는 "Y가 X에 함수적으로 종속된다"

```
Functional Dependency: X → Y

+--------+     determines     +--------+
|   X    | -----------------> |   Y    |
+--------+    (결정한다)       +--------+
Determinant                   Dependent
(결정자)                       (종속자)

If we know X, we can uniquely determine Y
X를 알면 Y를 유일하게 결정할 수 있다
```

### Simple Example (간단한 예제)

Consider a table with student information:
학생 정보 테이블을 고려해봅시다:

```
+------------+-------------+----------------+---------+
| student_id | student_name| department_id  | dept_name|
+------------+-------------+----------------+---------+
| S001       | Kim         | D10            | CS      |
| S002       | Lee         | D20            | Math    |
| S003       | Park        | D10            | CS      |
| S004       | Choi        | D30            | Physics |
+------------+-------------+----------------+---------+

Functional Dependencies identified:
식별된 함수적 종속성:

1. student_id → student_name
   (학번이 학생이름을 결정)

2. student_id → department_id
   (학번이 학과번호를 결정)

3. department_id → dept_name
   (학과번호가 학과이름을 결정)
```

### Determinant and Dependent (결정자와 종속자)

```
X → Y

+-------------------+     +-------------------+
|   Determinant     |     |    Dependent      |
|   (결정자)         | --> |    (종속자)        |
+-------------------+     +-------------------+
| - Left side of FD |     | - Right side of FD|
| - Determines Y    |     | - Determined by X |
| - Like a key      |     | - Like a value    |
+-------------------+     +-------------------+
```

### Types of Functional Dependencies (함수적 종속성의 종류)

```
+----------------------------------+
|    Functional Dependencies       |
+----------------------------------+
              |
    +---------+---------+
    |         |         |
    v         v         v
  Full    Partial   Transitive
  (완전)   (부분)     (이행적)
    |         |         |
    v         v         v
 No part   Part of   Through
 of key    key       another
 determines determines attribute
```

### Properties of Functional Dependencies (함수적 종속성의 성질)

**Armstrong's Axioms (암스트롱의 공리):**

1. **Reflexivity (반사성):** If Y ⊆ X, then X → Y
   - Example: {A, B} → A

2. **Augmentation (증가성):** If X → Y, then XZ → YZ
   - Example: If A → B, then AC → BC

3. **Transitivity (이행성):** If X → Y and Y → Z, then X → Z
   - Example: If A → B and B → C, then A → C

```
Armstrong's Axioms (암스트롱 공리)

1. Reflexivity (반사성)
   {A, B} → A  (subset is determined)

2. Augmentation (증가성)
   A → B
   --------  + C
   AC → BC

3. Transitivity (이행성)
   A → B
   B → C
   --------
   A → C
```

---

## 2. Full Functional Dependency (완전 함수적 종속)

### Definition (정의)

A functional dependency X → Y is a **full functional dependency** if removing any attribute from X means the dependency no longer holds.

함수적 종속 X → Y에서 X의 어떤 속성을 제거해도 더 이상 종속이 성립하지 않으면 **완전 함수적 종속**입니다.

```
Full Functional Dependency (완전 함수적 종속)

{A, B} → C

  +-----+-----+               +-----+
  |  A  |  B  |  ---------->  |  C  |
  +-----+-----+               +-----+
    (Both A AND B needed)
    (A와 B 모두 필요)

  A alone → C?  NO (X)
  B alone → C?  NO (X)
  {A, B} → C?   YES (O)

This is FULL functional dependency
이것이 완전 함수적 종속입니다
```

### Example of Full Functional Dependency (완전 함수적 종속 예제)

```
Order_Items Table (주문항목 테이블)
+----------+------------+----------+-------+
| order_id | product_id | quantity | price |
+----------+------------+----------+-------+
| O001     | P001       | 2        | 50.00 |
| O001     | P002       | 1        | 30.00 |
| O002     | P001       | 3        | 50.00 |
| O002     | P003       | 1        | 25.00 |
+----------+------------+----------+-------+

Primary Key: {order_id, product_id}

Full Functional Dependency:
{order_id, product_id} → quantity

Why is this FULL dependency?
왜 이것이 완전 종속인가?

- order_id alone CANNOT determine quantity
  (주문번호만으로는 수량을 결정할 수 없음)
  O001 has quantities 2 and 1

- product_id alone CANNOT determine quantity
  (제품번호만으로는 수량을 결정할 수 없음)
  P001 has quantities 2 and 3

- BOTH order_id AND product_id are needed
  (두 속성 모두 필요)
```

### Identifying Full FD (완전 함수적 종속 식별)

```
To check if X → Y is a Full FD:
X → Y가 완전 함수적 종속인지 확인하려면:

Step 1: Identify the determinant (X)
        결정자(X) 식별

Step 2: For each attribute A in X:
        X의 각 속성 A에 대해:
        - Check if (X - A) → Y still holds
        - (X - A) → Y가 여전히 성립하는지 확인

Step 3: If removing ANY attribute breaks the FD,
        then it's a Full FD
        어떤 속성을 제거해도 FD가 깨지면
        완전 함수적 종속임

Example:
{student_id, course_id} → grade

Test 1: student_id → grade?
  S001 has grades A, B, C (different courses)
  NO, cannot determine grade

Test 2: course_id → grade?
  CS101 has grades A, B, A (different students)
  NO, cannot determine grade

Conclusion: FULL FD
결론: 완전 함수적 종속
```

### Full FD Diagram (완전 함수적 종속 다이어그램)

```
Enrollment Table
+------------+-----------+-------+
| student_id | course_id | grade |
+------------+-----------+-------+
| S001       | CS101     | A     |
| S001       | CS102     | B     |
| S002       | CS101     | B     |
| S002       | CS103     | A     |
+------------+-----------+-------+

Primary Key (기본키): {student_id, course_id}

Functional Dependency Diagram:

    student_id ----+
                   |
                   +---> grade (Full FD)
                   |     (완전 함수적 종속)
    course_id -----+

Both attributes in the key are required to
determine the grade.
두 키 속성이 모두 성적을 결정하는 데 필요합니다.
```

---

## 3. Partial Functional Dependency (부분 함수적 종속)

### Definition (정의)

A **partial functional dependency** exists when a non-prime attribute is functionally dependent on only a part of a composite primary key.

**부분 함수적 종속**은 비주요 속성이 복합 기본키의 일부에만 함수적으로 종속될 때 존재합니다.

```
Partial Functional Dependency (부분 함수적 종속)

Primary Key: {A, B}
Dependency: A → C (NOT {A, B} → C)

  +-----+               +-----+
  |  A  |  ---------->  |  C  |
  +-----+               +-----+
    |
  +-----+
  |  B  |  (B is not needed for C)
  +-----+    (B는 C에 필요 없음)

Part of the key determines the attribute
키의 일부가 속성을 결정함
```

### Example of Partial FD (부분 함수적 종속 예제)

```
Student_Course Table (학생_수강 테이블)
+------------+-----------+---------------+-----------+
| student_id | course_id | student_name  | grade     |
+------------+-----------+---------------+-----------+
| S001       | CS101     | Kim           | A         |
| S001       | CS102     | Kim           | B         |
| S002       | CS101     | Lee           | B         |
| S002       | CS103     | Lee           | A         |
+------------+-----------+---------------+-----------+

Primary Key: {student_id, course_id}

Functional Dependencies:
1. {student_id, course_id} → grade  (Full FD - 완전 종속)
2. student_id → student_name        (Partial FD - 부분 종속!)

Why is #2 a Partial FD?
왜 #2가 부분 종속인가?

student_name depends ONLY on student_id
student_name은 student_id에만 종속됨

- course_id is NOT needed to determine student_name
- course_id는 student_name을 결정하는 데 필요 없음

This is problematic! (문제가 됨!)
- Data redundancy (데이터 중복)
- Update anomaly (갱신 이상)
```

### Visualizing Partial FD (부분 함수적 종속 시각화)

```
Primary Key: {student_id, course_id}

+-------------+                    +---------------+
| student_id  |--+                 |               |
+-------------+  |   Full FD       |    grade      |
                 +---------------->|               |
+-------------+  |   (완전 종속)    +---------------+
| course_id   |--+
+-------------+


+-------------+                    +---------------+
| student_id  |===================>| student_name  |
+-------------+     Partial FD     +---------------+
                    (부분 종속)
+-------------+
| course_id   |     NOT involved
+-------------+     (관여 안 함)


Problems caused by Partial FD (부분 종속으로 인한 문제):

1. Redundancy (중복):
   Kim's name is stored multiple times
   김 씨의 이름이 여러 번 저장됨

2. Update Anomaly (갱신 이상):
   Changing Kim's name requires multiple updates
   김 씨의 이름 변경 시 여러 행 수정 필요

3. Insertion Anomaly (삽입 이상):
   Cannot add student without course enrollment
   수강 없이 학생 추가 불가
```

### Another Partial FD Example (또 다른 부분 종속 예제)

```
Order_Details Table
+----------+------------+--------------+-------------+---------+
| order_id | product_id | product_name | quantity    | price   |
+----------+------------+--------------+-------------+---------+
| O001     | P001       | Widget A     | 2           | 50.00   |
| O001     | P002       | Widget B     | 1           | 30.00   |
| O002     | P001       | Widget A     | 3           | 50.00   |
+----------+------------+--------------+-------------+---------+

Primary Key: {order_id, product_id}

Functional Dependencies:
- {order_id, product_id} → quantity    (Full FD)
- {order_id, product_id} → price       (Full FD) *if price can vary by order
- product_id → product_name            (Partial FD!)

product_name depends only on product_id,
not on the full primary key.

To fix (해결방법): Normalize to 2NF
Split into two tables (두 테이블로 분리):

Table 1: Order_Items
+----------+------------+----------+-------+
| order_id | product_id | quantity | price |
+----------+------------+----------+-------+

Table 2: Products
+------------+--------------+
| product_id | product_name |
+------------+--------------+
```

### Detecting Partial FD (부분 함수적 종속 탐지)

```
Algorithm to detect Partial FD:
부분 함수적 종속 탐지 알고리즘:

1. Identify the composite primary key
   복합 기본키 식별
   PK = {A, B}

2. For each non-key attribute C:
   각 비키 속성 C에 대해:

3. Check if any proper subset of PK determines C
   PK의 진부분집합이 C를 결정하는지 확인:
   - Does A → C hold?
   - Does B → C hold?

4. If yes → Partial FD exists
   예 → 부분 함수적 종속 존재

Example Check:
PK = {student_id, course_id}
Non-key attribute: student_name

student_id → student_name? YES!
→ Partial FD detected!
```

---

## 4. Transitive Functional Dependency (이행적 함수 종속)

### Definition (정의)

A **transitive functional dependency** exists when a non-prime attribute is functionally dependent on another non-prime attribute, which is dependent on the primary key.

**이행적 함수 종속**은 비주요 속성이 기본키에 종속된 다른 비주요 속성에 함수적으로 종속될 때 존재합니다.

```
Transitive Functional Dependency (이행적 함수 종속)

If: X → Y and Y → Z (where Y is not a key)
Then: X → Z is a TRANSITIVE dependency

X → Y → Z
(X determines Y, Y determines Z)
(X가 Y를 결정, Y가 Z를 결정)

+-----+         +-----+         +-----+
|  X  |  ---->  |  Y  |  ---->  |  Z  |
+-----+  (1)    +-----+  (2)    +-----+
 (Key)        (Non-key)       (Non-key)

X → Z is indirect through Y
X → Z는 Y를 통한 간접 종속
```

### Example of Transitive FD (이행적 함수 종속 예제)

```
Employee Table (직원 테이블)
+-------------+---------------+---------------+----------------+
| employee_id | employee_name | department_id | department_name|
+-------------+---------------+---------------+----------------+
| E001        | Kim           | D10           | Engineering    |
| E002        | Lee           | D10           | Engineering    |
| E003        | Park          | D20           | Marketing      |
| E004        | Choi          | D20           | Marketing      |
+-------------+---------------+---------------+----------------+

Primary Key: employee_id

Functional Dependencies:
1. employee_id → employee_name    (Direct - 직접)
2. employee_id → department_id    (Direct - 직접)
3. department_id → department_name (Non-key → Non-key)
4. employee_id → department_name   (Transitive! 이행적!)

Dependency Chain (종속 체인):
employee_id → department_id → department_name
      |              |              |
    (Key)       (Non-key)      (Non-key)

This is TRANSITIVE because:
이것이 이행적인 이유:
- department_name depends on department_id
- department_id depends on employee_id
- Therefore: employee_id → department_name (through department_id)
```

### Visualizing Transitive FD (이행적 함수 종속 시각화)

```
Employee Table Dependency Diagram

+---------------+
| employee_id   |
| (Primary Key) |
+---------------+
       |
       | Direct FD (직접 종속)
       v
+---------------+        +-------------------+
| employee_name |        | department_id     |
+---------------+        | (Non-key)         |
                         +-------------------+
                                |
                                | Transitive FD
                                | (이행적 종속)
                                v
                         +-------------------+
                         | department_name   |
                         | (Non-key)         |
                         +-------------------+

employee_id → department_id → department_name
            (direct)         (transitive)
            (직접)            (이행적)
```

### Problems with Transitive FD (이행적 종속의 문제점)

```
+-------------------+-------------------------------------------+
| Problem           | Example                                   |
+-------------------+-------------------------------------------+
| Redundancy        | "Engineering" stored multiple times       |
| (중복)             | "Engineering"이 여러 번 저장됨             |
+-------------------+-------------------------------------------+
| Update Anomaly    | Changing dept name needs multiple updates |
| (갱신 이상)        | 부서명 변경 시 여러 행 수정 필요            |
+-------------------+-------------------------------------------+
| Insertion Anomaly | Can't add department without employee     |
| (삽입 이상)        | 직원 없이 부서 추가 불가                    |
+-------------------+-------------------------------------------+
| Deletion Anomaly  | Deleting all dept employees loses dept    |
| (삭제 이상)        | 부서의 모든 직원 삭제 시 부서 정보 소실     |
+-------------------+-------------------------------------------+
```

```
Scenario: Update Anomaly (갱신 이상 시나리오)

Before Update (수정 전):
+-------------+---------------+----------------+
| employee_id | department_id | department_name|
+-------------+---------------+----------------+
| E001        | D10           | Engineering    |
| E002        | D10           | Engineering    |
+-------------+---------------+----------------+

Task: Rename "Engineering" to "Development"
작업: "Engineering"을 "Development"로 변경

Problem: Must update multiple rows!
문제: 여러 행을 수정해야 함!

If only one row updated:
한 행만 수정되면:
+-------------+---------------+----------------+
| E001        | D10           | Development    | <- Updated
| E002        | D10           | Engineering    | <- Inconsistent!
+-------------+---------------+----------------+
Data inconsistency! (데이터 불일치!)
```

### Resolving Transitive FD (이행적 종속 해결)

```
Solution: Decompose into 3NF (3정규형으로 분해)

Original Table (원본 테이블):
+-------------+---------------+---------------+----------------+
| employee_id | employee_name | department_id | department_name|
+-------------+---------------+---------------+----------------+

                    ↓ Decomposition (분해) ↓

Table 1: Employees (직원)
+-------------+---------------+---------------+
| employee_id | employee_name | department_id |
+-------------+---------------+---------------+
| E001        | Kim           | D10           |
| E002        | Lee           | D10           |
+-------------+---------------+---------------+

Table 2: Departments (부서)
+---------------+----------------+
| department_id | department_name|
+---------------+----------------+
| D10           | Engineering    |
| D20           | Marketing      |
+---------------+----------------+

Now:
- No redundancy (중복 없음)
- Update in one place (한 곳에서만 수정)
- Can add departments without employees
```

### Another Transitive FD Example (또 다른 이행적 종속 예제)

```
Book Table (도서 테이블)
+----------+------------+-----------+----------------+
| book_id  | book_title | author_id | author_country |
+----------+------------+-----------+----------------+
| B001     | DB Design  | A01       | Korea          |
| B002     | SQL Guide  | A01       | Korea          |
| B003     | Data Mining| A02       | USA            |
+----------+------------+-----------+----------------+

Primary Key: book_id

Transitive Dependency:
book_id → author_id → author_country

          Direct              Transitive
book_id ---------> author_id ---------> author_country

Solution: Create Author table
해결책: Author 테이블 생성

Books:
+----------+------------+-----------+
| book_id  | book_title | author_id |
+----------+------------+-----------+

Authors:
+-----------+----------------+
| author_id | author_country |
+-----------+----------------+
```

---

## 5. Functional Dependency Diagrams (함수적 종속 다이어그램)

### How to Draw FD Diagrams (FD 다이어그램 그리는 방법)

```
Notation (표기법):

1. Attributes in boxes (속성은 박스로)
   +------------+
   | attribute  |
   +------------+

2. Arrows for dependencies (종속은 화살표로)
   A → B  means  A -----> B

3. Primary key underlined or marked
   (기본키는 밑줄 또는 표시)

4. Composite determinant with bracket
   (복합 결정자는 괄호로)
   {A, B} → C
```

### Complete FD Diagram Example (완전한 FD 다이어그램 예제)

```
Student_Enrollment Table
+------------+-----------+---------------+-------------+-----------+
| student_id | course_id | student_name  | course_name | grade     |
+------------+-----------+---------------+-------------+-----------+

Primary Key: {student_id, course_id}

Functional Dependencies:
1. {student_id, course_id} → grade        (Full FD)
2. student_id → student_name              (Partial FD)
3. course_id → course_name                (Partial FD)

FD Diagram:

+---------------+                    +---------------+
| student_id    |----+          +--->| student_name  |
| (PK part)     |    |          |    | (Partial FD)  |
+---------------+    |          |    +---------------+
      |              |          |
      +--------------+----------+
                     |
                     |    Full FD
                     v    (완전 종속)
               +----------+
               |  grade   |
               +----------+
                     |
      +--------------+----------+
      |              |          |
      |              |          |
+---------------+    |          |    +---------------+
| course_id     |----+          +--->| course_name   |
| (PK part)     |                    | (Partial FD)  |
+---------------+                    +---------------+
```

### FD Diagram for Transitive Dependency (이행적 종속 FD 다이어그램)

```
Employee Table FD Diagram

+----------------+
| employee_id    |
|   (PK)         |
+----------------+
       |
       +--------------------------------+
       |                                |
       v                                v
+----------------+               +----------------+
| employee_name  |               | department_id  |
+----------------+               +----------------+
                                        |
                                        v (Transitive)
                                 +----------------+
                                 | dept_name      |
                                 +----------------+

Legend:
----->  Direct FD (직접 종속)
- - ->  Transitive FD (이행적 종속)
```

### Complex FD Diagram (복잡한 FD 다이어그램)

```
Invoice Table
+------------+-------------+--------------+----------+-------------+
| invoice_no | customer_id | customer_name| item_no  | item_name   |
+------------+-------------+--------------+----------+-------------+
| inv_date   | amount      | discount     | quantity | unit_price  |
+------------+-------------+--------------+----------+-------------+

Primary Key: {invoice_no, item_no}

FD Analysis:

1. invoice_no → customer_id, customer_name, inv_date, discount
   (All invoice-level attributes)

2. customer_id → customer_name
   (Transitive through invoice_no)

3. item_no → item_name, unit_price
   (Partial FD - depends only on item_no)

4. {invoice_no, item_no} → quantity, amount
   (Full FD)


FD Diagram:

    +---------------+                +---------------+
    | invoice_no    |--------------->| inv_date      |
    | (PK part)     |                +---------------+
    +---------------+
           |
           +------------------------>+---------------+
           |                         | customer_id   |
           |                         +---------------+
           |                                |
           |                                v (Transitive)
           |                         +---------------+
           |                         | customer_name |
           |                         +---------------+
           |
           |         Full FD
           +------------+----------->+---------------+
                        |            | quantity      |
    +---------------+   |            +---------------+
    | item_no       |---+            +---------------+
    | (PK part)     |                | amount        |
    +---------------+                +---------------+
           |
           |  Partial FD
           +------------------------>+---------------+
           |                         | item_name     |
           |                         +---------------+
           +------------------------>+---------------+
                                     | unit_price    |
                                     +---------------+
```

### FD Diagram Types Comparison (FD 다이어그램 유형 비교)

```
+------------------------------------------------------------------+
|                    FD Type Comparison                             |
+------------------------------------------------------------------+

1. FULL FD (완전 함수적 종속)

   [A, B] ========> C
    (both needed)

   Entire key is needed to determine C
   전체 키가 C를 결정하는 데 필요


2. PARTIAL FD (부분 함수적 종속)

   [A, B]           C
    |
    A ============> C
    (A alone)

   Part of key determines C
   키의 일부가 C를 결정


3. TRANSITIVE FD (이행적 함수 종속)

   A ============> B ============> C
  (PK)          (Non-key)       (Non-key)

   Non-key determines another non-key
   비키가 다른 비키를 결정
+------------------------------------------------------------------+
```

---

## 6. Summary (요약)

### Functional Dependency Types (함수적 종속 유형)

| Type | Definition | Problem | Solution |
|------|------------|---------|----------|
| **Full FD** | Entire key determines attribute | None | Desired state |
| **Partial FD** | Part of key determines attribute | Redundancy | 2NF |
| **Transitive FD** | Non-key determines non-key | Redundancy | 3NF |

### Quick Reference (빠른 참조)

```
Functional Dependency: X → Y
X determines Y / X가 Y를 결정

Full FD:
- {A, B} → C (both A and B needed)
- No part of key alone can determine C
- 키의 일부만으로는 C를 결정할 수 없음

Partial FD:
- A → C (where {A, B} is PK)
- Part of composite key determines C
- 복합키의 일부가 C를 결정

Transitive FD:
- X → Y → Z (X is key, Y is non-key)
- Non-key attribute determines another
- 비키 속성이 다른 속성을 결정
```

### FD and Normalization (함수적 종속과 정규화)

```
+------------------+------------------+------------------+
| Normal Form      | Eliminates       | Based on         |
+------------------+------------------+------------------+
| 1NF              | Repeating groups | Atomic values    |
| 2NF              | Partial FDs      | Full FDs only    |
| 3NF              | Transitive FDs   | Direct FDs only  |
| BCNF             | All anomalies    | Every determinant|
|                  |                  | is a candidate key|
+------------------+------------------+------------------+
```

### Key Points (핵심 포인트)

1. **Functional Dependency** is the foundation of normalization
   - 함수적 종속은 정규화의 기초입니다

2. **Full FD** is desirable - entire key determines attributes
   - 완전 함수적 종속이 바람직함

3. **Partial FD** causes redundancy - fix with 2NF
   - 부분 함수적 종속은 중복을 야기함 - 2NF로 해결

4. **Transitive FD** causes anomalies - fix with 3NF
   - 이행적 함수 종속은 이상현상을 야기함 - 3NF로 해결

5. Understanding FDs is essential for good database design
   - FD 이해는 좋은 데이터베이스 설계에 필수적입니다

### Practice Exercise (연습 문제)

```
Given the table:
주어진 테이블:

Order_Table
+----------+------------+--------------+----------+---------------+
| order_id | product_id | product_name | quantity | customer_name |
+----------+------------+--------------+----------+---------------+

PK: {order_id, product_id}

Identify all functional dependencies:
모든 함수적 종속을 식별하시오:

1. {order_id, product_id} → quantity    (Type: _____)
2. product_id → product_name            (Type: _____)
3. order_id → customer_name             (Type: _____)

Answers:
1. Full FD (완전 종속)
2. Partial FD (부분 종속)
3. Partial FD (부분 종속)
```

---

*End of Chapter 11*
