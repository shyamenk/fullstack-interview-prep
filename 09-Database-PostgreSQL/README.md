# PostgreSQL Database Mastery

## 🎯 PostgreSQL Fundamentals

### Core Concepts
- [ ] **ACID Properties**
  - Atomicity: All or nothing transactions
  - Consistency: Data integrity maintained
  - Isolation: Concurrent transaction handling
  - Durability: Permanent data persistence
- [ ] **Data Types**
  - Numeric: INTEGER, BIGINT, DECIMAL, NUMERIC
  - Text: VARCHAR, TEXT, CHAR
  - Date/Time: DATE, TIME, TIMESTAMP, INTERVAL
  - JSON: JSON, JSONB
  - Arrays: INTEGER[], TEXT[]
  - Custom types and enums

### SQL Query Mastery
- [ ] **Basic Queries**
  - SELECT with WHERE, ORDER BY, LIMIT
  - Aggregate functions (COUNT, SUM, AVG, MIN, MAX)
  - GROUP BY and HAVING clauses
  - DISTINCT and filtering
- [ ] **Advanced Queries**
  - JOIN operations (INNER, LEFT, RIGHT, FULL OUTER)
  - Subqueries and CTEs (Common Table Expressions)
  - Window functions and analytics
  - UNION, INTERSECT, EXCEPT operations

## 🚀 Advanced PostgreSQL Features

### Performance Optimization
- [ ] **Indexing Strategies**
  - B-tree indexes (default)
  - Hash indexes for equality
  - GIN indexes for full-text search
  - GiST indexes for geometric data
  - Partial and expression indexes
- [ ] **Query Optimization**
  - EXPLAIN and EXPLAIN ANALYZE
  - Query planning and execution
  - Index usage optimization
  - Query rewriting techniques

### JSON and NoSQL Features
- [ ] **JSON Support**
  - JSON vs JSONB comparison
  - JSON operators and functions
  - Indexing JSON data
  - JSON path expressions
- [ ] **Advanced Data Types**
  - Arrays and array operations
  - Range types and operations
  - Geometric types
  - Network address types

## 💻 Common Interview Questions

### Conceptual Questions
1. **What's the difference between JSON and JSONB in PostgreSQL?**
2. **Explain MVCC (Multi-Version Concurrency Control)**
3. **What are the different types of indexes in PostgreSQL?**
4. **How does PostgreSQL handle transactions and isolation levels?**
5. **What's the difference between DELETE, TRUNCATE, and DROP?**

### Performance Questions
1. **How would you optimize a slow-performing query?**
2. **Explain query execution plans and how to read them**
3. **What strategies would you use for database scaling?**
4. **How do you handle connection pooling in PostgreSQL?**

### Design Questions
1. **Design a schema for a social media application**
2. **How would you implement soft deletes?**
3. **Design a database for an e-commerce platform**
4. **Implement audit trails for data changes**

## 🛠️ Practical Examples

### Database Schema Design
```sql
-- Users table with proper constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    profile_data JSONB,

    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Posts table with foreign key relationships
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    tags TEXT[] DEFAULT '{}',
    published BOOLEAN DEFAULT false,
    view_count INTEGER DEFAULT 0,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT title_length CHECK (LENGTH(title) >= 1),
    CONSTRAINT content_length CHECK (LENGTH(content) >= 1)
);

-- Comments table with self-referencing foreign key
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    parent_id INTEGER REFERENCES comments(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT content_not_empty CHECK (LENGTH(TRIM(content)) > 0)
);

-- Create indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);

-- GIN index for array and JSONB operations
CREATE INDEX idx_posts_tags ON posts USING GIN(tags);
CREATE INDEX idx_posts_metadata ON posts USING GIN(metadata);
CREATE INDEX idx_users_profile_data ON users USING GIN(profile_data);
```

