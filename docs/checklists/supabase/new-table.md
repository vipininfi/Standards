# Supabase — New Table Checklist

> **Core Rule:** Every table exposed through Supabase must have RLS enabled, proper constraints, indexes, and be fully documented before it is used in production.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-before-you-create-the-table) | Before You Create the Table |
| [2](#2-table-schema) | Table Schema |
| [3](#3-naming-standards) | Naming Standards |
| [4](#4-constraints) | Constraints |
| [5](#5-indexes) | Indexes |
| [6](#6-rls-setup) | RLS Setup |
| [7](#7-audit-fields) | Audit Fields |
| [8](#8-migration-file) | Migration File |
| [9](#9-documentation) | Documentation |
| [10](#10-new-table-checklist--before-marking-done) | New Table Checklist — Before Marking Done |

---

# 1. Before You Create the Table

```text
[ ] Requirements for this table are clear and documented
[ ] Confirmed this table does not already exist under a different name
[ ] Confirmed this is not better handled by extending an existing table
[ ] Multi-tenant scope confirmed: is this workspace-scoped, user-scoped, or global?
[ ] Soft delete or hard delete decision made and documented
[ ] Who can access this table and in what ways (SELECT / INSERT / UPDATE / DELETE)?
```

---

# 2. Table Schema

## Required Columns (Every Table)

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid DEFAULT gen_random_uuid()` | Primary key |
| `created_at` | `timestamptz DEFAULT now()` | Always included |
| `updated_at` | `timestamptz DEFAULT now()` | Always included |

## Workspace-Scoped Tables (Multi-Tenant)

| Column | Type | Notes |
|--------|------|-------|
| `workspace_id` | `uuid NOT NULL REFERENCES workspaces(id)` | Required for any tenant-scoped table |

## User-Owned Records

| Column | Type | Notes |
|--------|------|-------|
| `created_by` | `uuid NOT NULL REFERENCES auth.users(id)` | Track who created the record |

## Soft Delete (if applicable)

| Column | Type | Notes |
|--------|------|-------|
| `deleted_at` | `timestamptz DEFAULT NULL` | NULL = not deleted, timestamp = deleted |

---

# 3. Naming Standards

```text
[ ] Table name is plural snake_case: projects, task_items, workspace_members
[ ] Column names are singular snake_case: project_id, created_at, user_email
[ ] Foreign keys named: referenced_table_singular + _id: workspace_id, project_id, user_id
[ ] Boolean columns start with is_ or has_: is_active, is_verified, has_access
[ ] Timestamp columns end with _at: created_at, updated_at, deleted_at, verified_at
[ ] No reserved SQL keywords as column names (e.g., avoid "name" → use "project_name" or just "name" with care)
```

---

# 4. Constraints

```text
[ ] Primary key set: id uuid PRIMARY KEY
[ ] NOT NULL on every required column
[ ] FOREIGN KEY constraints with appropriate ON DELETE behavior
  — ON DELETE CASCADE: delete child when parent deleted (e.g., tasks when project deleted)
  — ON DELETE SET NULL: allow orphan (rarely needed)
  — ON DELETE RESTRICT: block parent deletion if children exist (default)
[ ] UNIQUE constraints where needed (e.g., workspace_id + slug, user_id + workspace_id)
[ ] CHECK constraints for enums or restricted values
  — Example: CHECK (status IN ('active', 'inactive', 'archived'))
[ ] CHECK constraints for numeric ranges (e.g., CHECK (percentage >= 0 AND percentage <= 100))
[ ] CHECK constraint: updated_at >= created_at if relevant
[ ] No nullable foreign keys unless intentionally optional
```

## ON DELETE Decision Guide

| Scenario | Use |
|----------|-----|
| Child records have no meaning without parent | CASCADE |
| Child records should be preserved for audit | RESTRICT or SET NULL |
| Foreign key is optional reference | SET NULL |
| Default — block parent deletion | RESTRICT |

---

# 5. Indexes

```text
[ ] Primary key already indexed (automatic)
[ ] Foreign key columns indexed:
  — workspace_id → index added
  — project_id → index added
  — user_id → index added
[ ] Columns used in WHERE clauses indexed
[ ] Columns used in ORDER BY indexed
[ ] Soft delete: index on deleted_at if filtering by it
[ ] Composite index if filtering on multiple columns together
  — Example: CREATE INDEX ON projects (workspace_id, status)
[ ] No over-indexing: only index what will be queried
```

---

# 6. RLS Setup

```text
[ ] RLS enabled on table: ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;
[ ] Default deny confirmed (no access unless explicitly granted by a policy)
[ ] SELECT policy added (who can read rows)
[ ] INSERT policy added (who can create rows)
[ ] UPDATE policy added (who can change rows)
[ ] DELETE policy added (or intentionally blocked with no policy)
[ ] Each policy uses auth.uid() — not a hardcoded value
[ ] Workspace-scoped policies verify workspace membership, not just workspace_id match
[ ] Policies tested as: owner / admin / member / viewer / non-member / anonymous
[ ] Service role not relied on for normal operations
```

## RLS Policy Patterns

```sql
-- SELECT: workspace members only
CREATE POLICY "workspace_members_select" ON projects
  FOR SELECT USING (
    workspace_id IN (
      SELECT workspace_id FROM workspace_members
      WHERE user_id = auth.uid()
    )
  );

-- INSERT: workspace members only, sets created_by automatically
CREATE POLICY "workspace_members_insert" ON projects
  FOR INSERT WITH CHECK (
    workspace_id IN (
      SELECT workspace_id FROM workspace_members
      WHERE user_id = auth.uid()
    )
    AND created_by = auth.uid()
  );

-- UPDATE: only creator or admin
CREATE POLICY "owner_or_admin_update" ON projects
  FOR UPDATE USING (
    created_by = auth.uid()
    OR EXISTS (
      SELECT 1 FROM workspace_members
      WHERE workspace_id = projects.workspace_id
      AND user_id = auth.uid()
      AND role = 'admin'
    )
  );

-- DELETE: only admin
CREATE POLICY "admin_delete" ON projects
  FOR DELETE USING (
    EXISTS (
      SELECT 1 FROM workspace_members
      WHERE workspace_id = projects.workspace_id
      AND user_id = auth.uid()
      AND role = 'admin'
    )
  );
```

---

# 7. Audit Fields

```text
[ ] created_at is set automatically (DEFAULT now())
[ ] updated_at is updated automatically via trigger (not manually in application code)
[ ] created_by is set from auth.uid() in RLS INSERT policy — not from request payload
[ ] If audit log table exists: insert log entry on significant changes
```

## Auto-Update Trigger for updated_at

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON your_table_name
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

---

# 8. Migration File

```text
[ ] Migration created in Supabase migrations folder or via Supabase CLI
[ ] Migration is reversible: includes both UP and DOWN migrations if needed
[ ] Migration name is descriptive: 20240115_create_projects_table.sql
[ ] Migration tested on local/staging before applying to production
[ ] No data mutations in schema migrations (keep schema and data migrations separate)
[ ] Migration does not depend on data that may not exist
```

---

# 9. Documentation

```text
[ ] Table purpose documented (what it stores, what entity it represents)
[ ] Column list documented (all columns, types, nullable, purpose)
[ ] Foreign key relationships documented
[ ] RLS access rules documented (who can do what)
[ ] Frontend team notified of new table and how to access it
[ ] API documentation written if a new API endpoint was created for this table
```

---

# 10. New Table Checklist — Before Marking Done

```text
SCHEMA
[ ] id uuid primary key
[ ] created_at timestamptz default now()
[ ] updated_at timestamptz default now() (with trigger)
[ ] workspace_id if workspace-scoped
[ ] created_by if user-owned
[ ] All required columns are NOT NULL
[ ] Enum/status columns have CHECK constraint

NAMING
[ ] Table name: plural snake_case
[ ] Column names: singular snake_case
[ ] Foreign keys: referenced_table_id pattern

CONSTRAINTS
[ ] Foreign keys defined with ON DELETE behavior
[ ] UNIQUE constraints where needed
[ ] CHECK constraints for restricted values

INDEXES
[ ] All foreign key columns indexed
[ ] Columns in common WHERE clauses indexed
[ ] Composite indexes if multiple columns filtered together

RLS
[ ] RLS enabled
[ ] SELECT policy added
[ ] INSERT policy added
[ ] UPDATE policy added
[ ] DELETE policy added or intentionally blocked
[ ] Policies use auth.uid() — not hardcoded values
[ ] Tested as owner, admin, member, non-member, anonymous

MIGRATION
[ ] Migration file created and named clearly
[ ] Tested on staging before production

DOCUMENTATION
[ ] Table documented
[ ] API documentation written if endpoint created
[ ] Frontend team notified
```

---

## Practice Task

Apply what you learned by creating a real table with the correct schema, constraints, indexes, soft delete, and migration file.

**→ [Task 01: Create the Projects Table](../../tasks/supabase/01-create-projects-table.md)**

Covers: all required columns (id, created_at, updated_at, workspace_id, created_by), CHECK constraints on status and end_date, soft delete column, updated_at trigger, composite index, migration file structure.
