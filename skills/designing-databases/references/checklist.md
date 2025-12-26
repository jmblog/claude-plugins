# Database Design Checklist

This checklist provides a systematic framework for designing and reviewing database schemas.

## How to Use This Checklist

1. **Review each category** relevant to your database type
2. **Rate each item** (✅ Good, ⚠️ Needs improvement, ❌ Missing/Poor, N/A Not applicable)
3. **Note specific issues** with table/collection names
4. **Prioritize improvements** based on data integrity and performance impact
5. **Reference patterns.md** for implementation examples

---

## 1. Schema Design & Data Modeling

### Entity Design
- [ ] **Clear entities**: Are entities well-defined and aligned with domain?
- [ ] **Naming conventions**: Are table/column names consistent and descriptive?
- [ ] **Data types**: Are appropriate data types used for each column?
- [ ] **Primary keys**: Does every table have a primary key?
- [ ] **UUIDs vs integers**: Is ID strategy appropriate (UUID for distributed, INT for single DB)?

### Relationships
- [ ] **Foreign keys**: Are foreign key relationships defined?
- [ ] **Relationship types**: Are 1:1, 1:N, N:M relationships properly modeled?
- [ ] **Junction tables**: Are many-to-many relationships using junction tables?
- [ ] **Cascading deletes**: Are ON DELETE/UPDATE rules appropriate?
- [ ] **Orphan prevention**: Are orphaned records prevented?

### Normalization
- [ ] **1NF compliance**: Are all values atomic (no arrays as strings)?
- [ ] **2NF compliance**: Are there no partial dependencies?
- [ ] **3NF compliance**: Are there no transitive dependencies?
- [ ] **Denormalization**: Is denormalization justified and documented?
- [ ] **Redundancy**: Is intentional redundancy for performance documented?

**Impact**: Critical - Foundation of data integrity
**Priority**: Must be correct before production

---

## 2. Data Types & Constraints

### Data Type Selection
- [ ] **Appropriate types**: Are data types appropriate for each field?
- [ ] **String lengths**: Are VARCHAR lengths appropriate (not too large)?
- [ ] **Numeric types**: Are INT/BIGINT/DECIMAL used correctly?
- [ ] **Date/time types**: Are TIMESTAMP/DATE types with timezone?
- [ ] **Boolean types**: Are BOOLEAN used instead of TINYINT/CHAR?
- [ ] **JSON types**: Is JSONB (PostgreSQL) or JSON used for semi-structured data?

### Constraints
- [ ] **NOT NULL**: Are required fields marked NOT NULL?
- [ ] **UNIQUE**: Are unique constraints defined where needed?
- [ ] **CHECK**: Are CHECK constraints used for validation?
- [ ] **DEFAULT**: Are DEFAULT values defined appropriately?
- [ ] **Foreign key constraints**: Are FK constraints enforced?

### Data Validation
- [ ] **Application level**: Is validation also in application code?
- [ ] **Database level**: Are critical validations in database?
- [ ] **Enum types**: Are limited value sets using ENUM or CHECK?
- [ ] **Range checks**: Are numeric ranges validated (age >= 0)?

**Impact**: High - Prevents data corruption
**Priority**: Critical for data integrity

---

## 3. Indexing Strategy

### Index Coverage
- [ ] **Primary keys**: Are all primary keys indexed (automatic)?
- [ ] **Foreign keys**: Are foreign keys indexed?
- [ ] **Query patterns**: Are indexes aligned with query patterns?
- [ ] **Unique constraints**: Are UNIQUE constraints indexed?
- [ ] **Composite indexes**: Are multi-column queries using composite indexes?

### Index Types
- [ ] **B-tree indexes**: Are B-tree indexes used for equality/range queries?
- [ ] **Hash indexes**: Are hash indexes used for equality only (if supported)?
- [ ] **GIN/GiST**: Are specialized indexes used (PostgreSQL JSONB, arrays)?
- [ ] **Full-text**: Is full-text search using proper indexes?
- [ ] **Partial indexes**: Are partial indexes used for subset queries?

