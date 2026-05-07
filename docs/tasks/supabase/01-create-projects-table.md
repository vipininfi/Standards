# Task: Create the Projects Table

**Platform:** Supabase
**Covers:** [New Table Checklist](../../checklists/supabase/new-table.md) · [Database & RLS Standards](../../standards/supabase/01-database-and-rls.md)

---

## Scenario

You are setting up the database for **WorkFlow** — a multi-tenant project management app. Each workspace can have multiple projects. Projects are created by workspace members and belong to a specific workspace. Projects can be soft-deleted (not permanently removed).

---

## What to Build

A `projects` table in Supabase with the correct schema, constraints, indexes, soft delete support, and an `updated_at` auto-update trigger. You will write the complete migration SQL file.

---

## Table Requirements

### Columns

| Column | Type | Nullable | Default | Notes |
|--------|------|----------|---------|-------|
| `id` | `uuid` | No | `gen_random_uuid()` | Primary key |
| `workspace_id` | `uuid` | No | — | FK → `workspaces(id)` ON DELETE CASCADE |
| `created_by` | `uuid` | No | — | FK → `auth.users(id)` ON DELETE RESTRICT |
| `name` | `text` | No | — | Project name, max enforced at API layer |
| `description` | `text` | Yes | `null` | Optional, max 500 chars at API layer |
| `status` | `text` | No | `'active'` | Must be: active, on-hold, or archived |
| `start_date` | `date` | Yes | `null` | Project start date |
| `end_date` | `date` | Yes | `null` | Must be after start_date when set |
| `created_at` | `timestamptz` | No | `now()` | Auto-set on insert |
| `updated_at` | `timestamptz` | No | `now()` | Auto-updated on every change |
| `deleted_at` | `timestamptz` | Yes | `null` | NULL = active, timestamp = soft-deleted |

### Constraints Required

- Primary key on `id`
- `workspace_id` NOT NULL + FK to `workspaces(id)` ON DELETE CASCADE
- `created_by` NOT NULL + FK to `auth.users(id)` ON DELETE RESTRICT
- `name` NOT NULL
- `status` CHECK: must be `'active'`, `'on-hold'`, or `'archived'`
- Uniqueness: `(workspace_id, name)` — no two projects with the same name in the same workspace (active projects only — soft deleted can repeat)
- CHECK: `end_date > start_date` when both are set

### Indexes Required

- `workspace_id` — most queries filter by workspace
- `created_by` — for user's own projects view
- `status` — for filtering active/archived
- Composite: `(workspace_id, deleted_at)` — for listing active projects per workspace

### Trigger Required

An `updated_at` auto-update trigger using the shared `update_updated_at()` function (create the function if it does not exist).

---

## Migration File

Write a complete SQL migration file named:

```
20240115_create_projects_table.sql
```

It must include (in this order):

1. Create the `update_updated_at()` function (with `CREATE OR REPLACE`)
2. Create the `projects` table with all columns and constraints
3. Create the `set_updated_at` trigger on `projects`
4. Create all required indexes
5. Enable RLS: `ALTER TABLE projects ENABLE ROW LEVEL SECURITY;`

> Leave RLS policies for the next task — this migration only enables RLS without policies yet.

---

## What You Should NOT Do

- Do not skip the `deleted_at` column and use hard deletes — soft delete is required
- Do not use `serial` or `integer` for the primary key — use `uuid`
- Do not name columns with camelCase — use `snake_case`
- Do not use `VARCHAR(n)` — use `text` with CHECK constraints or application-layer validation
- Do not create the trigger without the function — they must both be in the migration
- Do not skip the uniqueness constraint — duplicate project names in the same workspace should be blocked at the DB level

---

## Checklist to Run When Done

Use the [New Table Checklist](../../checklists/supabase/new-table.md#10-new-table-checklist--before-marking-done).

---

## Done When

```text
SCHEMA
[ ] All 11 columns present with correct types and nullability
[ ] id: uuid primary key, default gen_random_uuid()
[ ] workspace_id: NOT NULL, FK to workspaces(id) ON DELETE CASCADE
[ ] created_by: NOT NULL, FK to auth.users(id) ON DELETE RESTRICT
[ ] status: NOT NULL, CHECK constraint with 3 allowed values
[ ] Unique constraint on (workspace_id, name) for active projects
[ ] CHECK constraint: end_date > start_date when both set
[ ] deleted_at: nullable, NULL = active

TRIGGER
[ ] update_updated_at() function created (or already exists)
[ ] set_updated_at trigger created on projects table

INDEXES
[ ] workspace_id indexed
[ ] created_by indexed
[ ] status indexed
[ ] Composite index on (workspace_id, deleted_at)

RLS
[ ] RLS enabled on table (ALTER TABLE projects ENABLE ROW LEVEL SECURITY)
[ ] No policies yet (next task)

MIGRATION
[ ] File named with timestamp prefix
[ ] Runs without errors on a clean database
[ ] All objects created in correct dependency order (function before trigger, table before indexes)
```
