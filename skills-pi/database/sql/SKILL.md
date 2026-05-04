---
name: sql
description: SQL query conventions for SELECT, joins, aggregations, window functions, upsert, keyset pagination, and transactions.
---
# sql.md

## Purpose

Define SQL query conventions, patterns for common operations, performance
rules, and anti-patterns to avoid when writing raw SQL or ORM queries
against PostgreSQL.

---

## Conventions

### General Query Rules

- Always use parameterized queries — never string concatenation
- Always alias tables in multi-table queries for readability
- Always specify column names in `SELECT` — never use `SELECT *` in production code
- Always use explicit `JOIN` syntax — never implicit comma joins
- Always write keywords in UPPERCASE: `SELECT`, `FROM`, `WHERE`, `JOIN`

```sql
-- WRONG
SELECT * FROM users, orders WHERE users.id = orders.user_id

-- CORRECT
SELECT
    u.id,
    u.email,
    o.id AS order_id,
    o.status,
    o.created_at
FROM users u
INNER JOIN orders o ON o.user_id = u.id
WHERE u.is_active = TRUE
  AND o.deleted_at IS NULL
```

### SELECT Patterns

- Always include only the columns you need — avoid fetching unused data
- Use `COUNT(1)` instead of `COUNT(*)` for counting rows
- Use `EXISTS` instead of `COUNT` when checking for existence

```sql
-- Checking existence — use EXISTS, not COUNT
SELECT EXISTS (
    SELECT 1 FROM users WHERE email = $1
) AS email_taken;
```

### Filtering

- Always filter soft-deleted records: `WHERE deleted_at IS NULL`
- Use `BETWEEN` for range queries on indexed columns
- Use `IN` for small fixed sets, `= ANY($1::uuid[])` for dynamic arrays
- Avoid `LIKE '%search%'` on large tables — use full-text search instead
- Use `ILIKE` for case-insensitive search only on small datasets

### Joins

| Join Type    | When to Use                                     |
| ------------ | ----------------------------------------------- |
| `INNER JOIN` | Only matching rows from both tables             |
| `LEFT JOIN`  | All rows from left, NULL for non-matching right |
| `RIGHT JOIN` | Avoid — rewrite as LEFT JOIN for consistency    |
| `FULL JOIN`  | All rows from both tables, NULLs where no match |

- Always join on indexed columns
- Never join on non-indexed columns in large tables

### Aggregations

- Always use `GROUP BY` with all non-aggregated columns in `SELECT`
- Use `HAVING` to filter on aggregated values — not `WHERE`
- Use window functions for running totals, rankings, and row-level comparisons

```sql
-- Window function: rank orders per user by date
SELECT
    user_id,
    id AS order_id,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS order_rank
FROM orders
WHERE deleted_at IS NULL;
```

### Upsert Pattern

Use PostgreSQL `INSERT ... ON CONFLICT` for upserts:

```sql
INSERT INTO user_preferences (user_id, key, value, updated_at)
VALUES ($1, $2, $3, NOW())
ON CONFLICT (user_id, key)
DO UPDATE SET
    value = EXCLUDED.value,
    updated_at = NOW();
```

### Pagination

Always use keyset pagination for large datasets — avoid `OFFSET` on large tables:

```sql
-- AVOID for large tables (slow at high offsets)
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- PREFER: keyset pagination
SELECT * FROM orders
WHERE created_at < $1  -- last seen cursor value
ORDER BY created_at DESC
LIMIT 20;
```

### Transactions

- Wrap multi-step operations in explicit transactions
- Keep transactions short — never do long I/O inside a transaction
- Always handle rollback on error

```sql
BEGIN;
    UPDATE accounts SET balance = balance - $1 WHERE id = $2;
    UPDATE accounts SET balance = balance + $1 WHERE id = $3;
    INSERT INTO transactions (from_id, to_id, amount) VALUES ($2, $3, $1);
COMMIT;
```

### Performance Rules

- Use `EXPLAIN ANALYZE` to inspect query plans before deploying complex queries
- Avoid `N+1` queries — use JOINs or batch fetches instead
- Avoid functions on indexed columns in WHERE clauses — they prevent index use

```sql
-- WRONG — function on indexed column prevents index use
WHERE LOWER(email) = 'test@example.com'

-- CORRECT — use a functional index or store pre-lowercased value
WHERE email = LOWER($1)
```

---

## Anti-Patterns

- Never use `SELECT *` in production queries
- Never build queries with string concatenation — SQL injection risk
- Never use `OFFSET` pagination on large tables — use keyset instead
- Never put functions on indexed columns in `WHERE` clauses
- Never do multi-step operations without wrapping in a transaction
- Never use `RIGHT JOIN` — rewrite as `LEFT JOIN`
- Never skip soft-delete filter — always include `WHERE deleted_at IS NULL`
- Never use `LIKE '%term%'` on large tables — use full-text search

---

## Ready-to-Use Prompt

```
Task: Write SQL queries for [feature or operation]
Skill: database/sql, database/schema-design

REQUIREMENTS:
- Operation: [SELECT / INSERT / UPDATE / DELETE / UPSERT]
- Tables involved: [list]
- Filters needed: [describe WHERE conditions]
- Joins needed: [describe relationships]
- Pagination: [yes/no — if yes, keyset preferred]
- Aggregation: [describe if needed]

CONSTRAINTS:
- Always parameterized — no string concatenation
- SELECT only needed columns — no SELECT *
- Always filter deleted_at IS NULL on soft-delete tables
- Use keyset pagination for large result sets
- Wrap multi-step operations in transactions
- Run EXPLAIN ANALYZE on any complex query

DONE WHEN:
- Query returns correct results
- EXPLAIN ANALYZE shows index usage (no full table scans on large tables)
- Parameterized with no string interpolation
- Code review skill applied before commit
```

