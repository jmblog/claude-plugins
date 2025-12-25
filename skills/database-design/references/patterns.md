# Database Design Patterns and Examples

This reference provides detailed database design patterns, real-world examples, and anti-patterns.

## Table of Contents

1. [Schema Design Patterns](#schema-design-patterns)
2. [Relationship Patterns](#relationship-patterns)
3. [Indexing Patterns](#indexing-patterns)
4. [Query Patterns](#query-patterns)
5. [Partitioning Patterns](#partitioning-patterns)
6. [NoSQL Patterns](#nosql-patterns)
7. [Migration Patterns](#migration-patterns)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Schema Design Patterns

### Basic Entity Pattern

**Pattern**: Well-normalized entity with proper constraints

```sql
-- ✅ Good: Complete entity design
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'inactive', 'suspended')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status) WHERE status = 'active';

-- ❌ Bad: Poor design
CREATE TABLE users (
  id SERIAL PRIMARY KEY,  -- Sequential, predictable
  email TEXT,             -- Too broad, no constraint
  name TEXT,              -- Combined first/last name
  password TEXT,          -- Not hashed
  status INT,             -- Magic numbers instead of enum
  created_at TIMESTAMP    -- No timezone
);
```

### Audit Trail Pattern

**Pattern**: Track all changes to records

```sql
-- Audit log table
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  table_name VARCHAR(100) NOT NULL,
  record_id UUID NOT NULL,
  action VARCHAR(20) NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
  old_data JSONB,
  new_data JSONB,
  changed_by UUID REFERENCES users(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_table_record ON audit_log(table_name, record_id);
CREATE INDEX idx_audit_changed_at ON audit_log(changed_at);

-- Trigger to populate audit log
CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'DELETE' THEN
    INSERT INTO audit_log (table_name, record_id, action, old_data, changed_by)
    VALUES (TG_TABLE_NAME, OLD.id, TG_OP, row_to_json(OLD), current_setting('app.user_id', true)::UUID);
    RETURN OLD;
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO audit_log (table_name, record_id, action, old_data, new_data, changed_by)
    VALUES (TG_TABLE_NAME, NEW.id, TG_OP, row_to_json(OLD), row_to_json(NEW), current_setting('app.user_id', true)::UUID);
    RETURN NEW;
  ELSIF TG_OP = 'INSERT' THEN
    INSERT INTO audit_log (table_name, record_id, action, new_data, changed_by)
    VALUES (TG_TABLE_NAME, NEW.id, TG_OP, row_to_json(NEW), current_setting('app.user_id', true)::UUID);
    RETURN NEW;
  END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger_func();
```

### Soft Delete Pattern

**Pattern**: Mark records as deleted instead of removing

```sql
-- ✅ Good: Soft delete with index
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  content TEXT NOT NULL,
  author_id UUID NOT NULL REFERENCES users(id),
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index only active records
CREATE INDEX idx_posts_active ON posts(created_at) WHERE deleted_at IS NULL;

-- Query active records
SELECT * FROM posts WHERE deleted_at IS NULL;

-- Soft delete
UPDATE posts SET deleted_at = NOW() WHERE id = '...';

-- Restore
UPDATE posts SET deleted_at = NULL WHERE id = '...';
```

---

## Relationship Patterns

### One-to-Many Pattern

**Pattern**: Standard foreign key relationship

```sql
CREATE TABLE authors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL
);

CREATE TABLE books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  author_id UUID NOT NULL REFERENCES authors(id) ON DELETE CASCADE,
  published_at DATE
);

CREATE INDEX idx_books_author ON books(author_id);

-- Query
SELECT a.name, COUNT(b.id) as book_count
FROM authors a
LEFT JOIN books b ON b.author_id = a.id
GROUP BY a.id, a.name;
```

### Many-to-Many Pattern

**Pattern**: Junction table for N:M relationships

```sql
CREATE TABLE students (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL
);

CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL
);

-- Junction table
CREATE TABLE student_courses (
  student_id UUID NOT NULL REFERENCES students(id) ON DELETE CASCADE,
  course_id UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  enrolled_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  grade VARCHAR(2),
  PRIMARY KEY (student_id, course_id)
);

CREATE INDEX idx_student_courses_course ON student_courses(course_id);

-- Query: Students in a course
SELECT s.name
FROM students s
JOIN student_courses sc ON sc.student_id = s.id
WHERE sc.course_id = '...';

-- Query: Courses for a student
SELECT c.name, sc.grade
FROM courses c
JOIN student_courses sc ON sc.course_id = c.id
WHERE sc.student_id = '...';
```

### Polymorphic Association Pattern

**Pattern**: Relate entity to multiple types

```sql
-- ✅ Good: Separate junction tables (type-safe)
CREATE TABLE comments_on_posts (
  comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  PRIMARY KEY (comment_id, post_id)
);

CREATE TABLE comments_on_videos (
  comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,
  video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
  PRIMARY KEY (comment_id, video_id)
);

-- ❌ Bad: Polymorphic with type column (not type-safe)
CREATE TABLE comments (
  id UUID PRIMARY KEY,
  commentable_type VARCHAR(50),  -- 'Post', 'Video'
  commentable_id UUID,           -- No foreign key constraint
  content TEXT
);
```

### Self-Referential Pattern

**Pattern**: Tree structure (parent-child)

```sql
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  parent_id UUID REFERENCES categories(id) ON DELETE CASCADE,
  path TEXT,  -- Materialized path: '/electronics/computers/laptops'
  level INT NOT NULL DEFAULT 0
);

CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE INDEX idx_categories_path ON categories USING gin(to_tsvector('simple', path));

-- Get all children (using recursive CTE)
WITH RECURSIVE category_tree AS (
  SELECT id, name, parent_id, 0 as depth
  FROM categories
  WHERE id = '...'  -- Start category

  UNION ALL

  SELECT c.id, c.name, c.parent_id, ct.depth + 1
  FROM categories c
  JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;
```

---

## Indexing Patterns

### Composite Index Pattern

**Pattern**: Multi-column index for common queries

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),
  status VARCHAR(20) NOT NULL,
  total NUMERIC(10, 2) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Composite index for common query
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Query uses index efficiently
SELECT * FROM orders
WHERE user_id = '...' AND status = 'pending'
ORDER BY created_at DESC;

-- Include frequently accessed columns
CREATE INDEX idx_orders_user_status_total ON orders(user_id, status)
INCLUDE (total, created_at);
```

### Partial Index Pattern

**Pattern**: Index subset of rows

```sql
-- Index only active orders
CREATE INDEX idx_orders_active ON orders(user_id, created_at)
WHERE status IN ('pending', 'processing');

-- Index only recent records
CREATE INDEX idx_orders_recent ON orders(created_at)
WHERE created_at > NOW() - INTERVAL '30 days';

-- Index only non-null values
CREATE INDEX idx_users_phone ON users(phone)
WHERE phone IS NOT NULL;
```

### Expression Index Pattern

**Pattern**: Index computed values

```sql
-- Index lowercase email for case-insensitive search
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

SELECT * FROM users WHERE LOWER(email) = LOWER('User@Example.com');

-- Index extracted date part
CREATE INDEX idx_orders_year_month ON orders(
  EXTRACT(YEAR FROM created_at),
  EXTRACT(MONTH FROM created_at)
);
```

### Full-Text Search Pattern

**Pattern**: Efficient text search

```sql
-- Add tsvector column
ALTER TABLE posts ADD COLUMN search_vector tsvector;

-- Create GIN index
CREATE INDEX idx_posts_search ON posts USING gin(search_vector);

-- Update trigger to maintain search vector
CREATE OR REPLACE FUNCTION posts_search_trigger() RETURNS trigger AS $$
BEGIN
  NEW.search_vector :=
    setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.content, '')), 'B');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER posts_search_update
BEFORE INSERT OR UPDATE ON posts
FOR EACH ROW EXECUTE FUNCTION posts_search_trigger();

-- Search query
SELECT id, title, ts_rank(search_vector, query) as rank
FROM posts, to_tsquery('english', 'database & design') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

---

## Query Patterns

### Pagination Pattern

**Pattern**: Efficient pagination

```sql
-- ❌ Bad: OFFSET pagination (slow for large offsets)
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 1000;  -- Skips 1000 rows

-- ✅ Good: Cursor-based pagination
SELECT * FROM posts
WHERE created_at < '2024-01-15 10:00:00'  -- Last seen timestamp
ORDER BY created_at DESC
LIMIT 20;

-- Even better: Composite cursor (id + timestamp)
SELECT * FROM posts
WHERE (created_at, id) < ('2024-01-15 10:00:00', 'last-uuid')
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

### Batch Processing Pattern

**Pattern**: Process records in batches

```sql
-- Process in batches to avoid long transactions
DO $$
DECLARE
  batch_size INT := 1000;
  processed INT := 0;
BEGIN
  LOOP
    WITH batch AS (
      SELECT id FROM old_table
      WHERE processed_at IS NULL
      ORDER BY id
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    )
    UPDATE old_table
    SET processed_at = NOW()
    FROM batch
    WHERE old_table.id = batch.id;

    GET DIAGNOSTICS processed = ROW_COUNT;
    EXIT WHEN processed = 0;

    COMMIT;  -- Commit each batch
    PERFORM pg_sleep(0.1);  -- Brief pause
  END LOOP;
END $$;
```

### Upsert Pattern

**Pattern**: Insert or update if exists

```sql
-- PostgreSQL: ON CONFLICT
INSERT INTO user_stats (user_id, login_count, last_login)
VALUES ('...', 1, NOW())
ON CONFLICT (user_id) DO UPDATE
SET login_count = user_stats.login_count + 1,
    last_login = EXCLUDED.last_login;

-- MySQL: ON DUPLICATE KEY UPDATE
INSERT INTO user_stats (user_id, login_count, last_login)
VALUES ('...', 1, NOW())
ON DUPLICATE KEY UPDATE
  login_count = login_count + 1,
  last_login = VALUES(last_login);
```

### Window Function Pattern

**Pattern**: Advanced analytics queries

```sql
-- Rank users by order total
SELECT
  user_id,
  total,
  RANK() OVER (ORDER BY total DESC) as rank,
  PERCENT_RANK() OVER (ORDER BY total DESC) as percentile
FROM (
  SELECT user_id, SUM(total) as total
  FROM orders
  GROUP BY user_id
) user_totals;

-- Running total
SELECT
  created_at,
  amount,
  SUM(amount) OVER (ORDER BY created_at) as running_total
FROM transactions
ORDER BY created_at;

-- Moving average
SELECT
  date,
  revenue,
  AVG(revenue) OVER (
    ORDER BY date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) as weekly_avg
FROM daily_revenue
ORDER BY date;
```

---

## Partitioning Patterns

### Time-Based Partitioning

**Pattern**: Partition by date range

```sql
-- Create partitioned table
CREATE TABLE events (
  id UUID NOT NULL,
  event_type VARCHAR(50) NOT NULL,
  user_id UUID,
  data JSONB,
  created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

-- Create partitions
CREATE TABLE events_2024_01 PARTITION OF events
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE events_2024_02 PARTITION OF events
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE events_2024_03 PARTITION OF events
  FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Create indexes on each partition
CREATE INDEX idx_events_2024_01_user ON events_2024_01(user_id);
CREATE INDEX idx_events_2024_02_user ON events_2024_02(user_id);

-- Automatic partition creation script
CREATE OR REPLACE FUNCTION create_monthly_partition(
  base_table TEXT,
  partition_date DATE
)
RETURNS VOID AS $$
DECLARE
  partition_name TEXT;
  start_date DATE;
  end_date DATE;
BEGIN
  partition_name := base_table || '_' || to_char(partition_date, 'YYYY_MM');
  start_date := date_trunc('month', partition_date);
  end_date := start_date + INTERVAL '1 month';

  EXECUTE format(
    'CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
     FOR VALUES FROM (%L) TO (%L)',
    partition_name, base_table, start_date, end_date
  );

  EXECUTE format(
    'CREATE INDEX IF NOT EXISTS idx_%s_created ON %I(created_at)',
    partition_name, partition_name
  );
END;
$$ LANGUAGE plpgsql;
```

### Archival Pattern

**Pattern**: Move old data to archive

```sql
-- Current data table (hot)
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  total NUMERIC(10, 2),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Archive table (cold storage)
CREATE TABLE orders_archive (
  LIKE orders INCLUDING ALL
);

-- Archive old records
WITH archived AS (
  DELETE FROM orders
  WHERE created_at < NOW() - INTERVAL '1 year'
  RETURNING *
)
INSERT INTO orders_archive
SELECT * FROM archived;
```

---

## NoSQL Patterns

### MongoDB Document Pattern

**Pattern**: Embedded vs referenced documents

```typescript
// ✅ Good: Embed for 1:few relationship
interface User {
  _id: ObjectId
  email: string
  profile: {
    firstName: string
    lastName: string
    avatar: string
  }
  addresses: Array<{
    type: 'home' | 'work'
    street: string
    city: string
  }>
}

// ✅ Good: Reference for 1:many relationship
interface Post {
  _id: ObjectId
  title: string
  content: string
  authorId: ObjectId  // Reference to User
  comments: Array<{
    authorId: ObjectId
    text: string
    createdAt: Date
  }>  // Embed recent comments
}

// ❌ Bad: Unbounded array growth
interface BadPost {
  _id: ObjectId
  title: string
  comments: Comment[]  // Could grow infinitely!
}
```

### Redis Caching Pattern

**Pattern**: Cache-aside with Redis

```typescript
async function getUser(userId: string): Promise<User> {
  const cacheKey = `user:${userId}`

  // Try cache
  const cached = await redis.get(cacheKey)
  if (cached) {
    return JSON.parse(cached)
  }

  // Cache miss - fetch from DB
  const user = await db.users.findOne({ id: userId })
  if (!user) {
    throw new NotFoundException()
  }

  // Update cache
  await redis.setex(cacheKey, 3600, JSON.stringify(user))

  return user
}

// Invalidate cache on update
async function updateUser(userId: string, data: Partial<User>) {
  const user = await db.users.update({ id: userId }, data)

  // Invalidate cache
  await redis.del(`user:${userId}`)

  return user
}
```

---

## Migration Patterns

### Backward Compatible Change

**Pattern**: Multi-step migration

```sql
-- Step 1: Add new column (nullable)
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Backfill data (can run while app is running)
UPDATE users
SET full_name = first_name || ' ' || last_name
WHERE full_name IS NULL;

-- Step 3: Make it NOT NULL (after backfill completes)
ALTER TABLE users ALTER COLUMN full_name SET NOT NULL;

-- Step 4: Drop old columns (after code deploy)
ALTER TABLE users DROP COLUMN first_name;
ALTER TABLE users DROP COLUMN last_name;
```

### Zero-Downtime Index Creation

**Pattern**: Concurrent index creation

```sql
-- ✅ Good: CREATE INDEX CONCURRENTLY (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- ❌ Bad: Locks table during creation
CREATE INDEX idx_users_email ON users(email);
```

### Data Type Migration

**Pattern**: Safe data type change

```sql
-- Can't directly change: VARCHAR to INTEGER
-- Multi-step approach:

-- Step 1: Add new column
ALTER TABLE products ADD COLUMN price_cents INTEGER;

-- Step 2: Backfill
UPDATE products SET price_cents = (price * 100)::INTEGER;

-- Step 3: Make NOT NULL
ALTER TABLE products ALTER COLUMN price_cents SET NOT NULL;

-- Step 4: Add constraint
ALTER TABLE products ADD CONSTRAINT check_price_positive
CHECK (price_cents >= 0);

-- Step 5: Drop old column (after code deploy)
ALTER TABLE products DROP COLUMN price;

-- Step 6: Rename new column
ALTER TABLE products RENAME COLUMN price_cents TO price;
```

---

## Anti-Patterns to Avoid

### EAV (Entity-Attribute-Value)

```sql
-- ❌ Bad: EAV anti-pattern
CREATE TABLE entity_attributes (
  entity_id UUID,
  attribute_name VARCHAR(100),
  attribute_value TEXT
);
-- Problems: No type safety, complex queries, poor performance

-- ✅ Good: Proper schema with JSONB for truly dynamic data
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price NUMERIC(10, 2) NOT NULL,
  category VARCHAR(100) NOT NULL,
  specifications JSONB  -- For category-specific attributes
);

CREATE INDEX idx_products_specs ON products USING gin(specifications);
```

### God Table

```sql
-- ❌ Bad: Everything in one table
CREATE TABLE users (
  id UUID PRIMARY KEY,
  -- User info
  email VARCHAR(255),
  name VARCHAR(255),
  -- Profile
  bio TEXT,
  avatar TEXT,
  -- Settings
  theme VARCHAR(20),
  notifications_enabled BOOLEAN,
  -- Orders
  last_order_id UUID,
  total_spent NUMERIC,
  -- ... 50 more columns
);

-- ✅ Good: Normalized tables
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL
);

CREATE TABLE user_profiles (
  user_id UUID PRIMARY KEY REFERENCES users(id),
  bio TEXT,
  avatar TEXT
);

CREATE TABLE user_settings (
  user_id UUID PRIMARY KEY REFERENCES users(id),
  theme VARCHAR(20),
  notifications_enabled BOOLEAN
);
```

### CHAR for Variable Length

```sql
-- ❌ Bad: CHAR pads with spaces
CREATE TABLE users (
  email CHAR(255)  -- Wastes space
);

-- ✅ Good: VARCHAR for variable length
CREATE TABLE users (
  email VARCHAR(255)
);
```

### Implicit Type Conversions

```sql
-- ❌ Bad: String ID compared to integer
SELECT * FROM users WHERE id = '123';  -- If id is INTEGER

-- ✅ Good: Correct type
SELECT * FROM users WHERE id = 123;

-- ❌ Bad: Leading wildcards prevent index usage
SELECT * FROM users WHERE email LIKE '%@example.com';

-- ✅ Good: Trailing wildcards can use index
SELECT * FROM users WHERE email LIKE 'user%';
```

### Missing Indexes on Foreign Keys

```sql
-- ❌ Bad: No index on foreign key
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id)  -- No index!
);

-- ✅ Good: Index foreign keys
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id)
);
CREATE INDEX idx_orders_user ON orders(user_id);
```

### SELECT * in Application Code

```typescript
// ❌ Bad: Fetches all columns
const users = await db.query('SELECT * FROM users')

// ✅ Good: Select specific columns
const users = await db.query(
  'SELECT id, email, name FROM users'
)
```

### GUID/UUID as VARCHAR

```sql
-- ❌ Bad: UUID as string
CREATE TABLE users (
  id VARCHAR(36) PRIMARY KEY  -- '123e4567-e89b-12d3-a456-426614174000'
);

-- ✅ Good: Use UUID type
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid()
);
```
