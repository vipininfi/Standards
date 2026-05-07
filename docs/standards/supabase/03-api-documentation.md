# API Documentation Templates & Examples

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Sections 17–19 (documentation template, frontend code examples, RPC pattern), Section 41 (full worked API example: Add Project)

---

# 17. API Documentation Template

Every backend API must be documented like this.

```md id="47opy7"
# API: Create Project

## 1. Purpose
Creates a new project inside a workspace.

## 2. API Type
Direct Supabase Insert / RPC / Edge Function

## 3. Endpoint / Table / Function
Table: public.projects
Operation: insert

## 4. Auth Required
Yes.

Frontend must pass the logged-in user's Supabase access token.

Header:
Authorization: Bearer <access_token>

## 5. Permission Required
User must be a member of the workspace.
Allowed roles:
- owner
- admin
- manager

## 6. Request Payload

{
  "workspace_id": "uuid",
  "name": "string",
  "description": "string | optional"
}

## 7. Required Fields
- workspace_id
- name

## 8. Validation Rules
- workspace_id must be a valid UUID.
- name is required.
- name must be 2–100 characters.
- description is optional.
- description max length: 1000 characters.

## 9. Success Response

{
  "success": true,
  "data": {
    "id": "uuid",
    "workspace_id": "uuid",
    "name": "Website Redesign",
    "description": "Client website project",
    "status": "active",
    "created_by": "uuid",
    "created_at": "2026-05-07T10:00:00Z"
  },
  "message": "Project created successfully"
}

## 10. Error Responses

### 401 UNAUTHENTICATED
Case:
- User is not logged in.
- Missing or expired token.

Frontend Action:
- Redirect to login.
- Show: "Your session has expired. Please log in again."

### 403 FORBIDDEN
Case:
- User is not a workspace member.
- User role is viewer/member without create permission.

Frontend Action:
- Show: "You do not have permission to create projects in this workspace."

### 422 VALIDATION_ERROR
Case:
- Missing name.
- Invalid workspace_id.
- Name too short or too long.

Frontend Action:
- Highlight invalid field.

### 409 CONFLICT
Case:
- Project with same name already exists in workspace, if uniqueness rule is enabled.

Frontend Action:
- Show: "A project with this name already exists."

### 500 INTERNAL_ERROR
Case:
- Unexpected database/server error.

Frontend Action:
- Show generic error and allow retry.

## 11. RLS Policies Involved
- Workspace members can view projects.
- Owner/admin/manager can create projects.
- created_by must equal auth.uid().

## 12. Frontend Implementation Notes
- User must be logged in before calling this API.
- Get access token from Supabase session.
- Do not send created_by from frontend unless backend requires it.
- If sent, created_by must match auth.uid().
- Show loader while request is pending.
- Disable submit button during request.
- Preserve form values if request fails.

## 13. Test Cases
- Logged-in owner creates project successfully.
- Logged-in manager creates project successfully.
- Viewer cannot create project.
- Non-member cannot create project.
- Anonymous user cannot create project.
- Missing name returns validation error.
- Invalid workspace_id returns validation error.
- Duplicate project name returns conflict.
```

---

# 18. Example Frontend Supabase Call Documentation

Backend developer should provide exact frontend example.

```ts id="cpgu88"
const { data, error } = await supabase
  .from("projects")
  .insert({
    workspace_id: workspaceId,
    name: form.name.trim(),
    description: form.description?.trim() || null,
    created_by: user.id
  })
  .select("id, workspace_id, name, description, status, created_by, created_at")
  .single();

if (error) {
  // Map Supabase/PostgREST error to frontend message
  throw error;
}

return data;
```

But for better security, prefer backend-controlled `created_by` through RPC when possible.

---

# 19. Better Pattern: RPC for Create Project

For important business workflows, use RPC so frontend does not control sensitive fields.

```sql id="vg3gaa"
create or replace function public.create_project(
  p_workspace_id uuid,
  p_name text,
  p_description text default null
)
returns public.projects
language plpgsql
security invoker
as $$
declare
  v_project public.projects;
begin
  if p_name is null or length(trim(p_name)) < 2 then
    raise exception 'Project name must be at least 2 characters'
      using errcode = 'P0001';
  end if;

  insert into public.projects (
    workspace_id,
    name,
    description,
    created_by
  )
  values (
    p_workspace_id,
    trim(p_name),
    nullif(trim(p_description), ''),
    auth.uid()
  )
  returning * into v_project;

  return v_project;
end;
$$;
```

