# Xano Standards

Core rule:

> Xano is the backend. All business logic, permission checks, validation, and data integrity rules live in Xano — never in WeWeb, React, or any frontend. The frontend only calls Xano APIs and renders the result.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-xano-developer-responsibilities) | Developer Responsibilities |
| [2](#2-api-group-workspace-organization) | API Group Organization |
| [3](#3-endpoint-naming-standard) | Endpoint Naming Standard |
| [4](#4-authentication-standard) | Authentication — First Step |
| [5](#5-permission-check-standard) | Permission Check |
| [6](#6-input-validation-standard) | Input Validation |
| [7](#7-error-response-standard) | Error Response Format |
| [8](#8-reusable-functions-standard) | Reusable Functions |
| [9](#9-database-table--field-naming-standard) | Database Naming |
| [10](#10-webhook-endpoint-standard) | Webhooks |
| [11](#11-environment-variables-standard) | Environment Variables |
| [12](#12-api-documentation-standard) | API Documentation |
| [13](#13-xano--definition-of-done) | Definition of Done |

---

# 1. Xano Developer Responsibilities

| Area | Responsibility |
|------|---------------|
| API Design | Every endpoint is purposeful, named clearly, and documented. |
| Authentication | Every protected endpoint validates the auth token as the first step. |
| Validation | Every input is validated in Xano — do not trust frontend validation. |
| Business Logic | All rules, calculations, and workflows live in Xano. |
| Error Handling | Every endpoint returns a consistent, predictable error format. |
| Permissions | Role/ownership checks happen in Xano before any data operation. |
| Reusable Functions | Shared logic lives in Xano Functions, not duplicated across endpoints. |
| Documentation | Every API is documented before frontend integration starts. |

---

# 2. API Group (Workspace) Organization

Organize Xano APIs into logical groups by feature or app area.

## Good Group Structure

```text
Auth
  POST /auth/login
  POST /auth/signup
  POST /auth/forgot-password
  POST /auth/reset-password
  GET  /auth/me

Projects
  GET    /projects
  POST   /projects
  GET    /projects/:id
  PATCH  /projects/:id
  DELETE /projects/:id

Workspace Members
  GET    /workspace/:id/members
  POST   /workspace/:id/members/invite
  DELETE /workspace/:id/members/:member_id

Files
  POST   /files/upload
  GET    /files/:id
  DELETE /files/:id

Webhooks
  POST /webhooks/stripe
  POST /webhooks/sendgrid
```

## Bad Structure

```text
Endpoints
  POST /doStuff
  POST /handleProject
  GET /getData
  POST /misc
```

---

# 3. Endpoint Naming Standard

## Rules

| Rule | Standard |
|------|----------|
| Format | `kebab-case` |
| Pattern | `/resource` or `/resource/:id` |
| Action naming | Use HTTP method, not verb in URL |
| Nested resources | `/parent/:id/child` |
| Webhooks | `/webhooks/provider-name` |

## Good vs Bad

| Good | Bad |
|------|-----|
| `GET /projects` | `GET /getProjects` |
| `POST /projects` | `POST /createProject` |
| `PATCH /projects/:id` | `POST /updateProject` |
| `DELETE /projects/:id` | `POST /deleteProjectById` |
| `POST /workspace/:id/members/invite` | `POST /inviteMember` |

## Exception

For complex operations that don't map cleanly to CRUD, use an action noun:

```text
POST /projects/:id/archive
POST /invitations/:id/accept
POST /invitations/:id/decline
POST /users/:id/suspend
```

---

# 4. Authentication Standard

## Auth Token Validation — First Step in Every Protected Endpoint

Every endpoint that requires a logged-in user must validate the token **before** doing anything else.

### In Xano

```
Step 1: Auth Token (built-in middleware)
  → If invalid: stop, return 401
  → If valid: continue, authUser variable is set

Step 2: Permission check (if role required)
  → Query user role from database
  → If not allowed: stop, return 403

Step 3: Business logic
```

**Never skip the auth token step on protected endpoints.**

## Auth Requirements By Endpoint Type

| Endpoint Type | Auth Required | Role Check |
|---------------|--------------|------------|
| Public API (e.g. contact form) | No | No |
| User-specific data | Yes | Owner check |
| Workspace operation | Yes | Workspace membership + role |
| Admin operation | Yes | Admin role from database |
| Webhook | No auth token, but signature verification | — |

---

# 5. Permission Check Standard

After auth token validation, check what the user is allowed to do.

## Pattern

```text
1. Get auth user ID from token.
2. Query workspace_members where user_id = auth.id AND workspace_id = input.workspace_id.
3. If no record found: return 403 FORBIDDEN.
4. Check role is allowed for this action.
5. If role not allowed: return 403 FORBIDDEN.
6. Proceed with operation.
```

## Role Hierarchy

Define roles in a consistent order in every permission check:

```text
owner > admin > manager > member > viewer
```

Document which roles can perform which actions:

| Action | Owner | Admin | Manager | Member | Viewer |
|--------|-------|-------|---------|--------|--------|
| View records | Yes | Yes | Yes | Yes | Yes |
| Create record | Yes | Yes | Yes | No | No |
| Edit record | Yes | Yes | Yes | Own only | No |
| Delete record | Yes | No | No | No | No |
| Manage members | Yes | Yes | No | No | No |

---

# 6. Input Validation Standard

Every endpoint must validate every input — even if the frontend also validates.

## Validation Order in Xano

```text
1. Check required fields are present (not null, not empty string).
2. Check field types (text, integer, email, UUID format).
3. Check field lengths (min/max characters, min/max numbers).
4. Check field values (valid enum option, valid date range, valid UUID that exists in DB).
5. Check business rules (workspace ownership, uniqueness, status allowed).
```

## Required Fields Checklist

```text
[ ] All required inputs are checked for null/empty.
[ ] UUID fields are verified to exist in the database and be accessible by the user.
[ ] Email fields are validated for format.
[ ] Phone fields validated for format.
[ ] Enum fields validated against allowed values.
[ ] Text fields have min/max length enforced.
[ ] Number fields have min/max enforced where relevant.
[ ] File fields have type and size validation.
```

## Validation Error Response

When validation fails, return structured errors:

```json
{
  "status": 422,
  "code": "VALIDATION_ERROR",
  "message": "Please fix the highlighted fields.",
  "fields": {
    "name": "Project name is required.",
    "workspace_id": "Invalid workspace."
  }
}
```

---

# 7. Error Response Standard

Every endpoint must return a consistent error format.

## Standard Error Format

```json
{
  "status": 400,
  "code": "ERROR_CODE",
  "message": "Human-readable message for frontend to display."
}
```

## Error Codes

| Code | HTTP Status | When to Use |
|------|-------------|-------------|
| `UNAUTHENTICATED` | 401 | Token missing, expired, or invalid. |
| `FORBIDDEN` | 403 | Authenticated but lacks permission. |
| `VALIDATION_ERROR` | 422 | Input is missing or invalid. |
| `NOT_FOUND` | 404 | Record does not exist or user cannot access it. |
| `CONFLICT` | 409 | Duplicate data or conflicting state. |
| `RATE_LIMITED` | 429 | Too many requests. |
| `INTERNAL_ERROR` | 500 | Unexpected server error. |
| `EXTERNAL_SERVICE_ERROR` | 502 | Third-party call failed (Stripe, email, etc.). |

## In Xano

Use a **Precondition** or **Custom Response** block to return errors:

```text
Precondition: Check field is not null
  → Fail: return 422 with code VALIDATION_ERROR and message "Field is required."
```

Return errors using the Create Variable + Return Response pattern with the standard structure above.

---

# 8. Reusable Functions Standard

If the same logic is used in 2 or more endpoints, it must be extracted to a Xano Function.

## When to Create a Reusable Function

| Scenario | Action |
|----------|--------|
| Auth token check pattern | Built-in middleware (always use) |
| Permission/role check used in multiple APIs | Create Xano Function: `check_workspace_permission` |
| Creating a slug from a name | Create Xano Function: `generate_slug` |
| Sending an email via external API | Create Xano Function: `send_email` |
| Uploading a file to storage | Create Xano Function: `upload_file` |
| Formatting a date for display | Create Xano Function: `format_date` |
| Calculating a value used in multiple places | Create Xano Function: `calculate_[name]` |

## Naming Standard for Xano Functions

```text
verb_noun
```

| Good | Bad |
|------|-----|
| `check_workspace_permission` | `permissionCheck` |
| `send_email_notification` | `emailHelper` |
| `generate_slug` | `slugMaker` |
| `calculate_invoice_total` | `doCalc` |
| `validate_phone_number` | `phoneValidation` |

## Function Group Organization

Keep functions in organized groups:

```text
Auth Functions
  validate_token
  get_auth_user

Permission Functions
  check_workspace_permission
  check_record_ownership

Notification Functions
  send_email_notification
  send_sms_notification

Utility Functions
  generate_slug
  format_date
  calculate_discount
```

---

# 9. Database Table & Field Naming Standard

Follow the same naming conventions as Supabase.

## Table Names

```text
snake_case
plural
```

| Good | Bad |
|------|-----|
| `projects` | `Project` |
| `project_members` | `projectMember` |
| `workspace_invitations` | `inviteData` |

## Field Names

| Purpose | Standard |
|---------|----------|
| Primary key | `id` (integer auto-increment or UUID) |
| Timestamps | `created_at`, `updated_at` |
| Foreign key | `user_id`, `project_id`, `workspace_id` |
| Boolean | `is_active`, `is_verified`, `has_access` |
| Status | `status` with defined allowed values |
| Soft delete | `deleted_at` (nullable) |

## Every Table Should Have

```text
id          — Primary key
created_at  — Auto timestamp on create
updated_at  — Auto timestamp on update
```

For user/workspace-scoped tables:

```text
created_by_user_id   — Who created the record
workspace_id         — Which workspace owns it
```

---

# 10. Webhook Endpoint Standard

For webhooks from Stripe, SendGrid, Zoho, Smartlead, or any third party.

## Webhook Requirements

```text
[ ] Signature verification (if provider supports it — Stripe, SendGrid, etc.)
[ ] Idempotency check — store event ID, never process same event twice
[ ] Parse and validate webhook payload
[ ] Process business logic
[ ] Return 200 immediately to acknowledge (even if async processing)
[ ] Log the event (event type, status, timestamp)
[ ] Handle failure gracefully (do not return 500 unless truly broken)
```

## Webhook Error Handling

If something fails during processing:
- Return `200` to the provider (acknowledge receipt)
- Log the failure internally
- Add to a retry queue if needed

Returning `500` to a webhook provider may cause it to retry repeatedly and flood your system.

---

# 11. Environment Variables Standard

| Variable | Usage |
|----------|-------|
| Database credentials | Always use Xano environment settings — never hardcode |
| External API keys (Stripe, SendGrid, OpenAI) | Store in Xano Environment Variables |
| Webhook secrets | Store in Xano Environment Variables |
| Supabase service role key (if used in Xano) | Store in Xano Environment Variables |
| JWT secret | Managed by Xano — do not expose |

**Never hardcode API keys, passwords, or secrets inside Xano function logic.**

---

# 12. API Documentation Standard

Every Xano API must be documented before frontend integration.

Use the [API Documentation Template](../templates/api-doc-template.md).

At minimum, document:

```text
[ ] API name and purpose
[ ] HTTP method and endpoint path
[ ] Auth required: Yes / No
[ ] Required user role
[ ] Request body fields (name, type, required, validation rules)
[ ] Success response with example JSON
[ ] All error responses (401, 403, 422, 404, 409, 500)
[ ] Frontend implementation notes
[ ] Sample frontend fetch/axios call
[ ] Test cases
```

---

# 13. Xano — Definition of Done

An API built in Xano is complete only when:

```text
[ ] Auth token validation added (if protected)
[ ] Permission/role check added (if role-restricted)
[ ] All inputs validated (required, type, length, enum, ownership)
[ ] Error responses use the standard format
[ ] Reusable logic extracted to Xano Functions (not duplicated)
[ ] Database table/field naming follows snake_case convention
[ ] Webhook endpoints have signature verification and idempotency (if webhook)
[ ] External API keys stored in environment variables (not hardcoded)
[ ] API documented before frontend integration
[ ] Test cases run: success + auth failure + permission failure + validation failure
[ ] Staging tested before marking ready
```
