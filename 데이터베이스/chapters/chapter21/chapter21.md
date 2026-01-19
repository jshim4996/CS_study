# Chapter 21. NoSQL Overview

> Understand the emergence of NoSQL databases, their types, and when to choose them over traditional RDBMS.

---

## Why NoSQL Emerged? (NoSQL의 등장 배경)

### The Problem with Traditional RDBMS

```
Traditional RDBMS Challenges in Modern Era:

1. Scale Limitations (확장성 한계)
+-------------------+
| Single Server     | ← Vertical scaling has limits
| (Scale Up)        |    (bigger machine = more expensive)
| 64 cores, 512GB   |
+-------------------+

vs NoSQL:
+--------+ +--------+ +--------+ +--------+
|Server 1| |Server 2| |Server 3| |Server 4| ← Horizontal scaling
+--------+ +--------+ +--------+ +--------+   (add more machines)

2. Schema Rigidity (스키마 경직성)
RDBMS:
+----------+----------+----------+
| user_id  | name     | email    | ← All rows must have same structure
| INT      | VARCHAR  | VARCHAR  | ← Schema changes are expensive
+----------+----------+----------+

vs NoSQL (Document):
{ "user_id": 1, "name": "Kim", "email": "kim@mail.com" }
{ "user_id": 2, "name": "Lee", "phone": "010-1234-5678" }
{ "user_id": 3, "name": "Park", "social": { "twitter": "@park" } }
  ↑ Each document can have different structure

3. Performance at Scale (대규모 성능)
- JOIN operations become expensive with billions of rows
- ACID transactions across distributed systems are complex
- Real-time applications need sub-millisecond responses
```

### The Rise of Big Data

```
Data Growth Timeline:
+--------+------------------+------------------------+
| Era    | Data Volume      | Requirements           |
+--------+------------------+------------------------+
| 1990s  | Megabytes        | Structured, ACID       |
| 2000s  | Gigabytes        | Web scale, availability|
| 2010s  | Terabytes        | Real-time, flexible    |
| 2020s  | Petabytes        | AI/ML, streaming       |
+--------+------------------+------------------------+

Big Tech Companies Led the Way:
- Google: BigTable (2006) → Inspired HBase, Cassandra
- Amazon: Dynamo (2007) → Inspired DynamoDB, Riak
- Facebook: Cassandra (2008) → Open sourced
- MongoDB: Founded (2009) → Document database
```

### NoSQL Movement Goals

```
+---------------------------------------------------+
| NoSQL Design Goals                                |
+---------------------------------------------------+
| 1. Horizontal Scalability (수평 확장성)            |
|    - Scale out by adding more nodes               |
|    - Handle massive data volumes                  |
+---------------------------------------------------+
| 2. High Availability (고가용성)                   |
|    - No single point of failure                   |
|    - Continue operating despite failures          |
+---------------------------------------------------+
| 3. Flexible Schema (유연한 스키마)                |
|    - Schema-less or dynamic schema                |
|    - Easy to evolve data model                    |
+---------------------------------------------------+
| 4. Performance at Scale (대규모 성능)             |
|    - Optimized for specific access patterns       |
|    - Low latency for simple operations            |
+---------------------------------------------------+
```

---

## NoSQL Types (NoSQL 유형)

### 1. Key-Value Store (키-값 저장소)

```
Concept:
Simple key → value mapping, like a hash table

+------------+     +--------------------------------+
| Key        | --> | Value                          |
+------------+     +--------------------------------+
| user:1001  | --> | {"name":"Kim","email":"..."}   |
| session:abc| --> | {"userId":1001,"expires":...}  |
| cart:1001  | --> | {"items":[{"id":1,"qty":2}]}   |
+------------+     +--------------------------------+

Characteristics:
- Simplest NoSQL model
- Extremely fast reads/writes
- Values are opaque (no queries within values)
- Perfect for caching and session storage
```

**Examples: Redis, Amazon DynamoDB, Memcached**

```
Redis Example:
# Set a value
SET user:1001 '{"name":"Kim","email":"kim@mail.com"}'

# Get a value
GET user:1001
# Returns: {"name":"Kim","email":"kim@mail.com"}

# Set with expiration (TTL)
SETEX session:abc 3600 '{"userId":1001,"token":"xyz"}'

# Increment counter
INCR pageviews:homepage

# Hash operations
HSET user:1001 name "Kim" email "kim@mail.com"
HGET user:1001 name
# Returns: "Kim"
```

