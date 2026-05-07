# API Documentation Template

Use this for every backend API. Backend must complete this before frontend starts integration.

**Rule:** Frontend must never guess API behavior. Every field, error, and edge case must be documented.

**How to use:** Copy everything below the line, paste into a new file named `api-[name].md`, and replace all `[placeholders]`.

---
---

# API: [API Name]

## Status
`Draft` / `Ready for Integration` / `Testing` / `Deprecated`

## Owner
[Backend Developer Name]

## Last Updated
[YYYY-MM-DD]

---

## 1. Purpose

[One sentence — what this API does and why it exists.]

---

## 2. API Type

[Choose one]
- Direct Supabase Table Insert / Select / Update / Delete
- Supabase RPC Function
- Supabase Edge Function
- Xano Endpoint
- Django REST Endpoint

---

## 3. Endpoint / Table / Function

```
Table:    public.[table_name]
Function: [function_name]
Endpoint: POST /functions/v1/[function-name]
```

---

## 4. Auth Required

**Yes** / No

If yes — frontend must pass the logged-in user's access token:

```
Authorization: Bearer <access_token>
```

---

## 5. Permission Required

| Who Can Call | Allowed? |
|-------------|----------|
| Owner | Yes |
| Admin | Yes |
| Manager | [Yes / No] |
| Member | [Yes / No] |
| Viewer | No |
| Anonymous | No |

---

## 6. Request Payload

```json
{
  "field_name": "string",
  "field_name": "uuid",
  "field_name": "string (optional)"
}
```

---

## 7. Required Fields

| Field | Type | Required | Validation Rules |
|-------|------|----------|-----------------|
| `field_name` | `string` | Yes | [e.g. 2–100 characters] |
| `field_name` | `uuid` | Yes | Must be accessible by the calling user |
| `field_name` | `string` | No | Max [N] characters |

---

## 8. Success Response

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "field_name": "value"
  },
  "message": "[Human-readable success message]"
}
```

---

## 9. Error Responses

### 401 — UNAUTHENTICATED

**When:** User is not logged in. Token is missing or expired.

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHENTICATED",
    "message": "Your session has expired. Please log in again."
  }
}
```

**Frontend action:** Redirect to login. Clear local auth state if needed.

---

### 403 — FORBIDDEN

**When:** User is authenticated but does not have the required role or workspace access.

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to perform this action."
  }
}
```

**Frontend action:** Show permission error. Hide or disable the action for unauthorized roles.

---

### 422 — VALIDATION_ERROR

**When:** A required field is missing or a field does not meet validation rules.

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please fix the highlighted fields.",
    "fields": {
      "field_name": "Error message for this specific field."
    }
  }
}
```

**Frontend action:** Show field-level error messages next to each invalid field.

---

### 409 — CONFLICT

**When:** [Describe when a conflict occurs — e.g., a record with this name already exists in the workspace.]

```json
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "[Human-readable conflict message]"
  }
}
```

**Frontend action:** Show the conflict message near the relevant field.

---

### 404 — NOT_FOUND

**When:** The requested record does not exist or the user does not have access to it.

```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "The requested resource was not found."
  }
}
```

**Frontend action:** Show a not-found state or redirect.

---

### 500 — INTERNAL_ERROR

**When:** Unexpected server or database error.

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Something went wrong. Please try again."
  }
}
```

**Frontend action:** Show retry option. Do not clear form data.

---

## 10. RLS / Permission Policies Involved

- [List the RLS policies or permission checks that affect this API]
- [e.g. "workspace_members: only members of the workspace can insert"]

---

## 11. Frontend Implementation Notes

- [What frontend must do before calling — e.g., must be logged in, must have workspace_id]
- [Fields frontend should NOT send — e.g., `created_by` is set server-side from auth token]
- [Loading state expected — disable button, show "Saving..."]
- [On success — redirect to X / refresh list / show notification]
- [On 401 — redirect to login]
- [On 403 — show permission message, hide the action button]

---

## 12. Frontend Code Example

**Supabase RPC:**

```ts
const { data, error } = await supabase.rpc("function_name", {
  p_field_name: value,
  p_field_name: value,
});

if (error) {
  // handle error
}
```

**Supabase Direct Table:**

```ts
const { data, error } = await supabase
  .from("table_name")
  .insert({ field: value })
  .select("id, field1, field2")
  .single();
```

**Xano / Django (fetch):**

```ts
const response = await fetch("/api/endpoint", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${accessToken}`,
  },
  body: JSON.stringify({ field_name: value }),
});

const result = await response.json();
```

---

## 13. Test Cases

| # | Test Case | Expected Result |
|---|-----------|----------------|
| 1 | [Allowed role] performs action successfully | 200, returns data |
| 2 | Viewer role attempts action | 403 FORBIDDEN |
| 3 | Non-member attempts action | 403 FORBIDDEN |
| 4 | Anonymous user attempts action | 401 UNAUTHENTICATED |
| 5 | Required field `[field]` is missing | 422 VALIDATION_ERROR |
| 6 | Field `[field]` fails validation rules | 422 VALIDATION_ERROR |
| 7 | Duplicate `[field]` in same workspace | 409 CONFLICT |
| 8 | User attempts action on another workspace's data | 403 FORBIDDEN or 404 NOT_FOUND |

---

## Standard Error Codes Reference

| Code | HTTP | When to Use |
|------|------|------------|
| `UNAUTHENTICATED` | 401 | User is not logged in or token is expired |
| `FORBIDDEN` | 403 | Authenticated but lacks permission |
| `VALIDATION_ERROR` | 422 | Input is missing or fails validation |
| `NOT_FOUND` | 404 | Record does not exist or user cannot access it |
| `CONFLICT` | 409 | Duplicate or conflicting data |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `EXTERNAL_SERVICE_ERROR` | 502 | Third-party service call failed |
| `DATABASE_ERROR` | 500 | Database operation failed |