Frontend call:

```ts id="2j8ozj"
const { data, error } = await supabase.rpc("create_project", {
  p_workspace_id: workspaceId,
  p_name: form.name,
  p_description: form.description || null
});
```

---



---

# 41. Full API Documentation Example: Add Project

```md id="el4d4n"
# API: Add Project

## Status
Ready for frontend integration.

## Owner
Backend Team

## Last Updated
2026-05-07

## Purpose
Creates a new project inside a workspace.

## API Type
Supabase RPC

## Function Name
create_project

## Auth Required
Yes.

The frontend must use the Supabase session generated from login/signup.

Required Header:
Authorization: Bearer <access_token>

## Permission Required
The logged-in user must be a member of the workspace.

Allowed roles:
- owner
- admin
- manager

Not allowed:
- viewer
- non-member
- anonymous user

## Request Payload

{
  "p_workspace_id": "uuid",
  "p_name": "string",
  "p_description": "string | null"
}

## Required Parameters

| Field | Type | Required | Rules |
|---|---|---:|---|
| p_workspace_id | uuid | Yes | Must be a workspace the user belongs to. |
| p_name | string | Yes | 2–100 characters after trim. |
| p_description | string/null | No | Max 1000 characters. |

## Frontend Example

const { data, error } = await supabase.rpc("create_project", {
  p_workspace_id: workspaceId,
  p_name: values.name,
  p_description: values.description || null
});

## Success Response

{
  "success": true,
  "data": {
    "id": "2876d57e-6c37-4d56-84fc-87bdf03e97ba",
    "workspace_id": "23f6c9f2-8947-47e8-a1c2-d7e27c9cdb18",
    "name": "Hotel Renovation",
    "description": "FF&E renovation project",
    "status": "active",
    "created_by": "auth-user-id",
    "created_at": "2026-05-07T10:00:00Z"
  },
  "message": "Project created successfully."
}

## Error Responses

### 401 UNAUTHENTICATED

When:
- User is not logged in.
- Access token is missing.
- Access token expired.

Response:
{
  "success": false,
  "error": {
    "code": "UNAUTHENTICATED",
    "message": "Your session has expired. Please log in again."
  }
}

Frontend:
- Redirect to login.
- Clear local auth state if needed.

### 403 FORBIDDEN

When:
- User is not a member of the workspace.
- User role cannot create projects.

Response:
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to create projects in this workspace."
  }
}

Frontend:
- Show permission error.
- Hide create button for roles that cannot create.

### 422 VALIDATION_ERROR

When:
- Project name missing.
- Project name too short.
- Project name too long.
- Invalid workspace_id.

Response:
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please fix the highlighted fields.",
    "fields": {
      "p_name": "Project name must be between 2 and 100 characters."
    }
  }
}

Frontend:
- Show field-level validation.

### 409 CONFLICT

When:
- Project name already exists in same workspace.

Response:
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "A project with this name already exists."
  }
}

Frontend:
- Show duplicate error near project name field.

### 500 INTERNAL_ERROR

When:
- Unexpected database/server failure.

Response:
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Something went wrong. Please try again."
  }
}

Frontend:
- Show retry option.
- Do not clear form data.

## RLS Policies Used

Table:
- public.projects

Policies:
- Workspace members can view projects.
- Owner/admin/manager can create projects.
- created_by must equal auth.uid().
- Non-members cannot access project rows.

## Frontend UI Requirements

- Disable submit button while request is running.
- Show "Creating..." loading state.
- Preserve input on failure.
- On success, redirect to project detail page or update project list.
- On 401, redirect to login.
- On 403, show permission message.
- On 422, highlight fields.
- On 409, show duplicate project name warning.

## Test Cases

1. Owner creates project successfully.
2. Admin creates project successfully.
3. Manager creates project successfully.
4. Viewer cannot create project.
5. Non-member cannot create project.
6. Anonymous user cannot create project.
7. Missing name returns validation error.
8. Duplicate name returns conflict.
9. Invalid workspace_id returns validation error.
10. User cannot create project in another workspace.
```

---

