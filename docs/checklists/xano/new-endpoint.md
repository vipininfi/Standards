# Xano — New API Endpoint Checklist

> **Core Rule:** Every Xano endpoint must validate the auth token first, check permissions from the database (never from the request), validate all inputs, return standard error formats, and be documented before frontend handoff.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-before-you-build) | Before You Build |
| [2](#2-endpoint-naming) | Endpoint Naming |
| [3](#3-step-1-auth-token-validation) | Step 1 — Auth Token Validation |
| [4](#4-step-2-permission-check) | Step 2 — Permission Check |
| [5](#5-step-3-input-validation) | Step 3 — Input Validation |
| [6](#6-step-4-business-logic) | Step 4 — Business Logic |
| [7](#7-step-5-response-format) | Step 5 — Response Format |
| [8](#8-reusable-functions) | Reusable Functions |
| [9](#9-documentation-requirements) | Documentation Requirements |
| [10](#10-testing-requirements) | Testing Requirements |
| [11](#11-xano-endpoint-checklist--before-marking-done) | Xano Endpoint Checklist — Before Marking Done |

---

# 1. Before You Build

```text
[ ] What does this endpoint do? (one clear sentence)
[ ] What HTTP method is correct? (GET / POST / PATCH / DELETE)
[ ] Who can call this? (authenticated users / specific role / public)
[ ] What inputs does it take?
[ ] What does it return on success?
[ ] What are the failure cases?
[ ] Does similar logic already exist in another endpoint? (extract to function if yes)
[ ] Does a Xano Function already exist for auth check, permission check, or shared logic?
```

---

# 2. Endpoint Naming

```text
[ ] HTTP method matches the action:
  — GET: reading data
  — POST: creating a new record
  — PATCH: updating an existing record
  — DELETE: removing a record
[ ] URL uses kebab-case: /workspace-members, /project-tasks
[ ] URL uses REST conventions:
  — GET /projects → list all
  — POST /projects → create new
  — GET /projects/:id → get one
  — PATCH /projects/:id → update one
  — DELETE /projects/:id → delete one
[ ] No verbs in URL: NOT /getProjects, NOT /createProject
[ ] Nested resources scoped correctly: /workspaces/:workspace_id/projects
```

---

# 3. Step 1 — Auth Token Validation

This must be the **first step** in every protected endpoint. No logic runs before this.

```text
[ ] Auth token validated using Xano's built-in "Auth Token" middleware or precondition
[ ] If token is missing or invalid → return 401 AUTH_REQUIRED immediately
[ ] User record retrieved from the validated token — not from the request body
[ ] User ID never comes from the request payload — only from the validated token
```

**Xano Setup:**
- Add "Auth Token" as the first precondition (not just a step)
- This automatically halts the endpoint and returns 401 if the token is invalid

---

# 4. Step 2 — Permission Check

Immediately after auth, check that the authenticated user has the right to perform this action.

```text
[ ] User's role read from the database (e.g., workspace_members table)
[ ] Role NOT read from the request body, query params, or token claims
[ ] Workspace membership verified: user is a member of the requested workspace
[ ] Role-based check: if action requires admin, verify role = 'admin' in DB
[ ] If unauthorized → return 403 FORBIDDEN
[ ] Use a reusable Xano Function for the permission check (not inline code)
```

**Example pattern:**

```
1. Auth Token (precondition)
2. Query: workspace_members WHERE workspace_id = input.workspace_id AND user_id = auth.user_id
3. Precondition: record found → else return 403 FORBIDDEN
4. Precondition: record.role IN ['admin', 'owner'] → else return 403 FORBIDDEN
5. Continue with business logic...
```

---

# 5. Step 3 — Input Validation

```text
[ ] Every required field validated as NOT NULL and NOT EMPTY
[ ] String fields: max length validated
[ ] Email fields: email format validated
[ ] Phone fields: E.164 format validated (starts with +, digits only)
[ ] UUID fields: valid UUID format checked before querying
[ ] Enum/status fields: value must be in allowed list
[ ] Numeric fields: range validated (no negative where not allowed)
[ ] Date/time fields: valid format, logical range (end > start where applicable)
[ ] Validation runs BEFORE any database query or business logic
[ ] Field-level errors returned: which field failed and why
```

**Error format for validation:**

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Validation failed.",
  "fields": {
    "name": "Name is required.",
    "email": "Enter a valid email address."
  }
}
```

---

# 6. Step 4 — Business Logic

```text
[ ] All database queries scoped to the authenticated user's workspace — no cross-tenant data
[ ] Uniqueness checks performed before insert (check for duplicate before creating)
[ ] Shared/repeated logic extracted to a Xano Function — not duplicated inline
[ ] No business rule logic in the response step — logic goes in dedicated steps
[ ] Database errors caught and mapped to standard error codes
[ ] No sensitive data (passwords, full tokens, secret keys) in response
```

---

# 7. Step 5 — Response Format

All Xano endpoints must return this consistent format.

## Success (200/201)

```json
{
  "data": { ... },
  "message": "Project created successfully."
}
```

## Error (4xx/5xx)

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Human-readable description."
}
```

## Standard Error Codes

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| `AUTH_REQUIRED` | Not authenticated | 401 |
| `FORBIDDEN` | No permission | 403 |
| `NOT_FOUND` | Resource does not exist | 404 |
| `DUPLICATE` | Already exists | 409 |
| `VALIDATION_ERROR` | Invalid or missing input | 400 |
| `BUSINESS_RULE_VIOLATION` | Logic rule failed | 422 |
| `SERVER_ERROR` | Unexpected error | 500 |

```text
[ ] All success responses include data + message
[ ] All error responses include status code, code, and message
[ ] HTTP status codes are correct (not 200 for every response)
[ ] No internal error details or stack traces in response
```

---

# 8. Reusable Functions

```text
[ ] Auth check is a shared Xano Function (not copy-pasted per endpoint)
[ ] Permission check is a shared Xano Function (check_workspace_permission)
[ ] Slug generation, date formatting, price calculation → dedicated Xano Functions
[ ] Any logic used in 3+ endpoints has been extracted to a function
[ ] Functions are named clearly: verb_noun (check_workspace_permission, generate_slug)
[ ] Functions are grouped: Auth Functions, Utility Functions, Workspace Functions
```

**Reusability check:**

```text
[ ] This endpoint has no steps that are identical to steps in another endpoint
[ ] If duplicated steps found — extracted to a Xano Function before marking done
```

---

# 9. Documentation Requirements

```text
[ ] Endpoint name and description documented
[ ] HTTP method and URL documented
[ ] Auth requirement stated (protected / public)
[ ] Request inputs documented: name, type, required/optional, constraints
[ ] Success response documented with JSON example
[ ] All error cases documented: code, message, example response
[ ] Frontend call example provided (fetch or axios)
[ ] Test cases listed
```

---

# 10. Testing Requirements

```text
[ ] Success case: correct inputs → correct response
[ ] Missing required field → VALIDATION_ERROR returned
[ ] Invalid format (bad email, UUID) → VALIDATION_ERROR returned
[ ] Unauthenticated request → AUTH_REQUIRED (401)
[ ] Wrong workspace (other user's workspace_id) → FORBIDDEN (403)
[ ] Wrong role (member doing admin action) → FORBIDDEN (403)
[ ] Duplicate record → DUPLICATE (409)
[ ] Non-existent ID → NOT_FOUND (404)
[ ] Edge cases specific to this endpoint
[ ] Tested using Xano's built-in test runner
[ ] All response fields verified (no missing fields in success response)
```

---

# 11. Xano Endpoint Checklist — Before Marking Done

```text
NAMING
[ ] HTTP method correct for action
[ ] URL uses kebab-case REST conventions
[ ] No verbs in URL

AUTH (Step 1 — Must Be First)
[ ] Auth Token middleware/precondition at start
[ ] Invalid token returns 401 immediately
[ ] User ID from token — not from request payload

PERMISSIONS (Step 2)
[ ] Role read from database — not from request
[ ] Workspace membership verified
[ ] Role check for admin-only actions
[ ] Unauthorized returns 403 FORBIDDEN
[ ] Uses reusable permission check function

INPUT VALIDATION (Step 3)
[ ] All required fields checked
[ ] String lengths validated
[ ] Format rules validated (email, phone, UUID, enum)
[ ] Numeric ranges validated
[ ] Validation runs BEFORE any business logic
[ ] Field-level errors returned

BUSINESS LOGIC (Step 4)
[ ] All queries scoped to user's workspace
[ ] Uniqueness checked before insert
[ ] Shared logic extracted to Xano Function

RESPONSE FORMAT (Step 5)
[ ] Success: { data, message } — 200/201
[ ] Error: { status, code, message } — correct HTTP status
[ ] Validation: { status, code, message, fields }
[ ] No stack traces in response

REUSABILITY
[ ] No duplicated steps from other endpoints
[ ] Shared logic extracted to function

TESTING
[ ] Success case tested
[ ] All error cases tested
[ ] All roles tested
[ ] Tested in Xano test runner

DOCUMENTATION
[ ] Request/response documented
[ ] All error codes documented
[ ] Frontend call example provided
```

---

## Practice Task

Apply what you learned by building a real Xano endpoint with auth-first, DB permission check, validation, and a reusable function.

**→ [Task 01: Build a Create Project Endpoint](../../tasks/xano/01-create-project-endpoint.md)**

Covers: Auth Token as first precondition, reusable `check_workspace_permission` function, role check from workspace_members table, field-level validation errors, uniqueness check, created_by from token (not from input), standard error format with correct HTTP status codes.
