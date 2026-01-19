# Chapter 01. Introduction to Databases (데이터베이스 개요)

> A comprehensive overview of database concepts, comparing file systems with DBMS, and understanding the relational database model.

---

## What is a Database? (데이터베이스란?)

### Concept (개념)
A database (데이터베이스) is an organized collection of structured data stored electronically. It allows for efficient storage, retrieval, modification, and deletion of data. A Database Management System (DBMS, 데이터베이스 관리 시스템) is software that interacts with the database, applications, and users to capture and analyze data.

### Key Characteristics
- **Persistence (영속성)**: Data survives after the program that created it terminates
- **Convenience (편의성)**: Easy to store, retrieve, and update data
- **Efficiency (효율성)**: Optimized for large amounts of data
- **Reliability (신뢰성)**: Data remains consistent even during failures
- **Multi-user support (다중 사용자 지원)**: Multiple users can access simultaneously

---

## File System vs DBMS (파일 시스템 vs DBMS)

### Concept (개념)
Before DBMS, applications stored data in file systems. While simple, file systems have significant limitations that DBMS addresses.

### Comparison Table

| Aspect | File System (파일 시스템) | DBMS |
|--------|--------------------------|------|
| Data Redundancy (데이터 중복) | High redundancy | Minimized through normalization |
| Data Inconsistency (데이터 불일치) | Common problem | Maintained through constraints |
| Data Isolation (데이터 고립) | Data scattered in different files | Centralized management |
| Concurrent Access (동시 접근) | Difficult to manage | Built-in concurrency control |
| Security (보안) | File-level only | Fine-grained access control |
| Data Integrity (데이터 무결성) | Hard to enforce | Constraint enforcement |
| Backup/Recovery (백업/복구) | Manual, error-prone | Automated mechanisms |

### Problems with File Systems
1. **Data Redundancy (데이터 중복)**: Same data stored in multiple places
2. **Data Inconsistency (데이터 불일치)**: Updates may not propagate everywhere
3. **Difficulty in Accessing Data (데이터 접근 어려움)**: Need new programs for each query
4. **Data Isolation (데이터 고립)**: Data scattered across files in various formats
5. **Integrity Problems (무결성 문제)**: Hard to enforce business rules
6. **Atomicity Problems (원자성 문제)**: Partial operations can leave data inconsistent
7. **Concurrent Access Anomalies (동시 접근 이상)**: Multiple users may corrupt data

---

## Types of DBMS (DBMS 유형)

### Concept (개념)
DBMS can be categorized based on their data model, which determines how data is organized, stored, and accessed.

### Major Types

1. **Hierarchical DBMS (계층형 DBMS)**
   - Data organized in tree-like structure
   - Parent-child relationships
   - Example: IBM IMS

2. **Network DBMS (네트워크형 DBMS)**
   - More flexible than hierarchical
   - Many-to-many relationships possible
   - Example: IDMS

3. **Relational DBMS (관계형 DBMS)**
   - Data stored in tables (relations)
   - Most widely used today
   - Examples: MySQL, PostgreSQL, Oracle, SQL Server

4. **Object-Oriented DBMS (객체지향 DBMS)**
   - Stores objects rather than data in tables
   - Example: ObjectDB

5. **NoSQL DBMS (비관계형 DBMS)**
   - Designed for specific data models
   - Types: Document, Key-Value, Column-family, Graph
   - Examples: MongoDB, Redis, Cassandra, Neo4j

---

## Relational Database (관계형 데이터베이스)

### Concept (개념)
A relational database (관계형 데이터베이스) organizes data into tables (called relations, 릴레이션) consisting of rows (tuples, 튜플) and columns (attributes, 속성). Tables can be related to each other through common fields.

### Key Features
- Data stored in tables with rows and columns
- Relationships established through foreign keys
- SQL used for querying and manipulation
- ACID properties for transaction management
- Strong data integrity through constraints

### Example
```sql
-- Creating a simple relational table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INT,
    hire_date DATE,
    salary DECIMAL(10, 2)
);

-- Inserting data
INSERT INTO employees (employee_id, name, department_id, hire_date, salary)
VALUES (1, 'Kim Minjun', 10, '2023-01-15', 50000.00);

-- Querying data
SELECT name, salary
FROM employees
WHERE department_id = 10;
```

---

## Schema and Instance (스키마와 인스턴스)

### Concept (개념)

**Schema (스키마)**: The logical structure or blueprint of the database. It defines how data is organized including tables, columns, data types, and constraints. Schema changes infrequently.

**Instance (인스턴스)**: The actual data stored in the database at a particular moment in time. Instances change frequently as data is inserted, updated, or deleted.

### Three-Schema Architecture (3단계 스키마 구조)

1. **External Schema (외부 스키마)** - View Level
   - Individual user's view of the database
   - Also called user view or subschema
   - Different users see different portions

2. **Conceptual Schema (개념 스키마)** - Logical Level
   - Overall logical structure of entire database
   - Describes what data is stored and relationships
   - Independent of physical storage

3. **Internal Schema (내부 스키마)** - Physical Level
   - Physical storage structure
   - How data is actually stored on disk
   - Indexes, storage allocation, access paths

### Example
```sql
-- Schema definition (structure)
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2),
    stock_quantity INT
);

-- Instance (actual data at a point in time)
-- The following represents the instance
SELECT * FROM products;

/*
| product_id | product_name | price  | stock_quantity |
|------------|--------------|--------|----------------|
| 1          | Laptop       | 999.99 | 50             |
| 2          | Mouse        | 29.99  | 200            |
| 3          | Keyboard     | 79.99  | 150            |
*/
```

### Data Independence (데이터 독립성)

The three-schema architecture provides two types of data independence:

1. **Physical Data Independence (물리적 데이터 독립성)**
   - Changes to internal schema don't affect conceptual schema
   - Can change storage structures without changing logical design

2. **Logical Data Independence (논리적 데이터 독립성)**
   - Changes to conceptual schema don't affect external schemas
   - Can modify tables without affecting user applications

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Database | 데이터베이스 | Organized collection of data |
| DBMS | 데이터베이스 관리 시스템 | Software to manage databases |
| Relation | 릴레이션 | Table in relational DB |
| Schema | 스키마 | Structure/blueprint of database |
| Instance | 인스턴스 | Actual data at a point in time |
| Data Independence | 데이터 독립성 | Separation of data levels |

---
