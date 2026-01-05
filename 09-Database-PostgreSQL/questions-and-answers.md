# PostgreSQL Interview Questions & Answers

## Table of Contents

1. [What is Row-Level Security (RLS) in PostgreSQL?](#q1-what-is-row-level-security-rls-in-postgresql)

---

## Q1. What is Row-Level Security (RLS) in PostgreSQL?

### Concept:

Row-Level Security (RLS) restricts which rows a user can see or modify based on policies — **without changing application queries**.

- Enforces data access at the **database level**, not application code
- Prevents data leakage in multi-tenant systems
- Commonly used in SaaS, fintech, and healthcare apps

### How to Implement RLS:

**1. Create a table:**

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT,
  item TEXT,
  amount NUMERIC
);
```

**2. Enable RLS:**

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
```

**3. Create a policy:**

```sql
CREATE POLICY user_orders_policy
ON orders
USING (user_id = current_setting('app.user_id')::INT);
```

**4. Set user context from application:**

```sql
SET app.user_id = '42';
SELECT * FROM orders;  -- Returns only rows where user_id = 42
```

### Write Access Control:

```sql
CREATE POLICY user_orders_modify
ON orders
FOR ALL
USING (user_id = current_setting('app.user_id')::INT)
WITH CHECK (user_id = current_setting('app.user_id')::INT);
```

- `USING` → Which rows are accessible (SELECT, UPDATE, DELETE)
- `WITH CHECK` → What rows can be inserted/updated

### Important Interview Detail:

Superusers can bypass RLS unless you force it:

```sql
ALTER TABLE orders FORCE ROW LEVEL SECURITY;
```

### Analogy:

Think of an **apartment building**:

- **Without RLS** → Everyone has a master key and can enter any apartment
- **With RLS** → Each tenant's key only opens their own apartment, enforced by the lock system itself (not just a "please don't enter" sign)

### ⚡ One-liner to remember:
> RLS = Database-level row filtering that users cannot bypass, even with SELECT *.

---
