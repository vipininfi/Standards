# Supabase — New API Endpoint Checklist

> **Core Rule:** No endpoint is complete until auth is checked, input is validated, errors are standardized, it is tested across all roles and failure cases, and the frontend has documentation.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-before-you-build) | Before You Build |
| [2](#2-choosing-the-right-api-type) | Choosing the Right API Type |
| [3](#3-auth--permission-requirements) | Auth & Permission Requirements |
| [4](#4-input-validation) | Input Validation |
| [5](#5-response-format) | Response Format |
| [6](#6-standard-error-codes) | Standard Error Codes |
| [7](#7-rpc-function-checklist) | RPC Function Checklist |
| [8](#8-documentation-requirements) | Documentation Requirements |
| [9](#9-testing-requirements) | Testing Requirements |
| [10](#10-api-endpoint-checklist--before-marking-done) | API Endpoint Checklist — Before Marking Done |

---

# 1. Before You Build

```text
[ ] What does this endpoint do? (stated clearly in one sentence)
[ ] Who calls this? (frontend user role, another service, a webhook)
[ ] What data does it read or write?
[ ] Is this a read (SELECT), write (INSERT/UPDATE), delete (DELETE), or multi-step operation?
[ ] Does this already exist as a direct table query, an RPC, or an Edge Function?
[ ] Does this need a new table, or does it use existing tables?
[ ] What are the failure scenarios? (missing data, unauthorized, duplicate, invalid input)
[ ] What should happen in each failure case?
```

---

# 2. Choosing the Right API Type

| Use This | When |
|----------|------|
| Direct table query (`supabase.from(...)`) | Simple CRUD on one table with RLS |
| RPC / Postgres Function | Multi-step logic, atomic operations, cross-table writes, or complex business logic |
| Edge Function | Webhooks, third-party integrations, email sending, payment processing, logic requiring secrets |

```text
[ ] API type chosen and justified
[ ] Not using Edge Function when a simpler table query or RPC would suffice
[ ] Not using direct table query when logic involves multiple tables (use RPC)
```

---

# 3. Auth & Permission Requirements

```text
[ ] Is this endpoint protected (requires auth) or public?
[ ] If protected:
  — Auth validated at the start (auth.uid() is not null)
  — Role/permission read from database — NEVER from the request payload
  — Workspace membership verified if workspace-scoped
[ ] If public:
  — Confirmed intentionally public
  — No sensitive data exposed
[ ] RLS policies enforce data access — endpoint does not manually filter by user
[ ] Service role key NOT used in this flow (only for server-side admin tasks)
```

---

# 4. Input Validation

Every input must be validated at three layers:

| Layer | What It Does |
|-------|-------------|
| Frontend | UX feedback (required, format hints) |
| API / RPC | Business rule enforcement (length, format, uniqueness) |
| Database | Data integrity (NOT NULL, CHECK constraints, FK constraints) |

## Validation Rules

```text
[ ] Every required field validated as NOT NULL / NOT EMPTY at the API level
[ ] String fields: max length validated (prevent oversized inputs)
[ ] Email fields: format validated
[ ] Phone fields: E.164 format validated if applicable
[ ] Enum/status fields: allowed values validated (not just any string)
[ ] Numeric fields: range validated (no negative where not allowed, no out-of-range values)
[ ] Date/datetime: valid date format, range validated if applicable
[ ] Foreign key IDs: existence validated + access check (user has access to that entity)
[ ] File uploads: validated in Edge Function (size, MIME type) — not just extension
[ ] UUID fields: format validated before querying
```

## What to Return on Validation Error

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Validation failed.",
  "fields": {
    "name": "Name is required.",
    "email": "Enter a valid email address."
  }
}
```

---

# 5. Response Format

## Success Response

```json
{
  "data": { ... },
  "message": "Project created successfully."
}
```

## Error Response

```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable description of the error."
}
```

## Rules

```text
[ ] Success response always includes data (or null if nothing to return) + message
[ ] Error response always includes error code + human-readable message
[ ] HTTP status codes are correct:
  — 200: Success (GET, update)
  — 201: Created (POST)
  — 400: Validation error / bad request
  — 401: Not authenticated
  — 403: Authenticated but not authorized (forbidden)
  — 404: Resource not found
  — 409: Conflict (duplicate, already exists)
  — 422: Unprocessable entity (business rule violation)
  — 500: Server error
[ ] Frontend never has to guess what happened — error code is always included
[ ] No stack traces or internal error details in response (log them, don't return them)
```

---

# 6. Standard Error Codes

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| `AUTH_REQUIRED` | Not logged in | 401 |
| `FORBIDDEN` | Logged in but no permission | 403 |
| `NOT_FOUND` | Resource does not exist | 404 |
| `DUPLICATE` | Already exists (unique constraint) | 409 |
| `VALIDATION_ERROR` | Invalid or missing input | 400 |
| `BUSINESS_RULE_VIOLATION` | Failed a business rule | 422 |
| `SERVER_ERROR` | Unexpected internal error | 500 |

---

# 7. RPC Function Checklist

Only applies when building a Postgres RPC function:

```text
[ ] Function uses SECURITY DEFINER if it needs to bypass RLS for controlled operations
[ ] SECURITY DEFINER functions manually verify auth.uid() at the start
[ ] Function is VOLATILE (default) or STABLE — not incorrectly marked IMMUTABLE
[ ] Function parameters are typed correctly (UUID, TEXT, INT — not generic anyelement)
[ ] Function returns a consistent structure (JSON or typed row)
[ ] Function handles all error paths with RAISE EXCEPTION (not silent failure)
[ ] Transaction wraps multi-step operations (all succeed or all fail)
[ ] Function tested in isolation with valid and invalid inputs
```

## Example RPC Pattern

```sql
CREATE OR REPLACE FUNCTION create_project(
  p_workspace_id UUID,
  p_name TEXT,
  p_description TEXT DEFAULT NULL
)
RETURNS JSON
LANGUAGE plpgsql
SECURITY INVOKER  -- RLS still applies
AS $$
DECLARE
  v_project projects%ROWTYPE;
BEGIN
  -- Validate inputs
  IF p_name IS NULL OR trim(p_name) = '' THEN
    RAISE EXCEPTION 'VALIDATION_ERROR: name is required';
  END IF;

  IF length(trim(p_name)) > 100 THEN
    RAISE EXCEPTION 'VALIDATION_ERROR: name must be 100 characters or fewer';
  END IF;

  -- Check duplicate
  IF EXISTS (
    SELECT 1 FROM projects
    WHERE workspace_id = p_workspace_id AND name = trim(p_name)
  ) THEN
    RAISE EXCEPTION 'DUPLICATE: a project with this name already exists';
  END IF;

  -- Insert (RLS will enforce membership)
  INSERT INTO projects (workspace_id, name, description, created_by)
  VALUES (p_workspace_id, trim(p_name), p_description, auth.uid())
  RETURNING * INTO v_project;

  RETURN json_build_object('data', row_to_json(v_project));
END;
$$;
```

---

# 8. Documentation Requirements

Every API endpoint needs documentation before it is handed off to the frontend team.

```text
[ ] Endpoint name / function name documented
[ ] HTTP method and URL (or RPC call format) documented
[ ] Auth requirement stated (public / authenticated / role required)
[ ] Request payload documented (every field: name, type, required/optional, constraints)
[ ] Success response documented (full JSON example)
[ ] Error responses documented (all possible error codes + example response)
[ ] Frontend code example provided (Supabase client call)
[ ] Test cases listed (at least: success, missing required field, unauthorized, not found, duplicate)
```

---

# 9. Testing Requirements

```text
[ ] Success case: correct data in → correct response out
[ ] Missing required field: correct error returned
[ ] Invalid format (e.g., bad email): validation error returned
[ ] Unauthorized (not logged in): AUTH_REQUIRED returned
[ ] Forbidden (wrong workspace, wrong role): FORBIDDEN returned
[ ] Duplicate record: DUPLICATE error returned
[ ] Not found (wrong ID): NOT_FOUND returned
[ ] Edge cases specific to this endpoint (e.g., end date before start date)
[ ] Tested as each role: anonymous, member, admin, non-member
[ ] Tested with empty strings, null values, oversized strings
[ ] Tested with invalid UUID formats
[ ] RLS prevents cross-workspace data access (confirmed via test)
```

---

# 10. API Endpoint Checklist — Before Marking Done

```text
API TYPE
[ ] Correct type chosen: table query / RPC / Edge Function
[ ] Justified by complexity and requirements

AUTH & PERMISSIONS
[ ] Auth required (or intentionally public and documented)
[ ] Role verified from database — not from request payload
[ ] Workspace membership verified if multi-tenant

INPUT VALIDATION
[ ] Required fields checked
[ ] String lengths validated
[ ] Format rules enforced (email, phone, UUID, enum)
[ ] Numeric ranges validated
[ ] Foreign key IDs checked for existence and access

RESPONSE FORMAT
[ ] Success: { data, message }
[ ] Error: { error, message } with correct HTTP status
[ ] Field-level errors for validation: { error, message, fields }
[ ] No stack traces or internal details in response

ERROR CASES HANDLED
[ ] VALIDATION_ERROR (400)
[ ] AUTH_REQUIRED (401)
[ ] FORBIDDEN (403)
[ ] NOT_FOUND (404)
[ ] DUPLICATE (409)
[ ] SERVER_ERROR (500)

TESTING
[ ] Success case tested
[ ] All error cases tested
[ ] All roles tested (anonymous, member, admin, non-member)
[ ] Cross-workspace access confirmed blocked

DOCUMENTATION
[ ] Request payload documented
[ ] All error codes documented
[ ] Frontend code example provided
[ ] Handed off to frontend team
```

---

## Practice Task

Apply what you learned by building a real multi-step RPC function with atomic operations, validation, and full API documentation.

**→ [Task 03: Build a Create Project API (RPC)](../../tasks/supabase/03-create-project-api.md)**

Covers: RPC function with SECURITY INVOKER, six validation steps, workspace membership check in SQL, atomic insert of project + task list + audit log, RAISE EXCEPTION with standard error codes, full API documentation using the template.