### Advanced Queries
```sql
-- Complex JOIN with aggregations
SELECT
    u.first_name,
    u.last_name,
    COUNT(p.id) as post_count,
    COUNT(c.id) as comment_count,
    AVG(p.view_count) as avg_views,
    MAX(p.created_at) as last_post_date
FROM users u
LEFT JOIN posts p ON u.id = p.user_id AND p.published = true
LEFT JOIN comments c ON u.id = c.user_id
WHERE u.is_active = true
GROUP BY u.id, u.first_name, u.last_name
HAVING COUNT(p.id) > 0
ORDER BY post_count DESC, avg_views DESC
LIMIT 10;

-- Window functions for analytics
SELECT
    title,
    view_count,
    created_at,
    ROW_NUMBER() OVER (ORDER BY view_count DESC) as popularity_rank,
    RANK() OVER (PARTITION BY DATE(created_at) ORDER BY view_count DESC) as daily_rank,
    LAG(view_count) OVER (ORDER BY created_at) as previous_post_views,
    AVG(view_count) OVER (
        ORDER BY created_at
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) as moving_avg_views
FROM posts
WHERE published = true
ORDER BY view_count DESC;

-- CTE for hierarchical data (comment threads)
WITH RECURSIVE comment_thread AS (
    -- Base case: top-level comments
    SELECT
        id,
        post_id,
        user_id,
        content,
        parent_id,
        created_at,
        0 as depth,
        id::TEXT as path
    FROM comments
    WHERE parent_id IS NULL AND post_id = $1

    UNION ALL

    -- Recursive case: child comments
    SELECT
        c.id,
        c.post_id,
        c.user_id,
        c.content,
        c.parent_id,
        c.created_at,
        ct.depth + 1,
        ct.path || '.' || c.id::TEXT
    FROM comments c
    JOIN comment_thread ct ON c.parent_id = ct.id
)
SELECT
    ct.*,
    u.first_name,
    u.last_name
FROM comment_thread ct
JOIN users u ON ct.user_id = u.id
ORDER BY path;

-- JSONB queries
SELECT
    id,
    title,
    profile_data->>'location' as user_location,
    profile_data->'preferences'->>'theme' as theme_preference
FROM posts p
JOIN users u ON p.user_id = u.id
WHERE profile_data ? 'location'
  AND profile_data->>'location' = 'New York'
  AND metadata @> '{"featured": true}';

-- Full-text search
SELECT
    id,
    title,
    content,
    ts_rank(
        to_tsvector('english', title || ' ' || content),
        plainto_tsquery('english', $1)
    ) as relevance_score
FROM posts
WHERE published = true
  AND to_tsvector('english', title || ' ' || content) @@ plainto_tsquery('english', $1)
ORDER BY relevance_score DESC;
```

### Performance Optimization
```sql
-- Analyze query performance
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM posts p
JOIN users u ON p.user_id = u.id
WHERE p.created_at >= CURRENT_DATE - INTERVAL '30 days'
  AND u.is_active = true
ORDER BY p.view_count DESC
LIMIT 20;

-- Create covering index
CREATE INDEX idx_posts_covering ON posts (created_at, user_id)
INCLUDE (title, view_count)
WHERE published = true;

-- Partial index for active users
CREATE INDEX idx_users_active_email ON users (email)
WHERE is_active = true;

-- Expression index for case-insensitive searches
CREATE INDEX idx_users_email_lower ON users (LOWER(email));

-- Analyze table statistics
ANALYZE posts;
ANALYZE users;

-- Update table statistics manually
UPDATE pg_stat_user_tables SET n_tup_ins = 0, n_tup_upd = 0, n_tup_del = 0;
```

## 🔧 Database Administration

### Backup and Recovery
```bash
# Full database backup
pg_dump -h localhost -U username -d database_name > backup.sql

# Compressed backup
pg_dump -h localhost -U username -d database_name | gzip > backup.sql.gz

# Custom format backup (recommended)
pg_dump -h localhost -U username -d database_name -Fc > backup.dump

# Schema-only backup
pg_dump -h localhost -U username -d database_name --schema-only > schema.sql

# Data-only backup
pg_dump -h localhost -U username -d database_name --data-only > data.sql

# Restore from backup
psql -h localhost -U username -d database_name < backup.sql
pg_restore -h localhost -U username -d database_name backup.dump

# Point-in-time recovery setup
# In postgresql.conf:
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /path/to/archive/%f'
```

### Monitoring and Maintenance
```sql
-- Monitor active connections
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    backend_start,
    state,
    query
FROM pg_stat_activity
WHERE state = 'active';

-- Find slow queries
SELECT
    query,
    mean_exec_time,
    calls,
    total_exec_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Table size information
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Index usage statistics
SELECT
    indexrelname,
    idx_tup_read,
    idx_tup_fetch,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Vacuum and analyze
VACUUM ANALYZE posts;
REINDEX INDEX idx_posts_user_id;

-- Auto-vacuum configuration
ALTER TABLE posts SET (autovacuum_vacuum_scale_factor = 0.1);
ALTER TABLE posts SET (autovacuum_analyze_scale_factor = 0.05);
```

