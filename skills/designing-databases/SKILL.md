---
name: designing-databases
description: "Design and review database schemas, queries, and data models for optimal performance and integrity. Use when users request: (1) Database schema design, (2) Data modeling, (3) Query optimization, (4) Index strategy, (5) Migration planning, or (6) Database technology selection. Applies normalization principles, indexing strategies, and database best practices for SQL and NoSQL databases."
---

# Database Design

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

## Data Types

For comprehensive guidance on choosing appropriate data types:
- **PostgreSQL & MongoDB**: See [references/data-types.md](references/data-types.md)
- **Database selection guide**: See [references/data-types.md](references/data-types.md)

## Indexing & Query Optimization

For detailed indexing strategies and query optimization:
- **Index types and strategies**: See [references/indexing.md](references/indexing.md)
- **Query optimization techniques**: See [references/indexing.md](references/indexing.md)
- **N+1 query prevention**: See [references/indexing.md](references/indexing.md)

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

For comprehensive migration strategies and patterns:
- **Safe migration techniques**: See [references/migrations.md](references/migrations.md)
- **Backward compatible changes**: See [references/migrations.md](references/migrations.md)
- **Batched updates**: See [references/migrations.md](references/migrations.md)
- **Rollback strategies**: See [references/migrations.md](references/migrations.md)

## Database Selection

For detailed comparison and selection guidance:
- See [references/data-types.md](references/data-types.md) for PostgreSQL, MySQL, MongoDB, and Redis comparison

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
