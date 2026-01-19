# Chapter 02. Relational Model (관계형 모델)

> Understanding the mathematical foundation of relational databases including relations, attributes, tuples, keys, and integrity constraints.

---

## What is a Relation? (릴레이션이란?)

### Concept (개념)
A relation (릴레이션) is a two-dimensional table consisting of rows and columns. In relational database theory, a relation is a set of tuples that have the same attributes. Each relation represents an entity or a relationship between entities.

### Mathematical Definition
A relation R is a subset of the Cartesian product of domains:
- R ⊆ D₁ × D₂ × ... × Dₙ
- Where D₁, D₂, ..., Dₙ are domains (possible values for each attribute)

### Properties of Relations
1. **No duplicate tuples (중복 튜플 없음)**: Each row is unique
2. **Tuples are unordered (튜플 순서 무관)**: Row order doesn't matter
3. **Attributes are unordered (속성 순서 무관)**: Column order doesn't matter
4. **Atomic values (원자값)**: Each cell contains a single value (1NF)

### Example
```sql
-- A relation representing students
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50),
    major VARCHAR(50),
    enrollment_year INT
);

-- The relation instance
INSERT INTO students VALUES
(1001, 'Park Jiyeon', 'Computer Science', 2022),
(1002, 'Lee Seungho', 'Mathematics', 2023),
(1003, 'Kim Soyeon', 'Computer Science', 2022);
```

---

## Attributes, Tuples, and Domains (속성, 튜플, 도메인)

### Concept (개념)

**Attribute (속성)**: A column in a relation that represents a property of the entity. Each attribute has a name and a domain.

**Tuple (튜플)**: A row in a relation representing a single record or instance of the entity.

**Domain (도메인)**: The set of allowable values for an attribute. It defines the data type and constraints for the attribute.

### Terminology Comparison

| Formal Term | Alternative Term | Korean |
|-------------|------------------|--------|
| Relation | Table | 릴레이션/테이블 |
| Tuple | Row, Record | 튜플/행/레코드 |
| Attribute | Column, Field | 속성/열/필드 |
| Domain | Data Type | 도메인 |
| Cardinality | Number of rows | 카디널리티 |
| Degree/Arity | Number of columns | 차수 |

### Example
```sql
-- Domain definitions (conceptually)
-- student_id: INTEGER, range 1000-9999
-- name: VARCHAR(50), non-empty strings
-- gpa: DECIMAL(3,2), range 0.00-4.00

CREATE TABLE student_records (
    student_id INT,          -- Attribute 1
    name VARCHAR(50),        -- Attribute 2
    gpa DECIMAL(3,2)         -- Attribute 3
);

-- Each INSERT creates a tuple
INSERT INTO student_records VALUES (1001, 'Choi Minho', 3.85);
-- Tuple: (1001, 'Choi Minho', 3.85)

-- Degree (차수): 3 (number of attributes)
-- After insertion, Cardinality (카디널리티): 1 (number of tuples)
```

---

## Keys (키)

### Concept (개념)
Keys are attributes or combinations of attributes used to uniquely identify tuples and establish relationships between relations.

### Types of Keys

#### 1. Super Key (슈퍼키)
- Any set of attributes that uniquely identifies tuples
- May contain unnecessary attributes

#### 2. Candidate Key (후보키)
- Minimal super key (no subset is also a super key)
- Must be unique and not null
- A relation can have multiple candidate keys

#### 3. Primary Key (기본키)
- Chosen candidate key to uniquely identify tuples
- Only one per relation
- Cannot be NULL

#### 4. Alternate Key (대체키)
- Candidate keys not chosen as primary key

#### 5. Foreign Key (외래키)
- Attribute that references the primary key of another relation
- Establishes relationships between tables
- Can be NULL (if relationship is optional)

#### 6. Composite Key (복합키)
- Key consisting of two or more attributes

