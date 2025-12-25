---
name: database-design
description: "Design and review database schemas, queries, and data models for optimal performance and integrity. Use when users request: (1) Database schema design, (2) Data modeling, (3) Query optimization, (4) Index strategy, (5) Migration planning, or (6) Database technology selection. Applies normalization principles, indexing strategies, and database best practices for SQL and NoSQL databases."
---

# Database Design

## Overview

This skill provides comprehensive guidance for designing, reviewing, and optimizing database schemas and data models. It covers relational (SQL) and non-relational (NoSQL) databases, focusing on data integrity, performance, scalability, and best practices.

## Core Design Workflow

When designing or reviewing a database:

1. **Understand Requirements**
   - Identify data entities and relationships
   - Understand access patterns and query requirements
   - Determine data volume and growth projections
   - Consider consistency vs availability requirements
   - Review performance and latency requirements

2. **Design Data Model**
   - Define entities, attributes, and relationships
   - Apply normalization (or denormalization)
   - Design primary and foreign keys
   - Plan data types and constraints
   - Consider indexing strategy

3. **Optimize for Performance**
   - Identify query patterns
   - Design appropriate indexes
   - Plan caching strategy
   - Consider partitioning/sharding
   - Optimize for read vs write patterns

4. **Review and Validate**
   - Check for data integrity constraints
   - Validate referential integrity
   - Assess query performance
   - Plan migration strategy
   - Document design decisions

## Quick Reference: Database Principles

### Normalization Forms

**1NF (First Normal Form):**
- Atomic values (no repeating groups)
- Each column contains single value
- Unique row identifier

**2NF (Second Normal Form):**
- Must be in 1NF
- No partial dependencies
- Non-key attributes depend on entire primary key

**3NF (Third Normal Form):**
- Must be in 2NF
- No transitive dependencies
- Non-key attributes depend only on primary key

**BCNF (Boyce-Codd Normal Form):**
- Stricter version of 3NF
- Every determinant is a candidate key

### When to Denormalize

- Read-heavy workloads
- Frequently joined tables
- Performance optimization needed
- Data warehouse / analytics
- Caching layer

### ACID Properties (SQL)

- **Atomicity**: All or nothing transactions
- **Consistency**: Data integrity maintained
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data persists

### CAP Theorem (Distributed Systems)

Can only achieve 2 of 3:
- **Consistency**: All nodes see same data
- **Availability**: Every request gets response
- **Partition Tolerance**: System works despite network issues

Most databases choose: CP (SQL) or AP (NoSQL)

## Common Use Cases

### Relational Database Design

For traditional RDBMS:
- Entity-relationship modeling
- Normalization and denormalization
- Foreign key relationships
- Transaction design
- Index optimization

### NoSQL Database Design

For document/key-value stores:
- Document structure design
- Embedded vs referenced documents
- Denormalization strategies
- Query pattern optimization
- Eventual consistency handling

### Data Warehouse Design

For analytics databases:
- Star schema design
- Snowflake schema
- Fact and dimension tables
- Aggregate tables
- Columnar storage

### Time-Series Database

For metrics and events:
- Time-based partitioning
- Retention policies
- Downsampling strategies
- Tag/label design

## Design Checklist

Use `references/checklist.md` for comprehensive database review covering:
- Schema design and normalization
- Data types and constraints
- Indexing strategy
- Query optimization
- Scalability planning
- Migration strategy

## Resources

### references/patterns.md

Comprehensive guide to database patterns:
- Schema design patterns
- Indexing patterns
- Partitioning strategies
- Caching patterns
- Migration patterns
- Real-world examples and trade-offs

### references/checklist.md

Structured evaluation framework:
- Schema quality assessment
- Performance optimization checklist
- Security considerations
- Scalability review

## Data Types Best Practices

### PostgreSQL

```sql
-- ✅ Good: Appropriate data types
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  age INTEGER CHECK (age >= 0 AND age <= 150),
  balance NUMERIC(10, 2) NOT NULL DEFAULT 0,
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ❌ Bad: Wrong data types
CREATE TABLE users (
  id SERIAL PRIMARY KEY,           -- Sequential, predictable
  email TEXT,                      -- Too broad
  age VARCHAR(3),                  -- Should be numeric
  balance FLOAT,                   -- Precision issues
  metadata TEXT,                   -- Should be JSONB
  created_at TIMESTAMP             -- No timezone
);
```