## 🔒 Security Best Practices

### User Management and Permissions
```sql
-- Create database user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Create read-only user
CREATE USER readonly_user WITH PASSWORD 'readonly_password';

-- Grant specific permissions
GRANT CONNECT ON DATABASE myapp TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- Read-only permissions
GRANT CONNECT ON DATABASE myapp TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Row-level security
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_posts_policy ON posts
    FOR ALL TO app_user
    USING (user_id = current_setting('app.current_user_id')::INTEGER);

-- Function to set current user context
CREATE OR REPLACE FUNCTION set_current_user_id(user_id INTEGER)
RETURNS void AS $$
BEGIN
    PERFORM set_config('app.current_user_id', user_id::TEXT, false);
END;
$$ LANGUAGE plpgsql;
```

### Data Encryption and Auditing
```sql
-- Encrypt sensitive data
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Store encrypted passwords
INSERT INTO users (email, password_hash)
VALUES ('user@example.com', crypt('password123', gen_salt('bf', 8)));

-- Verify password
SELECT * FROM users
WHERE email = 'user@example.com'
  AND password_hash = crypt('password123', password_hash);

-- Audit trail table
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    operation VARCHAR(10) NOT NULL,
    user_id INTEGER,
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    ip_address INET
);

-- Audit trigger function
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, user_id, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, NEW.user_id, row_to_json(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, user_id, old_data, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, NEW.user_id, row_to_json(OLD), row_to_json(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, user_id, old_data)
        VALUES (TG_TABLE_NAME, TG_OP, OLD.user_id, row_to_json(OLD));
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Apply audit trigger
CREATE TRIGGER posts_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON posts
    FOR EACH ROW
    EXECUTE FUNCTION audit_trigger_function();
```

## 🧪 Testing Strategies

### Unit Testing with pgTAP
```sql
-- Install pgTAP extension
CREATE EXTENSION IF NOT EXISTS pgtap;

-- Test function
CREATE OR REPLACE FUNCTION test_user_creation()
RETURNS SETOF TEXT AS $$
BEGIN
    -- Test user insertion
    INSERT INTO users (email, password_hash, first_name, last_name)
    VALUES ('test@example.com', 'hashed_password', 'Test', 'User');

    -- Verify insertion
    RETURN NEXT ok(
        (SELECT COUNT(*) FROM users WHERE email = 'test@example.com') = 1,
        'User should be created successfully'
    );

    -- Test email uniqueness constraint
    RETURN NEXT throws_ok(
        'INSERT INTO users (email, password_hash, first_name, last_name)
         VALUES (''test@example.com'', ''another_hash'', ''Another'', ''User'')',
        'Users email should be unique'
    );

    -- Cleanup
    DELETE FROM users WHERE email = 'test@example.com';
END;
$$ LANGUAGE plpgsql;

-- Run tests
SELECT * FROM test_user_creation();
```

## 📚 Learning Resources

### Official Documentation
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)

### Recommended Books
- "PostgreSQL: Up and Running" by Regina Obe and Leo Hsu
- "Mastering PostgreSQL" by Hans-Jürgen Schönig
- "PostgreSQL High Performance" by Gregory Smith

### Online Courses
- "PostgreSQL Bootcamp" by Jose Portilla
- "Advanced PostgreSQL" by Pluralsight
- "Database Design and PostgreSQL" by freeCodeCamp

## ✅ Progress Tracker

### Core Concepts
- [ ] SQL fundamentals mastered
- [ ] Data types and constraints understood
- [ ] JOIN operations practiced
- [ ] Aggregate functions learned

### Advanced Features
- [ ] Index types and optimization strategies applied
- [ ] JSON/JSONB operations practiced
- [ ] Window functions and CTEs mastered
- [ ] Full-text search implemented

### Performance & Administration
- [ ] Query optimization techniques learned
- [ ] Database monitoring set up
- [ ] Backup and recovery procedures practiced
- [ ] Security measures implemented

### Practical Experience
- [ ] 3+ database schemas designed
- [ ] Performance tuning performed
- [ ] Complex queries written
- [ ] Production database management experience

**Target**: Master PostgreSQL for efficient data storage, complex querying, and production database management