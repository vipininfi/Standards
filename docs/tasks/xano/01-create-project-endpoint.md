# Task: Build a Create Project Endpoint in Xano

**Platform:** Xano
**Covers:** [New Endpoint Checklist](../../checklists/xano/new-endpoint.md) · [Xano Standards](../../standards/xano-standards.md)

---

## Scenario

You are building the project creation endpoint for **WorkFlow** in Xano. Workspace members can create projects. The endpoint must follow the auth-first pattern — validating the token before any other logic runs — and use the database to check the caller's role, never trusting the request payload.

---

## What to Build

A single Xano API endpoint:

```
POST /projects
Authorization: Bearer <auth_token>
```

---

## Endpoint Specification

### Request Body

```json
{
  "workspace_id": "integer or uuid",
  "name": "My Project",
  "description": "Optional project description.",
  "start_date": "2024-02-01",
  "end_date": "2024-06-30"
}
```

### Steps (in this exact order)

#### Step 1 — Auth Token Validation
Add the **Auth Token** middleware as a precondition (not a function step). This must be the first precondition. If the token is missing or invalid, Xano automatically returns 401 before any other step runs.

#### Step 2 — Permission Check (via Reusable Function)
Call a reusable Xano Function: `check_workspace_permission`

This function should:
- Accept `workspace_id` and `user_id` (from the auth token) as inputs
- Query the `workspace_members` table: find a record where `workspace_id = input.workspace_id AND user_id = auth_user.id`
- Return the membership record (including `role`)
- If no record found → raise error: `FORBIDDEN: you are not a member of this workspace`
- If role is `viewer` → raise error: `FORBIDDEN: viewers cannot create projects`

> If this function does not exist yet — create it. It will be reused across many endpoints.

#### Step 3 — Input Validation

Validate in this order, returning field-level errors:

| Field | Rule | Error |
|-------|------|-------|
| `workspace_id` | Required, must exist | "workspace_id is required." |
| `name` | Required, not empty after trim, min 3 chars, max 100 chars | "name is required." / "name must be at least 3 characters." / "name must be 100 characters or fewer." |
| `description` | Optional, max 500 chars if provided | "description must be 500 characters or fewer." |
| `start_date` | Optional, must be a valid date if provided | "start_date must be a valid date." |
| `end_date` | Optional, if provided and start_date also provided, must be after start_date | "end_date must be after start_date." |

Return validation errors in this format:

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Validation failed.",
  "fields": {
    "name": "name is required."
  }
}
```

#### Step 4 — Uniqueness Check

Query the `projects` table:
- WHERE `workspace_id = input.workspace_id AND name = trimmed_name AND deleted_at IS NULL`
- If a record exists → return:

```json
{
  "status": 409,
  "code": "DUPLICATE",
  "message": "A project with this name already exists in this workspace."
}
```

#### Step 5 — Create the Project

Insert into the `projects` table with:
- `workspace_id` from input
- `name` from input (trimmed)
- `description` from input (or null)
- `status` = `"active"` (hardcoded — not from input)
- `start_date`, `end_date` from input (or null)
- `created_by` = `auth_user.id` (from the auth token — never from input)

#### Step 6 — Return Success

```json
{
  "data": {
    "id": 123,
    "workspace_id": 1,
    "name": "My Project",
    "description": null,
    "status": "active",
    "start_date": "2024-02-01",
    "end_date": "2024-06-30",
    "created_by": 42,
    "created_at": "2024-01-15T10:00:00Z"
  },
  "message": "Project created successfully."
}
```

---

## Reusable Function: `check_workspace_permission`

If this function does not exist, create it in the **Auth Functions** group:

**Inputs:**
- `workspace_id` (integer/uuid)
- `user_id` (integer/uuid)
- `required_roles` (list of text) — optional, defaults to all roles except viewer

**Steps:**
1. Query `workspace_members` WHERE `workspace_id = input.workspace_id AND user_id = input.user_id`
2. Precondition: record found → else raise `FORBIDDEN: not a member`
3. If `required_roles` provided: precondition record.role IN required_roles → else raise `FORBIDDEN: insufficient role`
4. Return: membership record (including role)

---

## What You Should NOT Do

- Do not accept `created_by` from the request body — set it from `auth_user.id`
- Do not accept `status` from the request body — always default to `"active"`
- Do not skip the permission check — any authenticated user could create projects in any workspace
- Do not duplicate the permission check logic inline — use the reusable `check_workspace_permission` function
- Do not return 200 for error responses — use 400, 403, 409 etc.
- Do not run business logic before validation completes

---

## Checklist to Run When Done

Use the [New Endpoint Checklist](../../checklists/xano/new-endpoint.md#11-xano-endpoint-checklist--before-marking-done).

---

## Done When

```text
STRUCTURE
[ ] Endpoint: POST /projects
[ ] Auth Token middleware as first precondition
[ ] Steps in correct order: auth → permission → validation → uniqueness → insert → response

AUTH
[ ] Auth token as first step (precondition, not function step)
[ ] Invalid/missing token returns 401 automatically

PERMISSION CHECK
[ ] check_workspace_permission function exists (created if needed)
[ ] Function checks workspace membership in DB — not from request
[ ] Viewer role blocked
[ ] Wrong workspace returns 403

VALIDATION
[ ] workspace_id required
[ ] name required, trimmed, min 3, max 100
[ ] description optional, max 500
[ ] dates optional, format validated, end > start if both set
[ ] Field-level errors returned

UNIQUENESS
[ ] Duplicate name check before insert
[ ] Returns 409 DUPLICATE if found

INSERT
[ ] created_by = auth_user.id (not from input)
[ ] status = 'active' (not from input)
[ ] name trimmed before insert

RESPONSE
[ ] Success: { data: {...}, message: "..." } — 200
[ ] Errors: { status, code, message } with correct HTTP status codes

REUSABILITY
[ ] check_workspace_permission is a separate reusable function
[ ] Function is in the Auth Functions group
[ ] Function will be usable by other endpoints

TESTING (in Xano test runner)
[ ] Success case: project created, correct response
[ ] Missing name → VALIDATION_ERROR
[ ] Non-member workspace → FORBIDDEN
[ ] Viewer role → FORBIDDEN
[ ] Duplicate name → DUPLICATE
[ ] Unauthenticated → 401
[ ] end_date before start_date → VALIDATION_ERROR
```