### Index Optimization
- [ ] **Index order**: Are composite index column orders optimized?
- [ ] **Covering indexes**: Are frequently accessed columns included?
- [ ] **Index size**: Are index sizes monitored?
- [ ] **Unused indexes**: Are unused indexes identified and removed?
- [ ] **Duplicate indexes**: Are duplicate indexes avoided?

### Index Maintenance
- [ ] **REINDEX**: Is index maintenance scheduled (PostgreSQL)?
- [ ] **ANALYZE**: Are statistics updated regularly?
- [ ] **Bloat monitoring**: Is index bloat monitored?

**Impact**: Critical - Affects query performance
**Priority**: Essential for acceptable performance

---

## 4. Query Optimization

### Query Patterns
- [ ] **N+1 prevention**: Are N+1 queries prevented?
- [ ] **Batch queries**: Are batched queries used where possible?
- [ ] **JOINs**: Are JOINs used instead of multiple queries?
- [ ] **Subqueries**: Are correlated subqueries avoided?
- [ ] **SELECT * avoided**: Are specific columns selected?

### Query Performance
- [ ] **EXPLAIN usage**: Are slow queries analyzed with EXPLAIN?
- [ ] **Index usage**: Are indexes being used by queries?
- [ ] **Full table scans**: Are full table scans avoided?
- [ ] **Query limits**: Are LIMIT clauses used for large result sets?
- [ ] **Pagination**: Is efficient pagination implemented (cursor vs offset)?

### ORM Optimization
- [ ] **Eager loading**: Is eager loading used to prevent N+1?
- [ ] **Select specific**: Are specific fields selected, not all?
- [ ] **Raw queries**: Are complex queries using raw SQL?
- [ ] **Query logging**: Is query logging enabled in development?

**Impact**: High - Affects application performance
**Priority**: Address slow queries immediately

---

## 5. Scalability & Performance

### Read Scalability
- [ ] **Read replicas**: Are read replicas configured?
- [ ] **Caching layer**: Is caching used (Redis, Memcached)?
- [ ] **Connection pooling**: Is connection pooling configured?
- [ ] **Query caching**: Is query result caching used?
- [ ] **CDN**: Are static queries cached at edge?

### Write Scalability
- [ ] **Batch writes**: Are bulk inserts batched?
- [ ] **Async writes**: Are non-critical writes async?
- [ ] **Write optimization**: Are writes optimized (fewer indexes)?
- [ ] **Partitioning**: Is table partitioning used for large tables?
- [ ] **Sharding**: Is sharding strategy defined (if needed)?

### Partitioning Strategy
- [ ] **Partition type**: Is partition type appropriate (range, list, hash)?
- [ ] **Partition key**: Is partition key chosen well?
- [ ] **Partition pruning**: Are queries using partition pruning?
- [ ] **Partition maintenance**: Is partition management automated?

### Connection Management
- [ ] **Connection limits**: Are connection limits configured?
- [ ] **Pooling**: Is connection pooling used?
- [ ] **Timeout**: Are connection timeouts set?
- [ ] **Reuse**: Are connections properly reused?

**Impact**: High - Affects scalability and costs
**Priority**: Plan before scaling issues occur

---

## 6. Transaction Management

### Transaction Design
- [ ] **ACID compliance**: Are ACID properties needed?
- [ ] **Transaction scope**: Are transactions as short as possible?
- [ ] **Isolation level**: Is isolation level appropriate?
- [ ] **Deadlock prevention**: Are deadlocks prevented?
- [ ] **Retry logic**: Is retry logic implemented for conflicts?

### Concurrency Control
- [ ] **Optimistic locking**: Is optimistic locking used where appropriate?
- [ ] **Pessimistic locking**: Is pessimistic locking used carefully?
- [ ] **Row-level locking**: Are row-level locks used over table locks?
- [ ] **Lock timeout**: Are lock timeouts configured?