**Use Cases:**
- Session storage
- Caching
- Real-time analytics counters
- Message queues

---

### 2. Document Store (문서 저장소)

```
Concept:
Stores documents (usually JSON/BSON) that can contain nested data

+----------------------------------------------------------+
| Collection: users                                         |
+----------------------------------------------------------+
| {                                                         |
|   "_id": ObjectId("507f1f77bcf86cd799439011"),           |
|   "name": "Kim Minjun",                                  |
|   "email": "kim@mail.com",                               |
|   "address": {                          ← Nested object   |
|     "city": "Seoul",                                     |
|     "zipcode": "06234"                                   |
|   },                                                     |
|   "orders": [                           ← Array           |
|     {"orderId": 1, "total": 50000},                      |
|     {"orderId": 2, "total": 75000}                       |
|   ]                                                      |
| }                                                        |
+----------------------------------------------------------+

Characteristics:
- Documents are self-contained
- Rich query capabilities within documents
- Flexible schema (documents can vary)
- Natural fit for object-oriented programming
```

**Examples: MongoDB, CouchDB, Amazon DocumentDB**

```javascript
// MongoDB Example:

// Insert document
db.users.insertOne({
    name: "Kim Minjun",
    email: "kim@mail.com",
    age: 30,
    address: {
        city: "Seoul",
        district: "Gangnam"
    },
    interests: ["coding", "music", "travel"]
});

// Find documents
db.users.find({ "address.city": "Seoul" });

// Find with multiple conditions
db.users.find({
    age: { $gte: 25, $lte: 35 },
    interests: "coding"
});

// Update document
db.users.updateOne(
    { email: "kim@mail.com" },
    { $set: { age: 31 }, $push: { interests: "gaming" } }
);

// Aggregation pipeline
db.users.aggregate([
    { $match: { "address.city": "Seoul" } },
    { $group: { _id: "$address.district", count: { $sum: 1 } } },
    { $sort: { count: -1 } }
]);
```

**Use Cases:**
- Content management systems
- E-commerce product catalogs
- User profiles
- Real-time analytics

---

### 3. Column-Family Store (컬럼 패밀리 저장소)

```
Concept:
Data stored in columns rather than rows, optimized for reading/writing
large amounts of data across many columns

Row-Oriented (Traditional):
+--------+--------+--------+--------+
| id     | name   | email  | age    |  ← Rows stored together
+--------+--------+--------+--------+
| 1      | Kim    | k@m.co | 30     |  Read entire row at once
| 2      | Lee    | l@m.co | 25     |
| 3      | Park   | p@m.co | 35     |
+--------+--------+--------+--------+

Column-Oriented:
+-----+-----+-----+      +------+------+------+
| id  | id  | id  |      | name | name | name |
| 1   | 2   | 3   |      | Kim  | Lee  | Park |
+-----+-----+-----+      +------+------+------+

+----------+----------+----------+      +-----+-----+-----+
| email    | email    | email    |      | age | age | age |
| k@m.co   | l@m.co   | p@m.co   |      | 30  | 25  | 35  |
+----------+----------+----------+      +-----+-----+-----+

↑ Columns stored together
  Efficient for analytics (read specific columns only)
  Better compression (similar data types together)

Column Family Structure (Cassandra):
+----------------+
| Row Key: user1 |
+----------------+
| Column Family: profile                               |
| +--------+-------+--------+--------+                |
| | name   | email | city   | joined |                |
| | "Kim"  |"k@.." |"Seoul" | 2024   |                |
| +--------+-------+--------+--------+                |
|                                                      |
| Column Family: activity                              |
| +------------+------------+------------+             |
| | login_count| last_login | page_views |             |
| | 150        | 2024-01-15 | 2500       |             |
| +------------+------------+------------+             |
+------------------------------------------------------+
```

**Examples: Apache Cassandra, HBase, ScyllaDB**

