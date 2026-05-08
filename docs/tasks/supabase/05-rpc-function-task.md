# Task — Write an RPC Function: `archive_project`

**Platform:** Supabase (Postgres / PL/pgSQL)
**Checklist to Run:** [RPC Function Checklist](../../checklists/supabase/rpc-function.md)
**Standard:** [RPC & Postgres Function Standards](../../standards/supabase/07-rpc-functions.md)

---

## Scenario

You are building the "archive project" feature. When a project is archived:
1. The project's `status` is set to `'archived'` and `archived_at` is set to `now()`.
2. All open tasks in the project are set to `status = 'on-hold'`.
3. An audit log entry is written.
4. The operation must be atomic — if any step fails, all steps are rolled back.

This is a multi-step operation that touches 3 tables and must be atomic. It belongs in an RPC function.

---

## What to Build

A Postgres function named `archive_project` exposed via Supabase RPC.

---

## Requirements

### Function Signature

```sql
CREATE OR REPLACE FUNCTION archive_project(
  p_project_id  uuid,
  p_reason      text DEFAULT NULL
)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER
AS $$ ... $$;
```

### Validation (in this order)

| # | Check | Error Code |
|---|-------|-----------|
| 1 | Caller is authenticated (auth.uid() not null) | `AUTH_REQUIRED` |
| 2 | p_project_id is not null | `VALIDATION_ERROR` |
| 3 | Project exists | `NOT_FOUND` |
| 4 | Caller is a member of the project's workspace | `FORBIDDEN` |
| 5 | Caller is admin or owner of the workspace | `FORBIDDEN` |
| 6 | Project is not already archived | `BUSINESS_RULE_VIOLATION` |

### Logic (after all validation passes)

1. Update `projects`: set `status = 'archived'`, `archived_at = now()`, `archived_by = auth.uid()`
2. Update `tasks`: set `status = 'on-hold'` where `project_id = p_project_id AND status = 'open'`
3. Insert into `audit_logs`: `action = 'project.archived'`, `entity_type = 'project'`, `entity_id = p_project_id`, `actor_id = auth.uid()`, `notes = p_reason`

### Return Shape (on success)

```json
{
  "data": {
    "id": "uuid",
    "name": "Project Name",
    "status": "archived",
    "archived_at": "2024-01-15T10:00:00Z"
  },
  "message": "Project archived successfully."
}
```

### Error Format

All errors must follow: `'ERROR_CODE: description'`

Examples:
- `'AUTH_REQUIRED: authentication required'`
- `'NOT_FOUND: project not found'`
- `'FORBIDDEN: only admins can archive projects'`
- `'BUSINESS_RULE_VIOLATION: project is already archived'`

---

## What You Should NOT Do

```text
× Split this into 3 separate RPC calls from the frontend
× Use SECURITY DEFINER without documentation and a second reviewer
× Use SELECT * anywhere inside the function
× Skip the auth check (auth.uid() validation)
× Return the error inside the JSON data — use RAISE EXCEPTION
× Use COMMIT or ROLLBACK manually inside the function
× Skip the "already archived" check (idempotency guard)
```

---

## Frontend Call Example

```ts
const { data, error } = await supabase.rpc('archive_project', {
  p_project_id: projectId,
  p_reason: 'Project completed — archiving for record keeping.'
});

if (error) {
  const code = error.message.split(':')[0].trim();
  switch (code) {
    case 'FORBIDDEN':               return showError('Admin access required.');
    case 'NOT_FOUND':               return showError('Project not found.');
    case 'BUSINESS_RULE_VIOLATION': return showError('Project is already archived.');
    default:                        return showError('Something went wrong. Please try again.');
  }
}
```

---

## Checklist to Run

Before marking this done, run: [RPC Function Checklist](../../checklists/supabase/rpc-function.md)

---

## Done When

```text
[ ] Function signature matches the specification exactly
[ ] All 6 validation steps implemented in the correct order
[ ] All 3 business logic steps execute correctly
[ ] All steps are in one transaction (single function — no manual COMMIT)
[ ] Return shape matches the specification
[ ] All errors follow the 'ERROR_CODE: description' format
[ ] EXCEPTION WHEN OTHERS block at the end re-raises as SERVER_ERROR
[ ] Function tested: all success and failure cases pass
[ ] Frontend call example works with the real Supabase client
[ ] Documented using the API Documentation Template
```
