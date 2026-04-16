---
name: migrate
description: Create and run database migrations safely, with rollback plan
license: MIT
compatibility: opencode
metadata:
  tags: database, migrations
---

## What I do

Write, validate, and apply database schema or data migrations with safety checks, backward-compatibility consideration, and a tested rollback path.

## Steps

1. **Understand the change** — what schema or data is changing and why? Confirm before writing.
2. **Identify the migration tool**:
   - Prisma: `prisma migrate dev`, schema in `prisma/schema.prisma`
   - Alembic (Python): `alembic revision --autogenerate -m "description"`
   - Knex: `knex migrate:make <name>`
   - Flyway / Liquibase: numbered SQL files
   - Rails ActiveRecord: `rails generate migration <Name>`
   - Raw SQL: timestamped files in `db/migrations/`
3. **Read 2-3 existing migrations** to match naming, numbering, and content conventions
4. **Write the `up` migration**:
   - Schema: `ALTER TABLE`, `CREATE INDEX`, `ADD COLUMN`, `DROP COLUMN`
   - Data: batch updates (avoid full-table locks on large tables — see below)
5. **Write the `down` (rollback) migration**:
   - Every `up` must have a corresponding `down` that fully reverses it
   - If truly irreversible (data deletion, dropped unique constraint), document it explicitly with a comment
6. **Safety checks before running**:
   - Will this lock a large table? Use `CONCURRENTLY` for index creation; batch data changes
   - Does it drop or rename anything the running app code currently uses? → deploy app update first, then migrate
   - Is the app backward-compatible with both old and new schema? → required for zero-downtime deploys
7. **Run against non-production first**: dev → staging → production
8. **Apply to production** only after staging succeeded

## Tool-specific examples

### Prisma
```prisma
// schema.prisma — add nullable column first (backward-compatible)
model User {
  id        String   @id
  email     String   @unique
  avatarUrl String?  // new nullable column — safe to deploy before populating
}
```
```bash
npx prisma migrate dev --name add_user_avatar_url
npx prisma migrate deploy  # production
```

### Alembic (Python)
```python
# migrations/versions/20260416_add_avatar_url.py
def upgrade():
    op.add_column('users', sa.Column('avatar_url', sa.String(), nullable=True))

def downgrade():
    op.drop_column('users', 'avatar_url')
```

### Raw SQL — safe large-table index creation
```sql
-- UP
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders(user_id);

-- DOWN
DROP INDEX CONCURRENTLY idx_orders_user_id;
```

### Batched data migration (avoid full-table lock)
```sql
-- UP: backfill in batches of 1000 to avoid long lock
DO $$
DECLARE
  last_id BIGINT := 0;
BEGIN
  LOOP
    UPDATE users
    SET status = 'active'
    WHERE id > last_id AND id <= last_id + 1000 AND status IS NULL;

    EXIT WHEN NOT FOUND;
    last_id := last_id + 1000;
    PERFORM pg_sleep(0.01);  -- brief pause between batches
  END LOOP;
END $$;
```

## Zero-downtime deploy order

```
Adding a column:    migrate first → deploy app
Removing a column:  deploy app (stop using column) → migrate
Renaming a column:  add new column → deploy app (write to both) → backfill → deploy app (read new only) → drop old
Changing a type:    add new column → backfill → swap → drop old
```

## Rules

- Never run a destructive migration (DROP TABLE, TRUNCATE, mass DELETE) without explicit user confirmation
- Data migrations must be idempotent where possible (`UPDATE ... WHERE status IS NULL` not `UPDATE ...`)
- Never modify an already-applied migration — create a new one
- Always check that the app can run against both pre- and post-migration schema during rolling deploys
- `CREATE INDEX` on a production table must use `CONCURRENTLY` to avoid locking reads