```sql
-- Cassandra CQL Example:

-- Create keyspace (like a database)
CREATE KEYSPACE ecommerce
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

-- Create table (column family)
CREATE TABLE ecommerce.users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP
);

-- Create table with compound primary key
CREATE TABLE ecommerce.orders_by_user (
    user_id UUID,
    order_date TIMESTAMP,
    order_id UUID,
    total DECIMAL,
    PRIMARY KEY (user_id, order_date)
) WITH CLUSTERING ORDER BY (order_date DESC);
-- Partition key: user_id (determines which node stores data)
-- Clustering key: order_date (determines order within partition)

-- Insert data
INSERT INTO ecommerce.users (user_id, name, email, created_at)
VALUES (uuid(), 'Kim Minjun', 'kim@mail.com', toTimestamp(now()));

-- Query by partition key (fast)
SELECT * FROM ecommerce.orders_by_user
WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- Query recent orders
SELECT * FROM ecommerce.orders_by_user
WHERE user_id = 550e8400-e29b-41d4-a716-446655440000
AND order_date > '2024-01-01';
```

**Use Cases:**
- Time-series data
- IoT sensor data
- Analytics and data warehousing
- Write-heavy workloads

---

### 4. Graph Database (그래프 데이터베이스)

```
Concept:
Data stored as nodes (entities) and edges (relationships)
Optimized for traversing relationships

+--------+                    +--------+
| Node   |----[KNOWS]-------->| Node   |
| Alice  |                    | Bob    |
+--------+                    +--------+
    |                              |
[WORKS_AT]                    [WORKS_AT]
    |                              |
    v                              v
+--------+                    +--------+
| Node   |                    | Node   |
|Company1|<----[PARTNER]------|Company2|
+--------+                    +--------+

Graph Structure:
- Nodes: Entities (users, products, locations)
- Edges: Relationships between nodes
- Properties: Attributes on nodes and edges

Advantages over RDBMS for relationships:
- No expensive JOINs for connected data
- Traversing relationships is O(1) per hop
- Natural representation of complex relationships
```

**Examples: Neo4j, Amazon Neptune, ArangoDB**

```cypher
// Neo4j Cypher Query Language Example:

// Create nodes
CREATE (alice:Person {name: 'Alice', age: 30})
CREATE (bob:Person {name: 'Bob', age: 28})
CREATE (company:Company {name: 'TechCorp'})

// Create relationships
CREATE (alice)-[:KNOWS {since: 2020}]->(bob)
CREATE (alice)-[:WORKS_AT {role: 'Engineer'}]->(company)
CREATE (bob)-[:WORKS_AT {role: 'Designer'}]->(company)

// Find friends of friends
MATCH (person:Person {name: 'Alice'})-[:KNOWS]->(friend)-[:KNOWS]->(foaf)
WHERE person <> foaf
RETURN foaf.name AS FriendOfFriend

// Find shortest path between two people
MATCH path = shortestPath(
    (a:Person {name: 'Alice'})-[:KNOWS*]-(b:Person {name: 'Charlie'})
)
RETURN path

// Recommend connections (people who know my friends but I don't know)
MATCH (me:Person {name: 'Alice'})-[:KNOWS]->(friend)-[:KNOWS]->(suggestion)
WHERE NOT (me)-[:KNOWS]->(suggestion)
AND me <> suggestion
RETURN suggestion.name, COUNT(friend) AS mutual_friends
ORDER BY mutual_friends DESC
LIMIT 5

// Find all employees of a company
MATCH (p:Person)-[:WORKS_AT]->(c:Company {name: 'TechCorp'})
RETURN p.name, p.role
```

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs
- Network topology

---

### NoSQL Types Comparison

```
+------------------+----------------+------------------+----------------+
| Type             | Data Model     | Strengths        | Weaknesses     |
+------------------+----------------+------------------+----------------+
| Key-Value        | Key → Value    | Simple, fast     | No complex     |
|                  |                | Scalable         | queries        |
+------------------+----------------+------------------+----------------+
| Document         | JSON documents | Flexible schema  | Not ideal for  |
|                  |                | Rich queries     | relationships  |
+------------------+----------------+------------------+----------------+
| Column-Family    | Column groups  | Analytics        | Complex data   |
|                  |                | Write-heavy      | modeling       |
+------------------+----------------+------------------+----------------+
| Graph            | Nodes + Edges  | Relationships    | Not for simple |
|                  |                | Traversals       | key-value ops  |
+------------------+----------------+------------------+----------------+
```

---

## CAP Theorem (CAP 정리)

### Concept (개념)
The CAP theorem states that a distributed system can only guarantee two out of three properties: Consistency, Availability, and Partition Tolerance.

