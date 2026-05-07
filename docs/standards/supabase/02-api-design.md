# API Design, Response Format & Error Codes

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Sections 2–3 (API types, core build flow), Sections 13–16 (API design standards, response format, error codes, frontend contract)

---

# 2. Supabase API Types

A Supabase developer must choose the correct API pattern.

## API Type Decision Table

| API Type                     | Use When                                                    | Example                                               |
| ---------------------------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| Direct Supabase Client Query | Simple CRUD where RLS is enough.                            | Add project, list user projects.                      |
| RPC / Postgres Function      | Transactional DB logic or complex query.                    | Create project + owner membership in one transaction. |
| Edge Function                | Business logic, external APIs, webhooks, secret-based work. | Stripe webhook, SendGrid email, AI processing.        |
| Database Trigger             | Automatic database-side event.                              | Create profile after auth user creation.              |
| Database Webhook             | Notify external service after insert/update/delete.         | Send event to automation system.                      |

Supabase Edge Functions are server-side TypeScript functions running on Supabase’s Edge Runtime and are commonly used for webhooks, third-party integrations, and secure server-side logic. ([Supabase][2])

---

# 3. Core Rule: Never Build APIs Randomly

Every API must follow this flow:

```text id="uaod39"
1. Define business requirement.
2. Define database tables needed.
3. Define user roles and permissions.
4. Create migration.
5. Add constraints and indexes.
6. Enable RLS.
7. Add RLS policies.
8. Create API / RPC / Edge Function.
9. Add payload validation.
10. Add structured error handling.
11. Add logs/audit logs.
12. Test all success and failure cases.
13. Write frontend API documentation.
14. Give frontend sample request/response.
15. Mention exact auth/session requirement.
```

---



---

# 14. API Response Format Standard

Use a consistent response shape.

## Success Response

```json id="5kzam8"
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Website Redesign"
  },
  "message": "Project created successfully"
}
```

## Error Response

```json id="gt9bui"
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Project name is required.",
    "field": "name",
    "details": null
  }
}
```

Supabase/PostgREST database errors are returned as JSON objects containing fields like `code`, `details`, `hint`, and `message`; for example, uniqueness violations map to `23505`, foreign key violations to `23503`, and insufficient privileges to `42501`. ([Supabase][4])

---

# 15. Standard Error Codes

Use your own application-level error codes on top of Supabase errors.

| Code                     | HTTP | Meaning                                         |
| ------------------------ | ---: | ----------------------------------------------- |
| `UNAUTHENTICATED`        |  401 | User is not logged in.                          |
| `FORBIDDEN`              |  403 | User is logged in but lacks permission.         |
| `VALIDATION_ERROR`       |  422 | Payload is invalid.                             |
| `NOT_FOUND`              |  404 | Record does not exist or user cannot access it. |
| `CONFLICT`               |  409 | Duplicate or conflicting data.                  |
| `RATE_LIMITED`           |  429 | Too many requests.                              |
| `INTERNAL_ERROR`         |  500 | Unexpected server error.                        |
| `EXTERNAL_SERVICE_ERROR` |  502 | Third-party service failed.                     |
| `DATABASE_ERROR`         |  500 | Database operation failed.                      |

---

# 16. Frontend Must Never Guess API Behavior

For every API, backend documentation must answer:

```text id="h5qa8c"
Does this API need login?
Which token is required?
Which user role can call it?
Which fields are required?
Which fields are optional?
What exact payload should frontend send?
What exact response will frontend receive?
What errors can happen?
What should frontend show for each error?
Does RLS affect this request?
Does this API require workspace_id/project_id?
Can user retry safely?
Is the API idempotent?
```

---

