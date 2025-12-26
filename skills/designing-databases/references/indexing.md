# Indexing Strategies

## Index Types

### Single Column Index

```sql
-- For queries filtering by email
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'user@example.com';
```

**Use when:**
- Single column frequently queried
- High selectivity (unique or near-unique values)
- Simple equality or range queries

### Composite Index

```sql
-- For queries filtering by multiple columns
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

SELECT * FROM orders
WHERE user_id = '123' AND status = 'pending';
```

**Key principles:**
- Column order matters (most selective first)
- Index can be used for prefixes (user_id alone)
- Cannot use index starting from second column

### Partial Index

```sql
-- Index only active users
CREATE INDEX idx_active_users ON users(email)
WHERE status = 'active';
```

**Benefits:**
- Smaller index size
- Faster index scans
- Lower maintenance cost
- Perfect for filtered queries

### Covering Index

```sql
-- Include all columns needed by query
CREATE INDEX idx_users_email_name ON users(email)
INCLUDE (first_name, last_name);

-- Query can be satisfied by index alone
SELECT first_name, last_name FROM users
WHERE email = 'user@example.com';
```

**Benefits:**
- No table lookup needed (index-only scan)
- Significant performance improvement
- Larger index size (trade-off)

### Full-Text Search Index

```sql
-- PostgreSQL full-text search
CREATE INDEX idx_posts_search ON posts
USING GIN (to_tsvector('english', title || ' ' || content));

SELECT * FROM posts
WHERE to_tsvector('english', title || ' ' || content)
      @@ to_tsquery('english', 'database & design');
```

**Use when:**
- Text search across multiple columns
- Natural language queries
- Alternative to external search engines

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

**Key metrics to watch:**
- Seq Scan → add index
- High cost → optimize query
- Nested Loop with large datasets → check join strategy
- Execution time vs planning time

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

## Index Best Practices

**When to add indexes:**
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Columns in GROUP BY
- Foreign key columns

**When NOT to add indexes:**
- Small tables (< 1000 rows)
- Columns with low selectivity (boolean, status with few values)
- Frequently updated columns (index maintenance cost)
- Wide columns (large overhead)

**Index maintenance:**
- Monitor index usage (pg_stat_user_indexes)
- Remove unused indexes
- Rebuild fragmented indexes
- Update statistics regularly (ANALYZE)

## Common Issues

**Over-indexing:**
- Slows down INSERT/UPDATE/DELETE
- Increases storage requirements
- Higher maintenance cost
- Monitor and remove unused indexes

**Under-indexing:**
- Slow queries (sequential scans)
- High CPU usage
- Poor user experience
- Add indexes based on query patterns

**Wrong column order in composite index:**
- Index not used by queries
- Create index with most selective column first
- Consider query patterns when ordering
