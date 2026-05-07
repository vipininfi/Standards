# Supabase — RPC & Postgres Function Standards

> **Core Rule:** An RPC function is not just an API convenience — it is a contract. It must validate its inputs, verify the caller's identity, handle errors explicitly, wrap multi-step operations in a transaction, and return a consistent, predictable response.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-when-to-use-an-rpc-function) | When to Use an RPC Function |
| [2](#2-security-invoker-vs-security-definer) | SECURITY INVOKER vs SECURITY DEFINER |
| [3](#3-function-signature-standards) | Function Signature Standards |
| [4](#4-return-type-standards) | Return Type Standards |
| [5](#5-validation-inside-rpc) | Validation Inside RPC |
| [6](#6-error-raising-standard) | Error Raising Standard |
| [7](#7-transaction-handling) | Transaction Handling |
| [8](#8-auth--permission-pattern) | Auth & Permission Pattern |
| [9](#9-naming-conventions) | Naming Conventions |
| [10](#10-performance-rules) | Performance Rules |
| [11](#11-testing-an-rpc-function) | Testing an RPC Function |
| [12](#12-documentation-requirement) | Documentation Requirement |

---

# 1. When to Use an RPC Function

| Use RPC When | Use Direct Table Query When | Use Edge Function When |
|-------------|----------------------------|----------------------|
| Multi-step operation that must be atomic (all succeed or all fail) | Single-table CRUD with RLS | Logic requires a secret key |
| Business logic that cannot be enforced by RLS alone | Simple read with filter | Calling a third-party API |
| Cross-table writes in one transaction | No business logic — just data | Sending email or SMS |
| Uniqueness checks before insert | Standard CRUD for one entity | Handling a webhook payload |
| Computed return values involving joins | Data is already secured by RLS | Long-running background jobs |

---

# 2. SECURITY INVOKER vs SECURITY DEFINER

This is the most critical decision when writing an RPC function.

| Mode | What It Does | When to Use |
|------|-------------|-------------|
| `SECURITY INVOKER` | Function runs as the **calling user** — RLS policies still apply | Default. Use for most business operations. |
| `SECURITY DEFINER` | Function runs as the **function owner** (usually postgres) — RLS is bypassed | Only when the function intentionally needs to bypass RLS for a controlled operation (e.g., creating a workspace member record that the user cannot directly insert due to RLS). |

```sql
-- Default — RLS applies
CREATE FUNCTION my_function(...)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER  -- explicit is better than implicit
AS $$ ... $$;

-- Bypass RLS — use with extreme care
CREATE FUNCTION create_workspace_with_owner(...)
RETURNS json
LANGUAGE plpgsql
SECURITY DEFINER  -- document WHY this is needed
AS $$
DECLARE
  v_user_id uuid := auth.uid();  -- ALWAYS capture auth.uid() immediately in SECURITY DEFINER
BEGIN
  -- Must verify caller identity manually — RLS won't do it
  IF v_user_id IS NULL THEN
    RAISE EXCEPTION 'AUTH_REQUIRED: authentication required';
  END IF;
  -- rest of logic
END;
$$;
```

### SECURITY DEFINER Rules

```text
[ ] SECURITY DEFINER is only used when explicitly documented and justified
[ ] Every SECURITY DEFINER function captures auth.uid() as the FIRST line and validates it is not null
[ ] Every SECURITY DEFINER function performs manual permission checks (no relying on RLS)
[ ] SECURITY DEFINER functions are reviewed by a second developer before merging
[ ] The reason for SECURITY DEFINER is documented in a comment above the function
```

---

# 3. Function Signature Standards

```sql
CREATE OR REPLACE FUNCTION function_name(
  p_workspace_id  uuid,           -- prefix parameters with p_
  p_name          text,
  p_description   text DEFAULT NULL,  -- optional parameters get a default
  p_start_date    date DEFAULT NULL
)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER
AS $$
...
$$;
```

### Parameter Naming

```text
[ ] All parameters prefixed with p_: p_workspace_id, p_name, p_user_id
[ ] Avoids collision with column names inside the function body
[ ] Internal variables prefixed with v_: v_project, v_count, v_user_id
[ ] No single-letter variable names
[ ] Optional parameters use DEFAULT NULL (or appropriate default)
```

### Parameter Types

```text
[ ] UUID parameters typed as uuid — not text
[ ] Timestamp parameters typed as timestamptz — not timestamp (timezone-aware)
[ ] Date parameters typed as date — not text
[ ] Boolean parameters typed as boolean — not text or integer
[ ] Numeric parameters typed as numeric or integer — not text
[ ] Text parameters typed as text — not varchar(n) (use constraints, not types, for length)
[ ] Arrays typed correctly: text[], uuid[], integer[]
```

---

# 4. Return Type Standards

Choose one return type and use it consistently:

| Return Type | Use When | Example |
|-------------|----------|---------|
| `RETURNS json` | Returning a single record or mixed data | `{ "data": {...} }` |
| `RETURNS jsonb` | Same as json but binary — prefer for large objects | Same shape |
| `RETURNS TABLE (col1 type, col2 type)` | Returning multiple rows with known columns | Set of typed rows |
| `RETURNS SETOF table_name` | Returning rows from a known table | Full table rows |
| `RETURNS void` | Side-effect only — no data returned | Audit log inserts |
| `RETURNS boolean` | Simple success/failure flag | Rarely used — prefer json |

### Standard JSON Return Shape

```sql
-- Success: single record
RETURN json_build_object(
  'data', row_to_json(v_record),
  'message', 'Project created successfully.'
);

-- Success: list
RETURN json_build_object(
  'data', json_agg(row_to_json(r)),
  'count', v_count
);

-- Caller handles errors via EXCEPTION block — don't return error in data
```

---

# 5. Validation Inside RPC

Validate in this order. Raise an exception at the first failure — do not collect all errors:

```sql
-- 1. Required field check
IF p_name IS NULL OR trim(p_name) = '' THEN
  RAISE EXCEPTION 'VALIDATION_ERROR: name is required';
END IF;

-- 2. Length check
IF length(trim(p_name)) > 100 THEN
  RAISE EXCEPTION 'VALIDATION_ERROR: name must be 100 characters or fewer';
END IF;

-- 3. Format check (enum)
IF p_status NOT IN ('active', 'on-hold', 'archived') THEN
  RAISE EXCEPTION 'VALIDATION_ERROR: status must be active, on-hold, or archived';
END IF;

-- 4. Cross-field check
IF p_end_date IS NOT NULL AND p_start_date IS NOT NULL AND p_end_date <= p_start_date THEN
  RAISE EXCEPTION 'VALIDATION_ERROR: end_date must be after start_date';
END IF;

-- 5. Existence check (FK validation)
IF NOT EXISTS (SELECT 1 FROM workspaces WHERE id = p_workspace_id) THEN
  RAISE EXCEPTION 'NOT_FOUND: workspace not found';
END IF;

-- 6. Permission check (after existence, before business logic)
IF NOT EXISTS (
  SELECT 1 FROM workspace_members
  WHERE workspace_id = p_workspace_id AND user_id = auth.uid()
) THEN
  RAISE EXCEPTION 'FORBIDDEN: you are not a member of this workspace';
END IF;

-- 7. Business rule check (uniqueness, limits, etc.)
IF EXISTS (
  SELECT 1 FROM projects
  WHERE workspace_id = p_workspace_id AND name = trim(p_name) AND deleted_at IS NULL
) THEN
  RAISE EXCEPTION 'DUPLICATE: a project with this name already exists';
END IF;
```

---

# 6. Error Raising Standard

All RAISE EXCEPTION messages must follow the format `ERROR_CODE: description` so the frontend can parse the error code reliably.

```sql
-- Format: 'ERROR_CODE: human readable description'
RAISE EXCEPTION 'VALIDATION_ERROR: name is required';
RAISE EXCEPTION 'FORBIDDEN: only admins can delete projects';
RAISE EXCEPTION 'NOT_FOUND: project not found';
RAISE EXCEPTION 'DUPLICATE: a project with this name already exists';
RAISE EXCEPTION 'BUSINESS_RULE_VIOLATION: cannot archive a project with open tasks';
RAISE EXCEPTION 'SERVER_ERROR: unexpected error occurred';
```

### Frontend Parsing Pattern

The frontend Supabase client receives the message in `error.message`. Parse the error code:

```ts
const { data, error } = await supabase.rpc('create_project', params);

if (error) {
  const code = error.message.split(':')[0].trim(); // 'VALIDATION_ERROR'
  const msg  = error.message.split(':').slice(1).join(':').trim(); // 'name is required'

  switch (code) {
    case 'VALIDATION_ERROR':   return showFieldError(msg);
    case 'DUPLICATE':          return showError('A project with this name already exists.');
    case 'FORBIDDEN':          return showError('You do not have permission.');
    case 'NOT_FOUND':          return showError('Item not found. It may have been deleted.');
    default:                   return showError('Something went wrong. Please try again.');
  }
}
```

---

# 7. Transaction Handling

PL/pgSQL functions run inside a transaction by default. Multi-step operations are automatically atomic — if any step raises an exception, all previous steps are rolled back.

```sql
CREATE OR REPLACE FUNCTION create_project_with_defaults(
  p_workspace_id uuid,
  p_name text
)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER
AS $$
DECLARE
  v_project   projects%ROWTYPE;
  v_task_list task_lists%ROWTYPE;
BEGIN
  -- All three inserts are in one implicit transaction
  -- If any RAISE EXCEPTION fires, ALL are rolled back automatically

  INSERT INTO projects (workspace_id, name, created_by)
  VALUES (p_workspace_id, trim(p_name), auth.uid())
  RETURNING * INTO v_project;

  INSERT INTO task_lists (project_id, name, created_by)
  VALUES (v_project.id, 'General', auth.uid())
  RETURNING * INTO v_task_list;

  INSERT INTO audit_logs (action, entity_type, entity_id, actor_id)
  VALUES ('project.created', 'project', v_project.id, auth.uid());

  RETURN json_build_object('data', row_to_json(v_project));
END;
$$;
```

### Transaction Rules

```text
[ ] Multi-step operations (create + related records + audit log) always in one function
[ ] Never split an atomic operation across multiple frontend calls
[ ] Never use COMMIT or ROLLBACK manually inside a function — Postgres handles this
[ ] If a step might fail for an expected reason: use RAISE EXCEPTION with a clear error code
[ ] If catching unexpected errors: use a EXCEPTION WHEN OTHERS block at the end
[ ] EXCEPTION WHEN OTHERS: log the error and re-raise with SERVER_ERROR code
```

---

# 8. Auth & Permission Pattern

```sql
DECLARE
  v_user_id    uuid := auth.uid();          -- capture once at the top
  v_member     workspace_members%ROWTYPE;
BEGIN
  -- Step 1: Require authentication
  IF v_user_id IS NULL THEN
    RAISE EXCEPTION 'AUTH_REQUIRED: authentication required';
  END IF;

  -- Step 2: Check membership and role
  SELECT * INTO v_member
  FROM workspace_members
  WHERE workspace_id = p_workspace_id
    AND user_id = v_user_id;

  IF NOT FOUND THEN
    RAISE EXCEPTION 'FORBIDDEN: you are not a member of this workspace';
  END IF;

  -- Step 3: Check role if admin-only action
  IF v_member.role NOT IN ('admin', 'owner') THEN
    RAISE EXCEPTION 'FORBIDDEN: only admins can perform this action';
  END IF;

  -- Business logic continues...
END;
```

---

# 9. Naming Conventions

```text
[ ] Function names: snake_case verb_noun pattern
  — create_project, update_workspace_name, delete_member, get_project_stats
[ ] Parameter names: p_ prefix + snake_case: p_workspace_id, p_project_name
[ ] Internal variables: v_ prefix + snake_case: v_project, v_member_role, v_count
[ ] Avoid generic names: not p_data, not p_input, not v_result
[ ] Avoid abbreviations: p_workspace_id not p_ws_id, v_project not v_proj
```

---

# 10. Performance Rules

```text
[ ] Avoid SELECT * inside functions — select only the columns needed
[ ] Use FOUND after a query with INTO to check if a row was returned
[ ] Avoid N+1 queries inside a loop — use a single query with a JOIN or IN clause
[ ] Large result sets: use LIMIT to cap — do not return unbounded rows
[ ] Indexes exist for all WHERE clause columns used in the function's queries
[ ] Avoid calling other expensive functions inside a tight loop
[ ] For read-only functions: mark as STABLE (helps query planner optimize)
  — STABLE: safe to call multiple times with same result in same transaction
  — VOLATILE (default): can have side effects — use for write operations
```

---

# 11. Testing an RPC Function

```text
[ ] Success case: correct inputs → correct data returned, all side effects applied
[ ] Missing required field: VALIDATION_ERROR raised with correct field name
[ ] Invalid format: VALIDATION_ERROR raised
[ ] Cross-field failure: VALIDATION_ERROR raised
[ ] Non-existent FK: NOT_FOUND raised
[ ] Non-member caller: FORBIDDEN raised
[ ] Wrong role: FORBIDDEN raised
[ ] Duplicate record: DUPLICATE raised
[ ] All side effects verified: related records created, audit log written
[ ] Transaction atomicity: simulate a failure in step 3 — verify steps 1 and 2 were rolled back
[ ] RLS still applies (for SECURITY INVOKER): cross-workspace access blocked
```

---

# 12. Documentation Requirement

Every RPC function must be documented before frontend handoff using the [API Documentation Template](03-api-documentation.md):

```text
[ ] Function name and description documented
[ ] All parameters documented: name, type, required/optional, constraints
[ ] Success response documented with JSON example
[ ] All RAISE EXCEPTION codes documented with example frontend handling
[ ] Frontend call example: supabase.rpc('function_name', { p_param: value })
[ ] Test cases listed
```