### MongoDB

```typescript
// ✅ Good: Well-structured document
interface User {
  _id: ObjectId
  email: string
  profile: {
    firstName: string
    lastName: string
    age: number
  }
  addresses: Array<{
    type: 'home' | 'work'
    street: string
    city: string
    zipCode: string
  }>
  createdAt: Date
  updatedAt: Date
}

// ❌ Bad: Flat structure with arrays of primitives
interface BadUser {
  _id: ObjectId
  email: string
  firstName: string
  lastName: string
  addressTypes: string[]      // Separate from addresses
  streets: string[]
  cities: string[]
}
```

## Indexing Strategies

### Single Column Index

```sql
-- For queries filtering by email
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'user@example.com';
```

### Composite Index

```sql
-- For queries filtering by multiple columns
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

SELECT * FROM orders
WHERE user_id = '123' AND status = 'pending';
```

### Partial Index

```sql
-- Index only active users
CREATE INDEX idx_active_users ON users(email)
WHERE status = 'active';
```

### Covering Index

```sql
-- Include all columns needed by query
CREATE INDEX idx_users_email_name ON users(email)
INCLUDE (first_name, last_name);

-- Query can be satisfied by index alone
SELECT first_name, last_name FROM users
WHERE email = 'user@example.com';
```

### Full-Text Search Index

```sql
-- PostgreSQL full-text search
CREATE INDEX idx_posts_search ON posts
USING GIN (to_tsvector('english', title || ' ' || content));

SELECT * FROM posts
WHERE to_tsvector('english', title || ' ' || content)
      @@ to_tsquery('english', 'database & design');
```

## Query Optimization

### Explain Plans

```sql
-- Analyze query performance
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name;
```

### Avoid N+1 Queries

```typescript
// ❌ Bad: N+1 query problem
const users = await db.users.findAll()
for (const user of users) {
  user.orders = await db.orders.find({ userId: user.id }) // N queries
}

// ✅ Good: Join or batch load
const users = await db.users.findAll({
  include: [{ model: db.orders }]
})

// ✅ Good: DataLoader pattern
const orderLoader = new DataLoader(async (userIds) => {
  const orders = await db.orders.find({ userId: { $in: userIds } })
  return userIds.map(id => orders.filter(o => o.userId === id))
})
```

### Use Proper JOINs

```sql
-- ✅ Good: Use appropriate JOIN type
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON o.user_id = u.id
WHERE o.status = 'completed';

-- ❌ Bad: Cartesian product then filter
SELECT u.name, o.total
FROM users u, orders o
WHERE u.id = o.user_id AND o.status = 'completed';
```

## Partitioning Strategies

### Range Partitioning

```sql
-- Partition by date range
CREATE TABLE orders (
  id UUID,
  created_at TIMESTAMPTZ,
  total NUMERIC
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_01 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### List Partitioning

```sql
-- Partition by category
CREATE TABLE products (
  id UUID,
  category VARCHAR(50),
  name VARCHAR(255)
) PARTITION BY LIST (category);

CREATE TABLE products_electronics PARTITION OF products
  FOR VALUES IN ('electronics', 'computers');

CREATE TABLE products_clothing PARTITION OF products
  FOR VALUES IN ('clothing', 'accessories');
```

### Hash Partitioning

```sql
-- Distribute data evenly
CREATE TABLE events (
  id UUID,
  user_id UUID,
  event_type VARCHAR(50)
) PARTITION BY HASH (user_id);

CREATE TABLE events_0 PARTITION OF events
  FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE events_1 PARTITION OF events
  FOR VALUES WITH (MODULUS 4, REMAINDER 1);
```

## Migration Best Practices

### Safe Migrations

```sql
-- ✅ Good: Add column with default (no lock)
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';

