# Data Types Best Practices

## PostgreSQL

### Appropriate Data Types

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

### Key Principles

**UUID vs SERIAL:**
- Use UUID for distributed systems and security
- SERIAL is predictable and may leak information
- UUID provides better horizontal scaling

**Numeric Types:**
- Use INTEGER for whole numbers
- Use NUMERIC for exact decimal values (money)
- Avoid FLOAT/REAL for financial data (precision issues)

**Text Types:**
- VARCHAR(n) for limited-length strings
- TEXT for unlimited length
- Avoid CHAR(n) unless fixed-width needed

**Date/Time:**
- Always use TIMESTAMPTZ (with timezone)
- Avoid TIMESTAMP without timezone
- Store in UTC, convert for display

**JSON:**
- Use JSONB (binary) over JSON (text)
- JSONB supports indexing and faster queries
- Good for flexible/evolving schemas

## MongoDB

### Well-Structured Documents

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

### Key Principles

**Embedded vs Referenced:**
- Embed when data is always accessed together
- Reference for data shared across documents
- Consider document size limits (16MB)

**Array Fields:**
- Use embedded documents in arrays
- Avoid parallel arrays (hard to maintain consistency)
- Index array fields when queried

**Date Fields:**
- Use Date type, not strings
- Store in UTC
- Use ISODate format

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
