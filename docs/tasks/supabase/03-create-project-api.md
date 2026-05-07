# Task: Build a Create Project API Endpoint

**Platform:** Supabase (RPC / Postgres Function)
**Covers:** [New API Endpoint Checklist](../../checklists/supabase/new-api-endpoint.md) · [API Design Standards](../../standards/supabase/02-api-design.md) · [API Documentation Standards](../../standards/supabase/03-api-documentation.md)

---

## Scenario

Workspace members need to create projects. A direct `supabase.from('projects').insert(...)` is not sufficient here because creating a project must also:

1. Validate the project name is unique within the workspace
2. Create a default "General" task list inside the project
3. Log an audit entry

These three steps must be atomic — all succeed or none do. This means it must be an **RPC (Postgres Function)**.

---

## What to Build

### Part 1 — The RPC Function

A Postgres function named `create_project` that:

- Takes inputs: `p_workspace_id`, `p_name`, `p_description` (optional), `p_start_date` (optional), `p_end_date` (optional)
- Validates all inputs
- Checks the caller is a member of the workspace (not just any user)
- Checks the project name is unique within the workspace
- Inserts the project
- Inserts a default task list named "General" linked to the new project
- Inserts an audit log entry
- Returns the created project as JSON
- Raises exceptions on every failure case with a consistent error format

### Part 2 — The API Documentation

Write the complete API documentation for this endpoint following the [API Documentation Template](../../templates/api-doc-template.md).

---

## RPC Specification

### Signature

```sql
CREATE OR REPLACE FUNCTION create_project(
  p_workspace_id  uuid,
  p_name          text,
  p_description   text DEFAULT NULL,
  p_start_date    date DEFAULT NULL,
  p_end_date      date DEFAULT NULL
)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER  -- RLS still applies, user must be a workspace member
```

### Validation Steps (in this order)

1. `p_workspace_id` — must not be null
2. `p_name` — must not be null or empty after trim; max 100 characters
3. `p_description` — if provided, max 500 characters
4. `p_end_date` — if provided and `p_start_date` is also provided, end must be after start
5. Workspace membership — caller (`auth.uid()`) must be a member with role: owner, admin, or member (not viewer)
6. Uniqueness — no active project with the same name in this workspace (`deleted_at IS NULL`)

### Error Format

Use `RAISE EXCEPTION` with this format so the frontend can parse the error code:

```sql
RAISE EXCEPTION 'VALIDATION_ERROR: name is required';
RAISE EXCEPTION 'FORBIDDEN: you are not a member of this workspace';
RAISE EXCEPTION 'DUPLICATE: a project with this name already exists';
```

### Success Response

Return a JSON object:

```json
{
  "data": {
    "id": "uuid",
    "workspace_id": "uuid",
    "name": "My Project",
    "description": null,
    "status": "active",
    "start_date": null,
    "end_date": null,
    "created_by": "uuid",
    "created_at": "2024-01-15T10:00:00Z"
  }
}
```

### Related Tables

You will need these tables in the function:
- `projects` — created in Task 01
- `task_lists` — assume it exists with columns: `id`, `project_id`, `name`, `created_by`, `created_at`
- `audit_logs` — assume it exists with columns: `id`, `action`, `entity_type`, `entity_id`, `actor_id`, `created_at`

---

## Frontend Call Example

The frontend will call this as:

```ts
const { data, error } = await supabase.rpc('create_project', {
  p_workspace_id: workspaceId,
  p_name: formData.name,
  p_description: formData.description ?? null,
  p_start_date: formData.startDate ?? null,
  p_end_date: formData.endDate ?? null,
});
```

---

## What You Should NOT Do

- Do not let the frontend make three separate calls (insert project + insert task list + insert audit log) — all three must be in one atomic function
- Do not use `SECURITY DEFINER` unless you have a specific reason and explicitly verify `auth.uid()` inside the function
- Do not skip the uniqueness check — the DB constraint catches concurrent duplicates, but the function should check and return a clear error first
- Do not return raw Postgres error messages — catch errors and re-raise with standard codes
- Do not skip writing the API documentation — the frontend team cannot use this endpoint without it

---

## Checklist to Run When Done

Use the [New API Endpoint Checklist](../../checklists/supabase/new-api-endpoint.md#10-api-endpoint-checklist--before-marking-done).

---

## Done When

```text
RPC FUNCTION
[ ] Function created with correct signature and SECURITY INVOKER
[ ] All 6 validation steps implemented in order
[ ] Each validation raises exception with standard error code prefix
[ ] Workspace membership checked (role not viewer)
[ ] Uniqueness checked before insert
[ ] Project inserted
[ ] Default "General" task list inserted for the new project
[ ] Audit log entry inserted
[ ] All three inserts inside a transaction (implicit in plpgsql, but verify)
[ ] Returns JSON with full project data

ERROR CASES HANDLED
[ ] Missing workspace_id → VALIDATION_ERROR
[ ] Empty name → VALIDATION_ERROR
[ ] Name too long → VALIDATION_ERROR
[ ] End date before start date → VALIDATION_ERROR
[ ] Non-member calling → FORBIDDEN
[ ] Viewer calling → FORBIDDEN
[ ] Duplicate project name → DUPLICATE
[ ] Unexpected DB error → caught and re-raised as SERVER_ERROR

TESTING
[ ] Success: correct project + task list + audit log created
[ ] Each validation error returns correct code
[ ] Non-member call blocked
[ ] Viewer call blocked
[ ] Duplicate name blocked
[ ] Cross-workspace attempt blocked (RLS handles this)

DOCUMENTATION
[ ] API documentation written using the template
[ ] All error codes documented with example response
[ ] Frontend call example included
```
