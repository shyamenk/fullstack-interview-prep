# Database Design Interview Questions & Answers

## Table of Contents

### Database Fundamentals
1. [What is a transaction in a database?](#q1-what-is-a-transaction-in-a-database)
2. [What is atomicity in a database?](#q2-what-is-atomicity-in-a-database)
3. [What is throughput in databases?](#q3-what-is-throughput-in-databases)
4. [What are constraints in a database?](#q4-what-are-constraints-in-a-database)
5. [What is integrity in a database?](#q5-what-is-integrity-in-a-database)

### Database Design Decisions
6. [What are the core differences between SQL and NoSQL?](#q6-what-are-the-core-differences-between-sql-and-nosql)
7. [When should you use SQL vs NoSQL?](#q7-when-should-you-use-sql-vs-nosql)

### Database Operations
8. [How do you handle database migrations in production?](#q8-how-do-you-handle-database-migrations-in-production)

---

## Q1. What is a transaction in a database?

### Concept:

A transaction is a **group of database operations** that must all succeed or all fail together.

### Example:

```sql
BEGIN;
INSERT INTO orders (id, item) VALUES (1, 'phone');
UPDATE inventory SET stock = stock - 1 WHERE item = 'phone';
COMMIT;
```

If any step fails, all changes are rolled back.

### Why Transactions Matter:

| Without Transaction | With Transaction |
|---------------------|------------------|
| Order inserted, inventory update fails → inconsistent data | Both succeed or both fail → data stays consistent |

### Analogy:

Think of **ordering food online**:
- Charge card + reserve table + send confirmation
- If card declines, don't reserve table or send confirmation
- All steps must succeed together

### ⚡ One-liner to remember:
> Transaction = All operations succeed together or fail together.

---

## Q2. What is atomicity in a database?

### Concept:

Atomicity means a transaction is **all-or-nothing** — no partial completion.

### Example:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT; -- Both updates happen, or neither does
```

### Failed Transaction:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1; -- Success
UPDATE accounts SET balance = balance + 500 WHERE id = 999; -- Fails (no user)
ROLLBACK; -- First update is undone
```

Money is never partially transferred.

### Analogy:

Think of a **light switch**:
- Light is either ON or OFF
- Never stuck halfway
- Transaction either completes fully or reverts fully

### ⚡ One-liner to remember:
> Atomicity = All or nothing, no partial states.

---

## Q3. What is throughput in databases?

### Concept:

Throughput is the **number of transactions** a database can handle per second.

### Measurement:

- **Reads per second:** 10,000 queries/sec
- **Writes per second:** 5,000 inserts/sec
- **Transactions per second (TPS):** Combined metric

### Example:

```typescript
// High throughput scenario
for (let i = 0; i < 10000; i++) {
  await db.insert({ event: "click", timestamp: Date.now() });
}
// Database handles 5,000 inserts/sec = ~2 seconds total
```

### Why It Matters:

| Low Throughput | High Throughput |
|----------------|-----------------|
| Requests queue up, users wait | Handles peak traffic smoothly |
| Database bottleneck | Scales with load |

### Improving Throughput:

- Add indexes for faster reads
- Use connection pooling
- Horizontal scaling (sharding)
- Cache frequently accessed data

### Analogy:

Think of a **highway**:
- Throughput = cars per hour that can pass through
- More lanes = higher throughput
- Traffic jam = low throughput

### ⚡ One-liner to remember:
> Throughput = How many operations the database handles per second.

---

## Q4. What are constraints in a database?

### Concept:

Constraints are **rules** that restrict what data can be stored, ensuring data quality.

### Common Constraints:

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,           -- Unique identifier
  email TEXT UNIQUE,            -- No duplicate emails
  age INT CHECK (age >= 18),    -- Must be 18 or older
  name TEXT NOT NULL,           -- Cannot be empty
  country_id INT REFERENCES countries(id) -- Must match existing country
);
```

### Constraint Types:

| Constraint    | Purpose                          | Example                  |
|---------------|----------------------------------|--------------------------|
| PRIMARY KEY   | Unique identifier                | `id INT PRIMARY KEY`     |
| UNIQUE        | No duplicates allowed            | `email TEXT UNIQUE`      |
| NOT NULL      | Value required                   | `name TEXT NOT NULL`     |
| CHECK         | Custom validation rule           | `CHECK (age >= 18)`      |
| FOREIGN KEY   | References another table         | `REFERENCES users(id)`   |

### What Happens When Violated:

```sql
INSERT INTO users (id, email, age, name) VALUES (1, 'test@test.com', 15, 'John');
-- ERROR: Check constraint "age >= 18" violated
```

### Analogy:

Think of **airport security rules**:
- No liquids over 100ml (CHECK constraint)
- Must have valid passport (NOT NULL)
- Passport number must be unique (UNIQUE)
- Flight must exist in system (FOREIGN KEY)

### ⚡ One-liner to remember:
> Constraints = Database rules that prevent invalid data from being stored.

---

## Q5. What is integrity in a database?

### Concept:

Integrity ensures data is **accurate, consistent, and reliable** through relationships and constraints.

### Types of Integrity:

**1. Entity Integrity:**
Every table must have a primary key.

```sql
CREATE TABLE products (
  id INT PRIMARY KEY,  -- Ensures each product is unique
  name TEXT
);
```

**2. Referential Integrity:**
Foreign keys must reference valid data.

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY,
  user_id INT REFERENCES users(id)  -- Order must have valid user
);

-- This fails if user 999 doesn't exist:
INSERT INTO orders (id, user_id) VALUES (1, 999);
```

**3. Domain Integrity:**
Data must be of correct type and within valid range.

```sql
CREATE TABLE employees (
  salary DECIMAL(10,2) CHECK (salary >= 0)  -- No negative salaries
);
```

### Why Integrity Matters:

| Without Integrity | With Integrity |
|-------------------|----------------|
| Orphaned orders without users | Every order linked to valid user |
| Negative balances in accounts | Balances stay within valid range |
| Duplicate primary keys | Each record uniquely identifiable |

### Analogy:

Think of a **building foundation**:
- Integrity = Structural strength
- Without it, the building (data) collapses
- Constraints are the reinforcements that maintain integrity

### ⚡ One-liner to remember:
> Integrity = Data accuracy through constraints and relationships.

---

## Q6. What are the core differences between SQL and NoSQL?

### Concept:

SQL and NoSQL solve different problems — the choice depends on data structure, consistency needs, and scale.

### Key Differences:

| Aspect       | SQL (Relational)          | NoSQL (Non-Relational)       |
|--------------|---------------------------|------------------------------|
| Data model   | Tables, rows, columns     | Key-Value, Document, Graph   |
| Schema       | Fixed schema              | Flexible / schema-less       |
| Transactions | Strong ACID               | Eventual consistency         |
| Scaling      | Vertical (mostly)         | Horizontal (native)          |
| Query        | SQL language              | API / Query DSL              |
| Joins        | Native support            | Usually avoided              |

### Read/Write Characteristics:

**SQL (PostgreSQL, MySQL):**

```typescript
// Writes: Slower but guarantees integrity
BEGIN;
INSERT INTO orders (user_id, amount) VALUES (1, 99.99);
UPDATE inventory SET stock = stock - 1 WHERE product_id = 10;
COMMIT;

// Reads: Efficient for complex queries
SELECT u.name, COUNT(o.id) as order_count
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```

- **Writes:** Slower (transactions, constraints, indexes)
- **Reads:** Very efficient for complex queries and joins
- **Best for:** Data correctness over raw speed

**NoSQL (MongoDB, DynamoDB):**

```typescript
// Writes: Very fast
db.sessions.insertOne({
  userId: 42,
  token: "abc123",
  expiresAt: new Date()
});

// Reads: Fast for simple access
db.sessions.findOne({ userId: 42 });
```

- **Writes:** Very fast, high throughput
- **Reads:** Fast for simple, predictable access
- **Best for:** Scale and performance over strict consistency

### Analogy:

Think of a **library**:

- **SQL** → Traditional catalog (strict organization, find related books easily, slower to add new categories)
- **NoSQL** → Digital bookmarks (quick to add, flexible, but harder to find connections)

### ⚡ One-liner to remember:
> SQL = structured + consistent. NoSQL = flexible + scalable.

---

## Q2. When should you use SQL vs NoSQL?

### Concept:

Choose based on data relationships, consistency requirements, and scale needs.

### When to Use SQL:

**Use cases:**
- Banking and payments
- Healthcare (HIPAA systems)
- E-commerce orders
- Inventory management
- Reporting and analytics

**Characteristics:**
- Data is highly relational
- Need strong transactions
- Data consistency is critical

**Example:**

```sql
-- E-commerce order with inventory update (ACID transaction)
BEGIN;
INSERT INTO orders (user_id, total) VALUES (123, 199.99);
INSERT INTO order_items (order_id, product_id, qty) VALUES (1, 50, 2);
UPDATE inventory SET stock = stock - 2 WHERE product_id = 50;
COMMIT;
```

### When to Use NoSQL:

**Use cases:**
- User sessions
- Caching
- Logs and analytics
- Real-time feeds
- IoT data streams

**Characteristics:**
- Schema changes frequently
- Need massive horizontal scale
- High read/write throughput

**Example:**

```typescript
// User session (flexible structure, fast writes)
await redis.set(`session:${userId}`, JSON.stringify({
  token: "abc123",
  lastActive: Date.now(),
  preferences: { theme: "dark" }
}), { EX: 3600 });
```

### Decision Guide:

| Scenario                  | Best Choice       | Reason                    |
|---------------------------|-------------------|---------------------------|
| Financial transactions    | SQL               | Strong consistency        |
| User profiles             | NoSQL             | Flexible schema           |
| Read-heavy dashboards     | SQL               | Complex queries & joins   |
| High-write event logs     | NoSQL             | Write-optimized           |
| Caching                   | NoSQL (Redis)     | In-memory speed           |
| Multi-tenant SaaS core    | SQL               | Integrity + RLS           |

### Real-World Architecture (Hybrid):

Most systems use **both**:

```
PostgreSQL → Orders, Users, Payments (core business data)
Redis      → Sessions, Rate limiting (cache)
MongoDB    → Activity logs, Analytics (flexible data)
```

**Example:**

```typescript
// User places order
await postgres.query("INSERT INTO orders ...");  // SQL for transaction

// Cache user session
await redis.set(`user:${id}`, session);          // NoSQL for speed

// Log activity
await mongo.insert({ event: "order_placed" });   // NoSQL for logs
```

### Analogy:

Think of a **restaurant**:
- **SQL** → Kitchen inventory (precise counts, must be accurate)
- **NoSQL** → Customer preferences (flexible, changes often)
- **Hybrid** → Use both for the complete system

### ⚡ One-liner to remember:
> SQL for critical data integrity, NoSQL for flexibility and scale — use both together.

---

## Q8. How do you handle database migrations in production?

### Concept:

Database migrations are **version-controlled schema changes** that modify your database structure (tables, columns, indexes) in a safe, reproducible way.

### Why Migrations Matter:

| Without Migrations | With Migrations |
|-------------------|-----------------|
| Manual SQL changes (error-prone) | Automated, version-controlled |
| No rollback capability | Can revert changes safely |
| Schema drifts across environments | Consistent across dev/staging/prod |
| Downtime during changes | Zero-downtime deployments possible |

---

## 1. Drizzle ORM Migrations

### Setup

```typescript
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  out: "./drizzle/migrations",
  driver: "pg",
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!
  }
});
```

### Define Schema

```typescript
// src/db/schema.ts
import { pgTable, serial, text, integer, timestamp, index } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: timestamp("created_at").defaultNow()
}, (table) => ({
  emailIdx: index("email_idx").on(table.email)
}));

export const posts = pgTable("posts", {
  id: serial("id").primaryKey(),
  title: text("title").notNull(),
  content: text("content"),
  authorId: integer("author_id")
    .notNull()
    .references(() => users.id, { onDelete: "cascade" }),
  publishedAt: timestamp("published_at")
});
```

### Generate Migration

```bash
# Generate migration from schema changes
npx drizzle-kit generate:pg

# Creates: drizzle/migrations/0001_create_users_posts.sql
```

**Generated SQL:**

```sql
-- drizzle/migrations/0001_create_users_posts.sql
CREATE TABLE IF NOT EXISTS "users" (
  "id" serial PRIMARY KEY NOT NULL,
  "email" text NOT NULL,
  "name" text NOT NULL,
  "created_at" timestamp DEFAULT now(),
  CONSTRAINT "users_email_unique" UNIQUE("email")
);

CREATE TABLE IF NOT EXISTS "posts" (
  "id" serial PRIMARY KEY NOT NULL,
  "title" text NOT NULL,
  "content" text,
  "author_id" integer NOT NULL,
  "published_at" timestamp
);

CREATE INDEX IF NOT EXISTS "email_idx" ON "users" ("email");

ALTER TABLE "posts" ADD CONSTRAINT "posts_author_id_users_id_fk"
  FOREIGN KEY ("author_id") REFERENCES "users"("id") ON DELETE cascade;
```

### Run Migration

```typescript
// src/db/migrate.ts
import { drizzle } from "drizzle-orm/node-postgres";
import { migrate } from "drizzle-orm/node-postgres/migrator";
import { Pool } from "pg";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

const db = drizzle(pool);

async function runMigrations() {
  console.log("Running migrations...");

  await migrate(db, {
    migrationsFolder: "./drizzle/migrations"
  });

  console.log("Migrations complete!");
  await pool.end();
}

runMigrations().catch(console.error);
```

```bash
# Run migrations
npm run migrate
```

### Adding New Column (Safe Migration)

```typescript
// src/db/schema.ts - Add bio column
export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  bio: text("bio"), // New column (nullable = safe)
  createdAt: timestamp("created_at").defaultNow()
});
```

```bash
# Generate new migration
npx drizzle-kit generate:pg
# Creates: drizzle/migrations/0002_add_bio_column.sql
```

**Generated SQL:**

```sql
ALTER TABLE "users" ADD COLUMN "bio" text;
```

---

## 2. Prisma Migrations

### Schema Definition

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now()) @map("created_at")

  @@index([email])
  @@map("users")
}

model Post {
  id          Int       @id @default(autoincrement())
  title       String
  content     String?
  published   Boolean   @default(false)
  authorId    Int       @map("author_id")
  author      User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  publishedAt DateTime? @map("published_at")

  @@map("posts")
}
```

### Create Migration

```bash
# Create migration
npx prisma migrate dev --name init

# Creates: prisma/migrations/20240101000000_init/migration.sql
```

### Apply to Production

```bash
# Apply migrations (production)
npx prisma migrate deploy
```

### Migration Script

```typescript
// scripts/migrate.ts
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  console.log("Running Prisma migrations...");

  // Prisma automatically runs migrations via CLI
  // This script can include custom logic

  console.log("Migrations complete!");
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

---

## 3. TypeORM Migrations

### Entity Definition

```typescript
// src/entities/User.ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, OneToMany, Index } from "typeorm";
import { Post } from "./Post";

@Entity("users")
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  @Index()
  email: string;

  @Column()
  name: string;

  @Column({ nullable: true })
  bio: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @CreateDateColumn({ name: "created_at" })
  createdAt: Date;
}
```

### Generate Migration

```bash
# Generate migration from entity changes
npm run typeorm migration:generate -- -n CreateUsers

# Creates: src/migrations/1234567890-CreateUsers.ts
```

**Generated Migration:**

```typescript
// src/migrations/1234567890-CreateUsers.ts
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateUsers1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE "users" (
        "id" SERIAL PRIMARY KEY,
        "email" VARCHAR NOT NULL UNIQUE,
        "name" VARCHAR NOT NULL,
        "bio" VARCHAR,
        "created_at" TIMESTAMP NOT NULL DEFAULT now()
      )
    `);

    await queryRunner.query(`
      CREATE INDEX "IDX_email" ON "users" ("email")
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX "IDX_email"`);
    await queryRunner.query(`DROP TABLE "users"`);
  }
}
```

### Run Migration

```bash
# Run migrations
npm run typeorm migration:run

# Revert last migration
npm run typeorm migration:revert
```

---

## Production Migration Strategies

### 1. Forward-Only Migrations (Recommended)

**Never rollback in production — always migrate forward:**

```typescript
// ❌ Bad: Trying to rollback
await runMigration.down();

// ✅ Good: Fix with new forward migration
// migration_003_fix_column.sql
ALTER TABLE users ADD COLUMN email_verified boolean DEFAULT false;
```

**Why forward-only?**
- Rollbacks are risky in production
- Data loss potential
- May not restore previous state completely

### 2. Zero-Downtime Migrations

**Breaking change: Renaming column**

```sql
-- ❌ Bad: Causes downtime
ALTER TABLE users RENAME COLUMN name TO full_name;
-- Old code breaks immediately

-- ✅ Good: Multi-step migration
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN full_name text;

-- Step 2: Backfill data
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- Step 3: Deploy code reading both columns
-- (Application handles both name and full_name)

-- Step 4: Make new column NOT NULL
ALTER TABLE users ALTER COLUMN full_name SET NOT NULL;

-- Step 5: Deploy code using only full_name

-- Step 6: Drop old column
ALTER TABLE users DROP COLUMN name;
```

### 3. Additive Changes (Safe)

**Safe migrations that don't break existing code:**

```sql
-- ✅ Safe: Adding nullable column
ALTER TABLE users ADD COLUMN phone text;

-- ✅ Safe: Adding new table
CREATE TABLE notifications (
  id serial PRIMARY KEY,
  user_id integer REFERENCES users(id),
  message text NOT NULL
);

-- ✅ Safe: Adding index (use CONCURRENTLY)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- ✅ Safe: Adding default value
ALTER TABLE posts ADD COLUMN status text DEFAULT 'draft';
```

### 4. Dangerous Migrations (Requires Planning)

```sql
-- ⚠️ Dangerous: Dropping column (data loss)
ALTER TABLE users DROP COLUMN bio;

-- ⚠️ Dangerous: Adding NOT NULL without default
ALTER TABLE users ADD COLUMN age integer NOT NULL;
-- Fails if existing rows

-- ⚠️ Dangerous: Changing column type
ALTER TABLE users ALTER COLUMN id TYPE bigint;
-- Can cause downtime on large tables

-- ⚠️ Dangerous: Removing constraint
ALTER TABLE posts DROP CONSTRAINT posts_author_id_fk;
-- May break referential integrity
```

---

## Production Best Practices

### 1. Test Migrations

```typescript
// tests/migrations.test.ts
import { migrate } from "./migrate";
import { db } from "./db";

describe("Migrations", () => {
  it("should run all migrations successfully", async () => {
    await migrate();

    // Verify schema
    const result = await db.query(`
      SELECT column_name, data_type
      FROM information_schema.columns
      WHERE table_name = 'users'
    `);

    expect(result.rows).toContainEqual({
      column_name: "email",
      data_type: "text"
    });
  });

  it("should handle existing data correctly", async () => {
    // Insert test data
    await db.query("INSERT INTO users (email, name) VALUES ('test@test.com', 'Test')");

    // Run migration that adds column
    await migrate();

    // Verify data integrity
    const user = await db.query("SELECT * FROM users WHERE email = 'test@test.com'");
    expect(user.rows[0].name).toBe("Test");
  });
});
```

### 2. Staging Environment First

```bash
# 1. Apply to staging
DATABASE_URL=staging_db npm run migrate

# 2. Test application
npm run test:integration

# 3. Monitor for 24 hours

# 4. Apply to production (if successful)
DATABASE_URL=prod_db npm run migrate
```

### 3. Backup Before Migration

```typescript
// scripts/migrate-with-backup.ts
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);

async function migrateWithBackup() {
  const timestamp = new Date().toISOString().replace(/[:.]/g, "-");
  const backupFile = `backup-${timestamp}.sql`;

  console.log("Creating backup...");
  await execAsync(`pg_dump ${process.env.DATABASE_URL} > ${backupFile}`);

  console.log("Running migrations...");
  try {
    await runMigrations();
    console.log("✓ Migrations successful");
  } catch (error) {
    console.error("✗ Migration failed, restoring backup...");
    await execAsync(`psql ${process.env.DATABASE_URL} < ${backupFile}`);
    throw error;
  }

  console.log("Backup saved:", backupFile);
}
```

### 4. Migration Safety Checks

```typescript
// scripts/check-migration-safety.ts
import fs from "fs";
import path from "path";

interface SafetyCheck {
  passed: boolean;
  message: string;
}

function checkMigrationSafety(migrationFile: string): SafetyCheck[] {
  const sql = fs.readFileSync(migrationFile, "utf-8");
  const checks: SafetyCheck[] = [];

  // Check 1: No DROP TABLE
  checks.push({
    passed: !sql.match(/DROP\s+TABLE/i),
    message: "❌ Contains DROP TABLE (destructive)"
  });

  // Check 2: No DROP COLUMN
  checks.push({
    passed: !sql.match(/DROP\s+COLUMN/i),
    message: "❌ Contains DROP COLUMN (data loss)"
  });

  // Check 3: Adding NOT NULL requires default
  const addNotNull = sql.match(/ADD\s+COLUMN\s+\w+\s+\w+\s+NOT\s+NULL/i);
  const hasDefault = sql.match(/DEFAULT/i);
  checks.push({
    passed: !addNotNull || hasDefault,
    message: "❌ Adding NOT NULL without DEFAULT"
  });

  // Check 4: Use CONCURRENTLY for indexes
  const createIndex = sql.match(/CREATE\s+INDEX/i);
  const hasConcurrently = sql.match(/CONCURRENTLY/i);
  checks.push({
    passed: !createIndex || hasConcurrently,
    message: "⚠️  CREATE INDEX should use CONCURRENTLY"
  });

  return checks;
}

// Usage
const checks = checkMigrationSafety("./migrations/0003_add_column.sql");
checks.forEach(check => {
  if (!check.passed) {
    console.error(check.message);
  }
});

const allPassed = checks.every(c => c.passed);
if (!allPassed) {
  process.exit(1);
}
```

### 5. CI/CD Integration

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm ci

      - name: Check migration safety
        run: npm run check-migrations

      - name: Test migrations on staging
        env:
          DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
        run: npm run migrate

      - name: Run integration tests
        run: npm run test:integration

      - name: Backup production database
        env:
          DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}
        run: npm run backup

      - name: Migrate production
        env:
          DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}
        run: npm run migrate

      - name: Deploy application
        run: npm run deploy
```

---

## Migration Comparison

| Feature | Drizzle | Prisma | TypeORM |
|---------|---------|--------|---------|
| SQL generation | ✅ Auto | ✅ Auto | ✅ Auto + Manual |
| Type safety | ✅ Excellent | ✅ Excellent | ⚠️ Good |
| Rollback support | ❌ Forward-only | ✅ Yes | ✅ Yes |
| Zero-downtime | 🛠 Manual | 🛠 Manual | 🛠 Manual |
| SQL customization | ✅ Full control | ⚠️ Limited | ✅ Full control |
| Migration history | ✅ SQL files | ✅ SQL files + DB table | ✅ Migrations table |

---

## Common Migration Patterns

### 1. Adding Enum Type

```typescript
// Drizzle
import { pgEnum } from "drizzle-orm/pg-core";

export const roleEnum = pgEnum("role", ["user", "admin", "moderator"]);

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  role: roleEnum("role").default("user")
});
```

**Generated SQL:**

```sql
CREATE TYPE "role" AS ENUM ('user', 'admin', 'moderator');
ALTER TABLE "users" ADD COLUMN "role" "role" DEFAULT 'user';
```

### 2. Backfilling Data

```typescript
// migration_005_backfill_slugs.ts
import { sql } from "drizzle-orm";

export async function up(db: Database) {
  // Add column
  await db.execute(sql`
    ALTER TABLE posts ADD COLUMN slug text
  `);

  // Backfill slugs
  await db.execute(sql`
    UPDATE posts
    SET slug = lower(regexp_replace(title, '[^a-zA-Z0-9]+', '-', 'g'))
    WHERE slug IS NULL
  `);

  // Make NOT NULL after backfill
  await db.execute(sql`
    ALTER TABLE posts ALTER COLUMN slug SET NOT NULL
  `);
}
```

### 3. Data Migration with Validation

```typescript
export async function up(db: Database) {
  // Add new column
  await db.execute(sql`ALTER TABLE users ADD COLUMN email_verified boolean DEFAULT false`);

  // Migrate data based on logic
  const users = await db.execute(sql`SELECT id, email FROM users`);

  for (const user of users) {
    const verified = user.email.endsWith("@company.com");
    await db.execute(sql`
      UPDATE users
      SET email_verified = ${verified}
      WHERE id = ${user.id}
    `);
  }

  console.log(`Migrated ${users.length} users`);
}
```

---

## Migration Checklist

**Before running migration:**
- [ ] Test migration on local database
- [ ] Test migration on staging environment
- [ ] Backup production database
- [ ] Check migration for dangerous operations
- [ ] Ensure migration is additive (doesn't break existing code)
- [ ] Plan rollback strategy (forward migration to fix)
- [ ] Schedule during low-traffic period
- [ ] Notify team of migration

**During migration:**
- [ ] Monitor database performance
- [ ] Watch for errors in application logs
- [ ] Check database locks (long-running migrations)
- [ ] Monitor query performance

**After migration:**
- [ ] Verify data integrity
- [ ] Run application smoke tests
- [ ] Monitor for increased error rates
- [ ] Keep backup for 7+ days

---

## Analogy:

Think of database migrations like **renovating a house while people live in it**:
- **Forward-only** = Never tear down a wall until the new one is built
- **Zero-downtime** = Build new room before removing old one
- **Backup** = Take photos before renovating
- **Staging test** = Try renovation on model home first
- **Safety checks** = Building inspector approval

---

### ⚡ One-liner to remember:
> Migrations = Version-controlled schema changes; test on staging, backup production, prefer additive changes, always go forward.

---
