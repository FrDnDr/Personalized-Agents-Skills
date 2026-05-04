# migrations.md

## Purpose

Define conventions for creating, versioning, running, and rolling back
database migrations using Alembic (Python/SQLAlchemy) as the primary
migration tool.

---

## Conventions

### Tool Stack

- **Alembic** — primary migration tool for Python/SQLAlchemy projects
- **Flyway** or **Liquibase** — for Java or polyglot projects
- Never modify the database schema directly in production — always through migrations

### Folder Structure

```
/alembic
  env.py                  # Alembic environment config
  script.py.mako          # Migration file template
  /versions
    001_create_users.py
    002_add_orders_table.py
    003_add_index_orders_user_id.py
```

### Migration File Naming

Format: `<sequence>_<descriptive_action>.py`

- Use sequential numbering: `001`, `002`, `003`
- Use descriptive snake_case action names
- Action verbs: `create`, `add`, `remove`, `rename`, `alter`, `drop`, `add_index`, `drop_index`

```
001_create_users_table.py
002_create_orders_table.py
003_add_status_to_orders.py
004_add_index_orders_user_id.py
005_rename_fullname_to_full_name_in_users.py
```

### Migration File Structure

Every migration must have both `upgrade()` and `downgrade()`:

```python
"""add status column to orders

Revision ID: a1b2c3d4e5f6
Revises: 9z8y7x6w5v4u
Create Date: 2025-01-15 10:30:00
"""
from alembic import op
import sqlalchemy as sa

revision = 'a1b2c3d4e5f6'
down_revision = '9z8y7x6w5v4u'

def upgrade() -> None:
    op.add_column('orders',
        sa.Column('status', sa.String(50), nullable=False, server_default='pending')
    )
    op.create_check_constraint(
        'orders_status_check',
        'orders',
        "status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')"
    )

def downgrade() -> None:
    op.drop_constraint('orders_status_check', 'orders')
    op.drop_column('orders', 'status')
```

### Rules for Writing Migrations

#### Always

- Always write a `downgrade()` that fully reverses the `upgrade()`
- Always give constraints explicit names following `schema-design.md` conventions
- Always set `server_default` when adding a `NOT NULL` column to an existing table
- Always test both `upgrade` and `downgrade` locally before committing

#### Never

- Never modify an existing migration file that has already been run in any environment
- Never delete migration files
- Never write raw Python data manipulation in migrations — schema changes only
- Never drop a column without first deprecating it (remove from code, then drop in next release)
- Never run migrations directly against production without a backup

### Running Migrations

```bash
# Apply all pending migrations
alembic upgrade head

# Apply one migration at a time
alembic upgrade +1

# Roll back one migration
alembic downgrade -1

# Roll back to a specific revision
alembic downgrade a1b2c3d4e5f6

# Check current revision
alembic current

# Show migration history
alembic history --verbose
```

### Migration Workflow

```
1. Make schema changes in SQLAlchemy models
2. Generate migration: alembic revision --autogenerate -m "description"
3. Review generated migration — autogenerate is not always correct
4. Add explicit constraint names if missing
5. Write and verify downgrade() is correct
6. Run upgrade locally: alembic upgrade head
7. Test application against migrated schema
8. Test downgrade: alembic downgrade -1
9. Run upgrade again: alembic upgrade head
10. Commit migration file with application code changes together
```

### Production Migration Rules

- Always take a database backup before running migrations in production
- Always run migrations in a maintenance window for destructive changes
- Always test migration on staging environment first
- Use `alembic upgrade head` in CI/CD pipeline — never manual SQL scripts
- Monitor migration execution time — alert if it exceeds expected duration

### Data Migrations

When a migration requires transforming existing data:

- Separate schema change and data migration into two separate migration files
- Schema migration runs first (adds new column)
- Data migration runs second (backfills data)
- Keep data migrations idempotent — safe to run multiple times

---

## Anti-Patterns

- Never edit a migration file that has already been applied anywhere
- Never skip writing `downgrade()` — rollback must always be possible
- Never add a `NOT NULL` column without a `server_default` to existing tables
- Never drop a column in the same release it was removed from application code
- Never run raw SQL directly in production to work around migrations
- Never commit a migration without testing both upgrade and downgrade locally

---

## Ready-to-Use Prompt

```
Task: Create migration for [schema change description]
Skill: database/migrations, database/schema-design

CHANGE TYPE:
[ ] Create new table
[ ] Add column(s) to existing table
[ ] Remove column(s) from existing table
[ ] Add index
[ ] Add/modify constraint
[ ] Data backfill (separate migration)

REQUIREMENTS:
- Table: [name]
- Change: [describe exactly what needs to change]
- Nullable or NOT NULL: [if new column]
- Default value: [if NOT NULL on existing table]

CONSTRAINTS:
- Migration file named: <next_sequence>_<action>_<description>.py
- Both upgrade() and downgrade() must be complete
- All constraints explicitly named
- Test upgrade AND downgrade locally before committing
- Never modify existing migration files

DONE WHEN:
- upgrade() applies cleanly from current state
- downgrade() fully reverses the upgrade
- Both tested locally
- Migration committed alongside application code changes
- Code review skill applied before commit
```