### Example
```sql
-- Primary Key and Candidate Keys
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,           -- Primary Key (기본키)
    ssn VARCHAR(20) UNIQUE NOT NULL,       -- Alternate Key (대체키)
    email VARCHAR(100) UNIQUE NOT NULL,    -- Alternate Key (대체키)
    name VARCHAR(100),
    department_id INT
);

-- Super Keys: {employee_id}, {ssn}, {email}, {employee_id, name}, etc.
-- Candidate Keys: {employee_id}, {ssn}, {email}
-- Primary Key: {employee_id}
-- Alternate Keys: {ssn}, {email}

-- Foreign Key Example
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees_fk (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- Composite Key Example
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id, semester)  -- Composite Key (복합키)
);
```

---

## Relationships (관계)

### Concept (개념)
Relationships (관계) describe how entities (tables) are connected to each other in a database. Relationships are implemented using foreign keys.

### Types of Relationships

#### 1. One-to-One (1:1, 일대일)
- Each record in Table A relates to exactly one record in Table B
- Example: Person and Passport

```sql
CREATE TABLE persons (
    person_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE passports (
    passport_id INT PRIMARY KEY,
    person_id INT UNIQUE,  -- UNIQUE ensures 1:1
    passport_number VARCHAR(20),
    FOREIGN KEY (person_id) REFERENCES persons(person_id)
);
```

#### 2. One-to-Many (1:N, 일대다)
- Each record in Table A can relate to multiple records in Table B
- Most common relationship type
- Example: Department and Employees

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,  -- No UNIQUE, allows many employees per department
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

#### 3. Many-to-Many (M:N, 다대다)
- Records in Table A can relate to multiple records in Table B and vice versa
- Requires a junction table (연결 테이블)
- Example: Students and Courses

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Junction table for M:N relationship
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

---

## Integrity Constraints (무결성 제약조건)

### Concept (개념)
Integrity constraints (무결성 제약조건) are rules that maintain the accuracy and consistency of data in a database. The DBMS enforces these constraints automatically.

### Types of Integrity Constraints

#### 1. Domain Constraint (도메인 제약조건)
- Values must belong to the attribute's domain
- Enforced through data types and CHECK constraints

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    price DECIMAL(10,2) CHECK (price >= 0),  -- Domain constraint
    quantity INT CHECK (quantity >= 0),
    status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'discontinued'))
);
```

#### 2. Entity Integrity Constraint (개체 무결성 제약조건)
- Primary key cannot be NULL
- Primary key must be unique
- Ensures each tuple is uniquely identifiable

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,  -- Cannot be NULL, must be unique
    name VARCHAR(100) NOT NULL
);

-- This would fail:
-- INSERT INTO customers (customer_id, name) VALUES (NULL, 'Test');
```

#### 3. Referential Integrity Constraint (참조 무결성 제약조건)
- Foreign key must reference an existing primary key or be NULL
- Prevents orphan records
- Maintained through CASCADE, SET NULL, or RESTRICT options

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE SET NULL      -- When customer deleted, set to NULL
        ON UPDATE CASCADE       -- When customer_id changes, update here too
);

-- Alternative: ON DELETE RESTRICT (prevent deletion if referenced)
-- Alternative: ON DELETE CASCADE (delete related records)
```

#### 4. Key Constraint (키 제약조건)
- Ensures uniqueness of candidate keys
- Enforced through UNIQUE constraint

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,  -- Key constraint
    email VARCHAR(100) UNIQUE NOT NULL     -- Key constraint
);
```

#### 5. NOT NULL Constraint (NOT NULL 제약조건)
- Attribute must have a value
- Cannot be empty

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,    -- Required field
    email VARCHAR(100),            -- Optional field (can be NULL)
    hire_date DATE NOT NULL        -- Required field
);
```

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Relation | 릴레이션 | Table with rows and columns |
| Tuple | 튜플 | A row in a relation |
| Attribute | 속성 | A column in a relation |
| Domain | 도메인 | Set of allowable values |
| Primary Key | 기본키 | Main identifier for tuples |
| Foreign Key | 외래키 | Reference to another table's primary key |
| Candidate Key | 후보키 | Minimal unique identifier |
| Entity Integrity | 개체 무결성 | Primary key cannot be NULL |
| Referential Integrity | 참조 무결성 | Foreign key must reference valid primary key |

---