-- ✅ Good: Add index concurrently (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- ❌ Bad: Locks table during operation
ALTER TABLE users ADD COLUMN status VARCHAR(20) NOT NULL;
CREATE INDEX idx_users_email ON users(email);
```

### Backward Compatible Changes

```sql
-- Step 1: Add new column (nullable)
ALTER TABLE users ADD COLUMN new_email VARCHAR(255);

-- Step 2: Backfill data
UPDATE users SET new_email = email;

-- Step 3: Make it required
ALTER TABLE users ALTER COLUMN new_email SET NOT NULL;

-- Step 4: Add constraint
ALTER TABLE users ADD CONSTRAINT unique_new_email UNIQUE (new_email);

-- Step 5: Drop old column (after code deploy)
ALTER TABLE users DROP COLUMN email;
```

## Database Selection Guide

### PostgreSQL

**Use when:**
- Complex queries and JOINs
- ACID transactions required
- Strong consistency needed
- JSON document support needed
- GIS data (PostGIS)
- Full-text search

**Pros:**
- Feature-rich, JSONB support
- Strong consistency
- Excellent performance
- Extensible

**Cons:**
- Vertical scaling limits
- Complex replication setup

### MySQL

**Use when:**
- Read-heavy workloads
- Simple queries
- Wide hosting support needed
- WordPress/PHP ecosystem

**Pros:**
- Fast reads
- Wide adoption
- Good replication

**Cons:**
- Less feature-rich than PostgreSQL
- InnoDB limitations

### MongoDB

**Use when:**
- Flexible schema needed
- Rapid iteration
- Horizontal scaling required
- Document-oriented data

**Pros:**
- Flexible schema
- Easy horizontal scaling
- Good for prototyping

**Cons:**
- Eventual consistency
- Memory intensive
- Complex aggregations harder

### Redis

**Use when:**
- Caching needed
- Session storage
- Real-time leaderboards
- Pub/sub messaging
- Rate limiting

**Pros:**
- Extremely fast
- Rich data structures
- Persistence options

**Cons:**
- Memory-only (mostly)
- Single-threaded
- Not for primary storage

## Security Best Practices

### Access Control

```sql
-- Create read-only user
CREATE USER readonly_user WITH PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Create app user with limited permissions
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON users, orders TO app_user;
```

### Row-Level Security

```sql
-- PostgreSQL RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_documents ON documents
  FOR ALL
  TO app_user
  USING (user_id = current_setting('app.user_id')::UUID);
```

### Encryption

```sql
-- Encrypt sensitive columns
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Store encrypted data
INSERT INTO users (email, ssn)
VALUES ('user@example.com', pgp_sym_encrypt('123-45-6789', 'encryption_key'));

-- Retrieve decrypted data
SELECT email, pgp_sym_decrypt(ssn, 'encryption_key') as ssn
FROM users;
```

## Anti-Patterns to Avoid

### EAV (Entity-Attribute-Value)

```sql
-- ❌ Bad: EAV anti-pattern
CREATE TABLE entity_attributes (
  entity_id UUID,
  attribute_name VARCHAR(100),
  attribute_value TEXT
);

-- ✅ Good: Proper schema with JSONB for flexible attributes
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price NUMERIC(10, 2) NOT NULL,
  attributes JSONB  -- For truly dynamic attributes
);
```

### Storing Arrays as Strings

```sql
-- ❌ Bad: Comma-separated values
CREATE TABLE users (
  tags VARCHAR(255)  -- 'tag1,tag2,tag3'
);

-- ✅ Good: Use array type or junction table
CREATE TABLE users (
  tags TEXT[]  -- PostgreSQL array
);

-- Or junction table for many-to-many
CREATE TABLE user_tags (
  user_id UUID REFERENCES users(id),
  tag_id UUID REFERENCES tags(id),
  PRIMARY KEY (user_id, tag_id)
);
```

### Using NULL for Multiple Meanings

```sql
-- ❌ Bad: NULL means both "unknown" and "not applicable"
status VARCHAR(20) NULL  -- NULL could mean many things

-- ✅ Good: Explicit values
status VARCHAR(20) NOT NULL DEFAULT 'pending'
  CHECK (status IN ('pending', 'active', 'inactive', 'unknown'))
```

## Ethical Considerations

When designing databases:
- **Data Minimization**: Only store necessary data
- **Privacy by Design**: Plan for data privacy from the start
- **Right to Deletion**: Design for data deletion/anonymization
- **Encryption**: Encrypt sensitive data
- **Audit Trails**: Log access to sensitive data
- **Retention Policies**: Define and implement data retention
- **Compliance**: Consider GDPR, CCPA, HIPAA requirements