```
CAP Theorem Triangle:

                 Consistency (일관성)
                      /\
                     /  \
                    /    \
                   /  CA  \
                  /        \
                 /----------\
                /    CP  AP  \
               /              \
              /________________\
   Partition                    Availability
   Tolerance                    (가용성)
   (분할 내성)

- Consistency (C): All nodes see the same data at the same time
- Availability (A): Every request receives a response
- Partition Tolerance (P): System continues despite network failures
```

### The CAP Trade-offs

```
In distributed systems, network partitions WILL happen.
So you must choose between C and A when partitions occur.

Scenario: Network partition between Node A and Node B

+----------+         X         +----------+
| Node A   |----network-------|  Node B  |
| Data: v1 |     failure      |  Data: v1|
+----------+         X         +----------+

Client writes v2 to Node A. What happens?

CP System (Consistency + Partition Tolerance):
+----------+                   +----------+
| Node A   |                   | Node B   |
| Data: v2 | Write succeeds    | Data: v1 | Blocked! Cannot
+----------+                   +----------+ serve reads
- Node B becomes unavailable until partition heals
- All reads guaranteed to see v2 or fail
- Example: Traditional RDBMS, HBase, MongoDB (default)

AP System (Availability + Partition Tolerance):
+----------+                   +----------+
| Node A   |                   | Node B   |
| Data: v2 |                   | Data: v1 | Still serving!
+----------+                   +----------+ Returns stale v1
- Both nodes remain available
- But they may return different data
- Conflicts resolved later (eventual consistency)
- Example: Cassandra, DynamoDB, CouchDB
```

### CAP in Real Databases

```
+------------------+----------+--------------------------------+
| Database         | CAP      | Notes                          |
+------------------+----------+--------------------------------+
| PostgreSQL       | CA       | Single node, no partition      |
| MySQL Cluster    | CP       | Consistency over availability  |
| MongoDB          | CP/AP    | Configurable (read preference) |
| Cassandra        | AP       | Always available, eventual     |
| Redis Cluster    | CP       | Consistency prioritized        |
| DynamoDB         | AP       | Configurable consistency       |
| CockroachDB      | CP       | Distributed SQL                |
+------------------+----------+--------------------------------+
```

### PACELC Extension

```
PACELC: When there's Partition (P), choose Availability (A) or Consistency (C),
        Else (E), choose Latency (L) or Consistency (C)

        Network Partition?
              |
        +-----+-----+
        |           |
       YES          NO
        |           |
   Choose:      Choose:
   PA or PC     EL or EC

+------------------+--------+--------+
| Database         | P: A/C | E: L/C |
+------------------+--------+--------+
| Cassandra        | PA     | EL     | Fast reads, eventual consistency
| MongoDB          | PC     | EC     | Consistency focused
| DynamoDB         | PA     | EL     | Low latency prioritized
| CockroachDB      | PC     | EC     | Strong consistency
+------------------+--------+--------+
```

---

## RDBMS vs NoSQL Selection Criteria (RDBMS vs NoSQL 선택 기준)

### Decision Framework

```
+--------------------------------------------------+
| Choose RDBMS when:                               |
+--------------------------------------------------+
| [ ] Complex transactions with ACID guarantees    |
| [ ] Data has clear relationships (normalized)    |
| [ ] Complex queries with JOINs needed            |
| [ ] Data integrity is critical                   |
| [ ] Schema is well-defined and stable            |
| [ ] Moderate scale (millions of records)         |
| [ ] Need for ad-hoc queries and reporting        |
+--------------------------------------------------+

+--------------------------------------------------+
| Choose NoSQL when:                               |
+--------------------------------------------------+
| [ ] Massive scale (billions of records)          |
| [ ] High write throughput needed                 |
| [ ] Flexible/evolving schema                     |
| [ ] Simple queries (key-based access)            |
| [ ] Horizontal scaling is required               |
| [ ] High availability is critical                |
| [ ] Specific data model fits (graph, time-series)|
+--------------------------------------------------+
```

### Use Case Matching