### Consistency
- [ ] **Strong consistency**: Is strong consistency required?
- [ ] **Eventual consistency**: Is eventual consistency acceptable?
- [ ] **Consistency checks**: Are consistency checks in place?

**Impact**: High - Affects data integrity
**Priority**: Critical for financial/critical data

---

## 7. Security

### Access Control
- [ ] **Principle of least privilege**: Do users have minimal permissions?
- [ ] **Role-based access**: Are roles used instead of per-user permissions?
- [ ] **Application user**: Is there a dedicated app database user?
- [ ] **Admin separation**: Are admin credentials separate?
- [ ] **Connection security**: Are connections encrypted (SSL/TLS)?

### Data Protection
- [ ] **Encryption at rest**: Is sensitive data encrypted?
- [ ] **Encryption in transit**: Is SSL/TLS enforced?
- [ ] **Column encryption**: Are sensitive columns encrypted?
- [ ] **Password hashing**: Are passwords hashed (bcrypt, not MD5)?
- [ ] **PII identification**: Is PII identified and protected?

### SQL Injection Prevention
- [ ] **Parameterized queries**: Are parameterized queries used?
- [ ] **ORM usage**: Is ORM used for query building?
- [ ] **Input validation**: Is input validated before queries?
- [ ] **Stored procedures**: Are stored procedures used safely?

### Audit & Compliance
- [ ] **Audit logs**: Are sensitive operations logged?
- [ ] **Access logs**: Are database accesses logged?
- [ ] **Data retention**: Is retention policy defined?
- [ ] **GDPR compliance**: Is GDPR compliance addressed?
- [ ] **Backup encryption**: Are backups encrypted?

**Impact**: Critical - Security breaches are catastrophic
**Priority**: Must be addressed immediately

---

## 8. Backup & Recovery

### Backup Strategy
- [ ] **Regular backups**: Are backups automated and regular?
- [ ] **Backup testing**: Are restore processes tested?
- [ ] **Point-in-time recovery**: Is PITR available?
- [ ] **Backup retention**: Is retention policy defined?
- [ ] **Off-site backups**: Are backups stored off-site?

### Disaster Recovery
- [ ] **Recovery plan**: Is DR plan documented and tested?
- [ ] **RTO/RPO**: Are RTO and RPO defined?
- [ ] **Failover**: Is automatic failover configured?
- [ ] **Geo-replication**: Is geo-replication needed?

### Data Integrity
- [ ] **Backup verification**: Are backups verified?
- [ ] **Corruption detection**: Is corruption monitored?
- [ ] **Consistency checks**: Are consistency checks automated?

**Impact**: Critical - Data loss is unacceptable
**Priority**: Must be in place before production

---

## 9. Monitoring & Maintenance

### Performance Monitoring
- [ ] **Query performance**: Are slow queries monitored?
- [ ] **Connection metrics**: Are connection counts monitored?
- [ ] **Cache hit rates**: Are cache metrics tracked?
- [ ] **Disk usage**: Is disk space monitored?
- [ ] **Replication lag**: Is replication lag monitored?

### Health Checks
- [ ] **Database health**: Are health checks automated?
- [ ] **Alerting**: Are alerts configured for critical issues?
- [ ] **Deadlocks**: Are deadlocks monitored?
- [ ] **Long queries**: Are long-running queries detected?

### Maintenance Tasks
- [ ] **VACUUM**: Is VACUUM automated (PostgreSQL)?
- [ ] **ANALYZE**: Are statistics updated?
- [ ] **Index rebuild**: Is index maintenance scheduled?
- [ ] **Log rotation**: Are logs rotated?
- [ ] **Cleanup jobs**: Are old data cleanup jobs scheduled?

**Impact**: Medium - Prevents degradation
**Priority**: Implement monitoring from start

---

## 10. Migration Strategy

