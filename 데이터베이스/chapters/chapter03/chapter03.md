# Chapter 03. Entity-Relationship Diagram (개체-관계 다이어그램)

> Learn to design databases visually using ERD, understanding entities, attributes, relationships, and various notations.

---

## What is an ERD? (ERD란?)

### Concept (개념)
An Entity-Relationship Diagram (ERD, 개체-관계 다이어그램) is a visual representation of entities and their relationships in a database. It is a conceptual data model used in the database design phase to communicate the structure of data before implementation.

### Purpose of ERD
1. **Conceptual Design (개념적 설계)**: Model real-world entities and relationships
2. **Communication (의사소통)**: Bridge between business requirements and technical implementation
3. **Documentation (문서화)**: Serve as database documentation
4. **Validation (검증)**: Verify data requirements with stakeholders

### ERD Development Process
1. Identify entities (개체 식별)
2. Define attributes for each entity (속성 정의)
3. Identify primary keys (기본키 식별)
4. Establish relationships (관계 설정)
5. Determine cardinality and participation (카디널리티와 참여도 결정)

---

## Entities (개체)

### Concept (개념)
An entity (개체) is a real-world object or concept that can be distinctly identified. Entities become tables in the physical database.

### Types of Entities

#### 1. Strong Entity (강한 개체)
- Can exist independently
- Has its own primary key
- Represented by a rectangle

#### 2. Weak Entity (약한 개체)
- Depends on another entity for identification
- Has a partial key (discriminator)
- Combined with owner's key to form primary key
- Represented by a double rectangle

### Example
```sql
-- Strong Entity: Department
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

-- Weak Entity: Dependent (depends on Employee)
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE dependents (
    employee_id INT,
    dependent_name VARCHAR(100),  -- Partial key (discriminator)
    relationship VARCHAR(50),
    PRIMARY KEY (employee_id, dependent_name),  -- Combined key
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
        ON DELETE CASCADE
);
```

---

## Attributes (속성)

### Concept (개념)
Attributes (속성) are properties that describe an entity. Each attribute has a domain that specifies the set of valid values.

### Types of Attributes

#### 1. Simple Attribute (단순 속성)
- Cannot be divided further
- Example: Age, Price

#### 2. Composite Attribute (복합 속성)
- Can be divided into smaller parts
- Example: Full Name (First Name + Last Name), Address

#### 3. Single-Valued Attribute (단일값 속성)
- Has only one value for each entity
- Example: SSN, Birth Date

#### 4. Multi-Valued Attribute (다중값 속성)
- Can have multiple values for each entity
- Represented by double ellipse in ERD
- Usually normalized into separate table
- Example: Phone Numbers, Skills

#### 5. Derived Attribute (유도 속성)
- Value computed from other attributes
- Represented by dashed ellipse in ERD
- Example: Age (derived from Birth Date)

#### 6. Key Attribute (키 속성)
- Uniquely identifies entity instances
- Underlined in ERD
- Example: Employee ID, ISBN

### Example
```sql
-- Composite Attribute: Address decomposed
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    -- Simple attributes
    email VARCHAR(100),
    birth_date DATE,
    -- Composite attribute broken down
    street VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    zip_code VARCHAR(10),
    -- Derived attribute (typically not stored, calculated when needed)
    -- age is derived from birth_date
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Multi-valued attribute: Separate table
CREATE TABLE customer_phones (
    customer_id INT,
    phone_number VARCHAR(20),
    phone_type VARCHAR(20),  -- 'mobile', 'home', 'work'
    PRIMARY KEY (customer_id, phone_number),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Derived attribute example (using a view)
CREATE VIEW customer_with_age AS
SELECT
    customer_id,
    email,
    birth_date,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age  -- Derived
FROM customers;
```

---

## Relationships (관계)

### Concept (개념)
A relationship (관계) describes how entities are associated with each other. Relationships are represented as diamonds in ERD and implemented through foreign keys.

### Relationship Degree (관계 차수)