```
+-------------------------+-------------+---------------------------+
| Use Case                | Best Choice | Reason                    |
+-------------------------+-------------+---------------------------+
| Banking transactions    | RDBMS       | ACID, data integrity      |
| E-commerce orders       | RDBMS       | Transactions, joins       |
| User sessions/cache     | Key-Value   | Fast, simple access       |
| Product catalog         | Document    | Flexible attributes       |
| Social network          | Graph       | Relationship traversal    |
| IoT sensor data         | Column      | Time-series, write-heavy  |
| Content management      | Document    | Flexible content types    |
| Real-time analytics     | Column      | Aggregations, time-series |
| Gaming leaderboards     | Key-Value   | Fast reads, sorted sets   |
| Fraud detection         | Graph       | Pattern detection         |
+-------------------------+-------------+---------------------------+
```

### Hybrid Approaches (Polyglot Persistence)

```
Modern Application Architecture:

+-------------------+
| E-Commerce App    |
+-------------------+
        |
        v
+-------------------+    +-------------------+    +-------------------+
| PostgreSQL        |    | MongoDB           |    | Redis             |
| - Orders          |    | - Product catalog |    | - Sessions        |
| - Payments        |    | - Reviews         |    | - Cart            |
| - Inventory       |    | - User profiles   |    | - Cache           |
+-------------------+    +-------------------+    +-------------------+
        |                        |                        |
        +------------------------+------------------------+
                                 |
                                 v
                    +-------------------+
                    | Elasticsearch     |
                    | - Product search  |
                    | - Full-text search|
                    +-------------------+

Each database handles what it's best at!
```

### Comparison Summary

```
+------------------+------------------+------------------+
| Aspect           | RDBMS            | NoSQL            |
+------------------+------------------+------------------+
| Data Model       | Relational       | Various          |
|                  | (tables, rows)   | (doc, KV, etc)   |
+------------------+------------------+------------------+
| Schema           | Fixed, predefined| Flexible, dynamic|
+------------------+------------------+------------------+
| Scaling          | Vertical (scale  | Horizontal (scale|
|                  | up)              | out)             |
+------------------+------------------+------------------+
| ACID             | Full support     | Varies (BASE)    |
+------------------+------------------+------------------+
| Transactions     | Complex multi-   | Limited (often   |
|                  | table            | single document) |
+------------------+------------------+------------------+
| Query Language   | SQL (standard)   | Varies by type   |
+------------------+------------------+------------------+
| JOINs            | Powerful, native | Limited or none  |
+------------------+------------------+------------------+
| Best For         | Complex queries, | Scale, speed,    |
|                  | transactions     | flexibility      |
+------------------+------------------+------------------+
```

### BASE vs ACID

```
ACID (Traditional RDBMS):
+-------------------+----------------------------------------+
| Atomicity         | All or nothing transactions            |
| Consistency       | Data always valid                      |
| Isolation         | Transactions don't interfere           |
| Durability        | Committed data survives failures       |
+-------------------+----------------------------------------+

BASE (Many NoSQL):
+-------------------+----------------------------------------+
| Basically Available| System always responds                |
| Soft state        | Data may change over time              |
| Eventually        | System becomes consistent eventually   |
| Consistent        |                                        |
+-------------------+----------------------------------------+

Timeline:
ACID: Write → Immediately consistent everywhere
BASE: Write → Node A consistent → ... → Node B eventually consistent

+-----------+    +-----------+    +-----------+
| Write to  | -> | Propagate | -> | Eventually|
| Node A    |    | to others |    | all sync  |
+-----------+    +-----------+    +-----------+
     t=0           t=10ms           t=100ms
```

---

## Summary (요약)

| Topic | Korean | Key Points |
|-------|--------|------------|
| NoSQL Emergence | NoSQL 등장 | Scale, flexibility, performance needs |
| Key-Value | 키-값 저장소 | Simple, fast, for caching/sessions |
| Document | 문서 저장소 | Flexible JSON, rich queries |
| Column-Family | 컬럼 패밀리 | Analytics, time-series, write-heavy |
| Graph | 그래프 DB | Relationships, social networks |
| CAP Theorem | CAP 정리 | Choose 2: Consistency, Availability, Partition Tolerance |
| Selection Criteria | 선택 기준 | Based on data model, scale, query needs |

### Quick Decision Guide

```
Need complex transactions? → RDBMS
Need massive scale? → NoSQL
Need flexible schema? → Document (MongoDB)
Need blazing fast cache? → Key-Value (Redis)
Need relationship queries? → Graph (Neo4j)
Need time-series analytics? → Column-Family (Cassandra)
Not sure? → Start with RDBMS, migrate specific use cases to NoSQL
```

---
