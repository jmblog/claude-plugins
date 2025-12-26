# Migration Best Practices

## Safe Migrations

### Avoid Locking Operations

```sql
-- ✅ Good: Add column with default (no lock)
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';

-- ✅ Good: Add index concurrently (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- ❌ Bad: Locks table during operation
ALTER TABLE users ADD COLUMN status VARCHAR(20) NOT NULL;
CREATE INDEX idx_users_email ON users(email);
```

**Key principles:**
- Use `DEFAULT` when adding NOT NULL columns
- Use `CONCURRENTLY` for index creation (PostgreSQL)
- Break large migrations into smaller steps
- Avoid blocking operations during peak hours

### Backward Compatible Changes

Multi-step approach for breaking changes:

```sql
-- Step 1: Add new column (nullable)
ALTER TABLE users ADD COLUMN new_email VARCHAR(255);

-- Step 2: Backfill data (in batches for large tables)
UPDATE users SET new_email = email WHERE new_email IS NULL LIMIT 1000;
-- Repeat until all rows updated

-- Step 3: Make it required
ALTER TABLE users ALTER COLUMN new_email SET NOT NULL;

-- Step 4: Add constraint
ALTER TABLE users ADD CONSTRAINT unique_new_email UNIQUE (new_email);

-- Step 5: Drop old column (after code deploy)
ALTER TABLE users DROP COLUMN email;

-- Step 6: Rename column (optional)
ALTER TABLE users RENAME COLUMN new_email TO email;
```

**Deployment strategy:**
1. Deploy migration (Step 1-4)
2. Deploy application code that uses both columns
3. Monitor for issues
4. Deploy migration (Step 5-6)
5. Deploy application code that uses only new column

### Batched Updates

For large tables, update in batches:

```sql
-- ❌ Bad: Updates entire table at once (locks for long time)
UPDATE users SET status = 'active' WHERE status IS NULL;

-- ✅ Good: Batch updates
DO $$
DECLARE
  batch_size INT := 1000;
  rows_affected INT;
BEGIN
  LOOP
    UPDATE users SET status = 'active'
    WHERE id IN (
      SELECT id FROM users
      WHERE status IS NULL
      LIMIT batch_size
    );

    GET DIAGNOSTICS rows_affected = ROW_COUNT;
    EXIT WHEN rows_affected = 0;

    -- Small delay between batches
    PERFORM pg_sleep(0.1);
  END LOOP;
END $$;
```

## Migration Patterns

### Adding a Required Column

```sql
-- Step 1: Add nullable column with default
ALTER TABLE products ADD COLUMN category VARCHAR(50) DEFAULT 'uncategorized';

-- Step 2: Backfill existing data (if needed)
UPDATE products SET category = 'electronics' WHERE type = 'gadget';

-- Step 3: Remove default and make NOT NULL
ALTER TABLE products ALTER COLUMN category DROP DEFAULT;
ALTER TABLE products ALTER COLUMN category SET NOT NULL;
```

### Renaming a Column

```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Copy data
UPDATE users SET full_name = name;

-- Step 3: Update application to write to both columns

-- Step 4: Drop old column (after code deployment)
ALTER TABLE users DROP COLUMN name;
```

### Changing Column Type

```sql
-- Step 1: Add new column with new type
ALTER TABLE orders ADD COLUMN price_cents INTEGER;

-- Step 2: Backfill data
UPDATE orders SET price_cents = (price * 100)::INTEGER;

-- Step 3: Make NOT NULL
ALTER TABLE orders ALTER COLUMN price_cents SET NOT NULL;

-- Step 4: Drop old column (after code deployment)
ALTER TABLE orders DROP COLUMN price;

-- Step 5: Rename new column
ALTER TABLE orders RENAME COLUMN price_cents TO price;
```

### Splitting a Table

```sql
-- Step 1: Create new table
CREATE TABLE user_profiles (
  user_id UUID PRIMARY KEY REFERENCES users(id),
  bio TEXT,
  avatar_url VARCHAR(500),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Step 2: Migrate data
INSERT INTO user_profiles (user_id, bio, avatar_url)
SELECT id, bio, avatar_url FROM users;

-- Step 3: Update application to read from both tables

-- Step 4: Drop columns from original table (after code deployment)
ALTER TABLE users DROP COLUMN bio;
ALTER TABLE users DROP COLUMN avatar_url;
```

## Rollback Strategy

### Always Plan for Rollback

```sql
-- Migration up
CREATE TABLE new_feature (
  id UUID PRIMARY KEY,
  data JSONB
);

-- Migration down (rollback)
DROP TABLE IF EXISTS new_feature;
```

### Non-reversible Migrations

Some migrations cannot be automatically rolled back:
- Data deletion
- Column drops with data
- Complex data transformations

**Solution:** Manual rollback procedures and backups

```sql
-- Before dropping column, backup data
CREATE TABLE users_email_backup AS
SELECT id, email FROM users;

-- Now safe to drop
ALTER TABLE users DROP COLUMN email;

-- Rollback procedure (manual):
-- ALTER TABLE users ADD COLUMN email VARCHAR(255);
-- UPDATE users u SET email = b.email FROM users_email_backup b WHERE u.id = b.id;
```

## Testing Migrations

### Test on Production-like Data

1. Create snapshot of production database
2. Run migration on snapshot
3. Measure execution time
4. Check for locking issues
5. Verify data integrity
6. Test application compatibility

### Measure Impact

```sql
-- Check table size before migration
SELECT pg_size_pretty(pg_total_relation_size('users'));

-- Time the migration
\timing on
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'active';

-- Verify no data loss
SELECT COUNT(*) FROM users;
```

## Common Pitfalls

**Adding NOT NULL without DEFAULT:**
- Causes full table lock
- Fails if any NULL values exist
- Use DEFAULT or multi-step approach

**Creating indexes on large tables:**
- Locks table during creation
- Use `CONCURRENTLY` option
- Consider creating during low-traffic periods

**Renaming columns without code changes:**
- Application breaks immediately
- Use multi-step approach with both columns
- Gradual migration strategy

**Large data migrations:**
- Can timeout
- Locks tables for extended periods
- Use batched updates

**Dropping columns with data:**
- Irreversible without backup
- Always backup first
- Consider soft-delete pattern