#### 1. Unary (Recursive) Relationship (1차/순환 관계)
- Entity related to itself
- Example: Employee manages Employee

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);
```

#### 2. Binary Relationship (2차 관계)
- Two entities involved
- Most common type
- Example: Employee works in Department

#### 3. Ternary Relationship (3차 관계)
- Three entities involved
- Example: Doctor prescribes Medicine to Patient

```sql
CREATE TABLE prescriptions (
    doctor_id INT,
    patient_id INT,
    medicine_id INT,
    prescription_date DATE,
    dosage VARCHAR(100),
    PRIMARY KEY (doctor_id, patient_id, medicine_id, prescription_date),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id),
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (medicine_id) REFERENCES medicines(medicine_id)
);
```

### Relationship Attributes
Relationships can have their own attributes:

```sql
-- Relationship 'works_on' has attribute 'hours'
CREATE TABLE works_on (
    employee_id INT,
    project_id INT,
    hours_per_week DECIMAL(5,2),  -- Relationship attribute
    role VARCHAR(50),              -- Relationship attribute
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id),
    FOREIGN KEY (project_id) REFERENCES projects(project_id)
);
```

---

## Cardinality (카디널리티)

### Concept (개념)
Cardinality (카디널리티) specifies the number of instances of one entity that can be associated with instances of another entity.

### Cardinality Types

#### 1. One-to-One (1:1, 일대일)
- Each entity A instance relates to at most one entity B instance
- Example: Employee has one Passport

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employee_passports (
    passport_id INT PRIMARY KEY,
    employee_id INT UNIQUE,  -- Ensures 1:1
    passport_number VARCHAR(50),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

#### 2. One-to-Many (1:N, 일대다)
- Each entity A instance can relate to many entity B instances
- Entity B instance relates to at most one entity A instance
- Example: Department has many Employees

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,  -- Many employees can have same department_id
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

#### 3. Many-to-Many (M:N, 다대다)
- Each entity A instance can relate to many entity B instances
- Each entity B instance can relate to many entity A instances
- Requires junction table
- Example: Students enroll in many Courses

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    title VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### Participation Constraints (참여 제약조건)

#### Total Participation (전체 참여)
- Every entity instance must participate in relationship
- Represented by double line in ERD
- Implemented with NOT NULL foreign key

#### Partial Participation (부분 참여)
- Some entity instances may not participate
- Represented by single line in ERD
- Foreign key can be NULL

```sql
-- Total participation: Every employee MUST belong to a department
CREATE TABLE employees_total (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT NOT NULL,  -- Cannot be NULL (total participation)
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- Partial participation: Employee MAY have a mentor
CREATE TABLE employees_partial (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    mentor_id INT,  -- Can be NULL (partial participation)
    FOREIGN KEY (mentor_id) REFERENCES employees_partial(employee_id)
);
```

---

## ERD Notations (ERD 표기법)

### Concept (개념)
Different notation styles exist for drawing ERDs. Each has its own symbols and conventions.

### Common Notations

#### 1. Chen Notation (첸 표기법)
- Original ERD notation by Peter Chen (1976)
- Entities: Rectangles
- Attributes: Ellipses
- Relationships: Diamonds
- Lines connect elements

#### 2. Crow's Foot Notation (까마귀발 표기법)
- Most widely used in industry
- Entities: Rectangles with attributes listed inside
- Relationships: Lines with symbols at ends
  - One: Single line
  - Many: Crow's foot (three lines)
  - Mandatory: Solid line
  - Optional: Circle

#### 3. IE (Information Engineering) Notation
- Similar to Crow's Foot
- Used in many CASE tools

#### 4. UML Class Diagram
- Object-oriented approach
- Used when designing with OO methodology

### Crow's Foot Symbols

| Symbol | Meaning | Korean |
|--------|---------|--------|
| `──┤` | Exactly one (mandatory) | 정확히 하나 (필수) |
| `──○┤` | Zero or one (optional) | 0 또는 1 (선택) |
| `──<` | One or many (mandatory) | 1개 이상 (필수) |
| `──○<` | Zero or many (optional) | 0개 이상 (선택) |

---

## ERD Practice (ERD 실습)

### Scenario: Online Bookstore (온라인 서점)

Design an ERD for an online bookstore with the following requirements:
- Customers can place orders
- Each order contains multiple books
- Books are written by authors (a book can have multiple authors)
- Books belong to categories

### Implementation

```sql
-- Entity: Categories
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    description TEXT
);

-- Entity: Authors
CREATE TABLE authors (
    author_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    biography TEXT,
    birth_date DATE
);

-- Entity: Books
CREATE TABLE books (
    book_id INT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    isbn VARCHAR(20) UNIQUE NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    published_date DATE,
    category_id INT,
    stock_quantity INT DEFAULT 0,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

-- Relationship: Book-Author (M:N)
CREATE TABLE book_authors (
    book_id INT,
    author_id INT,
    author_order INT,  -- For ordering multiple authors
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);

-- Entity: Customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Entity: Customer Addresses (Multi-valued attribute)
CREATE TABLE customer_addresses (
    address_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    address_type VARCHAR(20),  -- 'shipping', 'billing'
    street VARCHAR(200),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100),
    is_default BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Entity: Orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'pending',
    shipping_address_id INT,
    total_amount DECIMAL(12,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (shipping_address_id) REFERENCES customer_addresses(address_id)
);

-- Relationship: Order-Book (M:N with attributes)
CREATE TABLE order_items (
    order_id INT,
    book_id INT,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,  -- Price at time of order
    PRIMARY KEY (order_id, book_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id)
);
```

### Sample Queries

```sql
-- Find all books by a specific author
SELECT b.title, b.price, a.name AS author
FROM books b
JOIN book_authors ba ON b.book_id = ba.book_id
JOIN authors a ON ba.author_id = a.author_id
WHERE a.name = 'Stephen King';

-- Get order details with customer info
SELECT
    o.order_id,
    c.email,
    c.first_name,
    c.last_name,
    o.order_date,
    o.total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed';

-- Calculate order total
SELECT
    o.order_id,
    SUM(oi.quantity * oi.unit_price) AS calculated_total
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id;
```

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Entity | 개체 | Real-world object or concept |
| Attribute | 속성 | Property of an entity |
| Relationship | 관계 | Association between entities |
| Cardinality | 카디널리티 | Number of relationship instances |
| Strong Entity | 강한 개체 | Independent entity with own key |
| Weak Entity | 약한 개체 | Dependent entity |
| Total Participation | 전체 참여 | All instances must participate |
| Partial Participation | 부분 참여 | Some instances may participate |
| Chen Notation | 첸 표기법 | Original ERD notation |
| Crow's Foot | 까마귀발 표기법 | Industry-standard notation |

---