### Migration Planning
- [ ] **Version control**: Are migrations version controlled?
- [ ] **Reversible**: Are migrations reversible (down migrations)?
- [ ] **Testing**: Are migrations tested in staging?
- [ ] **Zero-downtime**: Are migrations designed for zero downtime?
- [ ] **Data backfill**: Are data backfills planned?

### Migration Safety
- [ ] **Backward compatible**: Are changes backward compatible?
- [ ] **Locking**: Are table locks avoided?
- [ ] **Concurrent indexes**: Are indexes created concurrently?
- [ ] **Batch updates**: Are large updates batched?
- [ ] **Rollback plan**: Is rollback plan defined?

### Migration Tools
- [ ] **Migration framework**: Is migration tool used (Knex, Alembic, Flyway)?
- [ ] **Automated deployment**: Are migrations automated in CI/CD?
- [ ] **Migration history**: Is migration history tracked?

**Impact**: High - Failed migrations cause downtime
**Priority**: Essential for production deployments

---

## 11. Documentation

### Schema Documentation
- [ ] **ER diagrams**: Are ER diagrams up to date?
- [ ] **Table descriptions**: Are tables documented?
- [ ] **Column comments**: Are complex columns documented?
- [ ] **Relationship docs**: Are relationships documented?
- [ ] **Index rationale**: Are index purposes documented?

### Query Documentation
- [ ] **Complex queries**: Are complex queries documented?
- [ ] **Performance notes**: Are performance considerations noted?
- [ ] **Query examples**: Are query examples provided?

### Operational Docs
- [ ] **Backup procedures**: Is backup process documented?
- [ ] **Recovery procedures**: Is recovery process documented?
- [ ] **Maintenance tasks**: Are maintenance tasks documented?
- [ ] **Troubleshooting**: Is troubleshooting guide available?

**Impact**: Medium - Helps team collaboration
**Priority**: Maintain alongside schema

---

## 12. Database-Specific Considerations

### PostgreSQL
- [ ] **Extensions**: Are needed extensions installed (uuid-ossp, pgcrypto)?
- [ ] **Sequences**: Are sequences used correctly?
- [ ] **JSONB**: Is JSONB used over JSON?
- [ ] **Array types**: Are array types used appropriately?
- [ ] **Generated columns**: Are generated columns used?

### MongoDB
- [ ] **Document structure**: Are documents well-structured?
- [ ] **Embedding vs referencing**: Is embedding/referencing strategy clear?
- [ ] **Schema validation**: Is schema validation defined?
- [ ] **Indexes**: Are compound indexes on frequently queried fields?
- [ ] **Aggregation pipeline**: Are aggregation pipelines optimized?

### MySQL
- [ ] **Engine type**: Is InnoDB used (not MyISAM)?
- [ ] **Character set**: Is utf8mb4 used?
- [ ] **Row format**: Is row format appropriate?

**Impact**: Medium - Database-specific optimizations
**Priority**: Review for chosen database

---

## Scoring and Prioritization

### Issue Severity

**Critical (Fix immediately):**
- No primary keys
- Missing foreign key constraints
- No backups configured
- SQL injection vulnerabilities
- No encryption for sensitive data
- Data integrity violations

**High (Fix before production):**
- Missing indexes on frequently queried columns
- Poor normalization causing data anomalies
- No migration strategy
- Missing NOT NULL constraints
- Inefficient queries causing performance issues

**Medium (Address soon):**
- Suboptimal data types
- Missing documentation
- No monitoring
- Unused indexes
- Missing CHECK constraints

**Low (Nice to have):**
- Better naming conventions
- Additional covering indexes
- Optimization opportunities
- More comprehensive documentation

### Review Process

1. **Schema review** (30 min): Tables, relationships, constraints
2. **Index review** (20 min): Index coverage and optimization
3. **Query review** (30 min): Analyze slow queries and patterns
4. **Security review** (20 min): Access control, encryption
5. **Scalability review** (15 min): Partitioning, replication strategy

**Total time**: ~2 hours for comprehensive database review
