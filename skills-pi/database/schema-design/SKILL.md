---
name: schema-design
description: PostgreSQL schema conventions including UUID PKs, required columns, constraint naming, FK ON DELETE behavior, and indexing rules.
---
# schema-design.md

## Purpose

Define conventions for designing database schemas — covering table naming,
column conventions, indexing rules, constraints, and relationship patterns
using PostgreSQL as the primary database.

---

## Conventions

### Table Naming

- Always use `snake_case` plural nouns: `users`, `orders`, `order_items`
- Junction/pivot tables: combine both table names alphabetically: `role_users`, `tag_posts`
- Never use reserved SQL words as table names: `user`, `order`, `group`
- Never use prefixes like `tbl_` or `t_`

### Column Naming

- Always `snake_case`: `first_name`, `created_at`, `is_active`
- Boolean columns: prefix with `is_` or `has_`: `is_active`, `has_verified_email`
- Timestamp columns: suffix with `_at`: `created_at`, `updated_at`, `deleted_at`
- Foreign keys: reference table singular + `_id`: `user_id`, `order_id`
- Never use ambiguous names: `data`, `info`, `value`, `flag`

### Primary Keys

- Always use `UUID` as primary key — never auto-increment integers for public-facing APIs
- Column name: always `id`
- Default value: `gen_random_uuid()` (PostgreSQL 13+)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ...
);
```

### Required Columns (Every Table)

Every table must include:

```sql
id          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
```

For soft-delete tables also include:

```sql
deleted_at  TIMESTAMPTZ NULL,  -- NULL means not deleted
```

### Constraints

- Always define `NOT NULL` on columns that must have a value
- Always define `UNIQUE` constraints at the DB level — not just application level
- Always define `CHECK` constraints for value ranges and enums
- Always define `FOREIGN KEY` constraints with explicit `ON DELETE` behavior

```sql
-- Explicit constraint naming convention: <table>_<column>_<type>
ALTER TABLE orders ADD CONSTRAINT orders_status_check
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'));

ALTER TABLE order_items ADD CONSTRAINT order_items_quantity_check
    CHECK (quantity > 0);

ALTER TABLE order_items ADD CONSTRAINT order_items_order_id_fk
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE;
```

### Foreign Key ON DELETE Behavior

| Relationship               | ON DELETE behavior   |
| -------------------------- | -------------------- |
| Child depends on parent    | CASCADE              |
| Child can exist alone      | SET NULL             |
| Deletion should be blocked | RESTRICT             |
| Log/audit records          | SET NULL or RESTRICT |

### Indexing Rules

- Always index foreign key columns
- Always index columns used in `WHERE`, `ORDER BY`, or `JOIN` conditions frequently
- Always index columns used for lookups: `email`, `slug`, `external_id`
- Use partial indexes for filtered queries on large tables
- Use composite indexes when queries filter on multiple columns together
- Never over-index — each index slows down writes

```sql
-- Index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Index lookup columns
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Partial index for active records only
CREATE INDEX idx_orders_status_active ON orders(status)
    WHERE status NOT IN ('delivered', 'cancelled');

-- Composite index for common query pattern
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### Enums

- Use PostgreSQL `ENUM` type or `CHECK` constraints for fixed value sets
- Document all possible values inline as a comment
- Prefer `CHECK` constraints over `ENUM` type for easier migrations

### Soft Deletes

- Use `deleted_at TIMESTAMPTZ NULL` for soft deletes
- Never hard delete records that are referenced by other tables
- Always filter `WHERE deleted_at IS NULL` in application queries
- Create a partial index to exclude deleted records from common queries

---

## Anti-Patterns

- Never use auto-increment integers as public-facing IDs — use UUID
- Never skip `NOT NULL` on required columns
- Never rely on application-level uniqueness only — enforce at DB level with UNIQUE constraint
- Never use vague column names: `data`, `value`, `flag`, `misc`
- Never forget to index foreign key columns
- Never use `ON DELETE CASCADE` without understanding the full deletion impact
- Never store JSON blobs for data that needs to be queried — normalize it

---

## Schema Checklist (Run Before Every Migration)

- [ ] All tables have `id`, `created_at`, `updated_at`
- [ ] All foreign keys indexed
- [ ] All required columns have `NOT NULL`
- [ ] All unique fields have `UNIQUE` constraint at DB level
- [ ] All `ON DELETE` behaviors explicitly defined
- [ ] All constraint names follow naming convention
- [ ] No reserved words used as table or column names

---

## Ready-to-Use Prompt

```
Task: Design schema for [feature or domain name]
Skill: database/schema-design, database/data-modeling, database/migrations

REQUIREMENTS:
- Entities: [list tables needed]
- Relationships: [describe one-to-many, many-to-many, etc.]
- Soft delete needed: [yes/no]
- Enums/fixed values: [list constrained columns]

CONSTRAINTS:
- UUID primary keys with gen_random_uuid()
- Every table: id, created_at, updated_at
- All foreign keys indexed
- All unique fields have DB-level UNIQUE constraint
- Explicit ON DELETE behavior on all foreign keys
- snake_case naming throughout
- No reserved SQL words as names

DONE WHEN:
- Schema checklist passes
- All relationships modeled correctly
- Indexes cover all FK and lookup columns
- Migration file created following migrations.md
- Code review skill applied before commit
```

