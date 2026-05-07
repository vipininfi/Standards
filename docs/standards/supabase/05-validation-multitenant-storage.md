# Validation, Multi-Tenant & Storage Standards

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Sections 24–26 (validation layers, frontend error contract, field-level errors), Sections 27–31 (multi-tenant scoping, workspace membership table, role matrix, storage security, storage RLS)

---

# 24. Validation Standards

## Backend Must Validate

| Field Type | Validation                                |
| ---------- | ----------------------------------------- |
| UUID       | Valid UUID format and accessible by user. |
| Email      | Trim, lowercase, valid format.            |
| Phone      | E.164 format.                             |
| Text       | Trim, min/max length.                     |
| Number     | Min/max/decimal precision.                |
| Enum       | Must be allowed value.                    |
| URL        | Must be valid `http`/`https`.             |
| Date       | Valid ISO date and logical range.         |
| File       | MIME, extension, size, ownership.         |
| JSON       | Schema validation.                        |

## Validation Must Exist In Three Layers

| Layer             | Purpose                                   |
| ----------------- | ----------------------------------------- |
| Frontend          | Good UX.                                  |
| Edge Function/RPC | Security and business rules.              |
| Database          | Final protection through constraints/RLS. |

---

# 25. Frontend Error Handling Contract

Backend developer must tell frontend exactly what to do.

| Backend Error            | Frontend Action                            |
| ------------------------ | ------------------------------------------ |
| `UNAUTHENTICATED`        | Redirect to login/session expired modal.   |
| `FORBIDDEN`              | Show no-permission message or hide action. |
| `VALIDATION_ERROR`       | Highlight field-level errors.              |
| `NOT_FOUND`              | Show 404/resource unavailable.             |
| `CONFLICT`               | Show duplicate/conflict message.           |
| `RATE_LIMITED`           | Disable retry temporarily, show cooldown.  |
| `INTERNAL_ERROR`         | Show generic retry message.                |
| `EXTERNAL_SERVICE_ERROR` | Show integration-specific failure.         |

---

# 26. API Documentation Must Include Field-Level Errors

Example:

```json id="zi29py"
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please fix the highlighted fields.",
    "fields": {
      "name": "Project name is required.",
      "workspace_id": "Workspace ID must be a valid UUID."
    }
  }
}
```

Frontend should not parse raw database messages directly. Backend should map them.

---

# 27. Multi-Tenant SaaS Standards

If the app has multiple companies, clients, hotels, agencies, teams, or workspaces, every important table must include tenant scoping.

## Required Pattern

```text id="i77nqx"
workspace_id / organization_id must exist on tenant-owned records.
```

## Every Query Must Be Scoped

Bad:

```ts id="kv0gj2"
supabase.from("projects").select("*");
```

Good:

```ts id="ew62t1"
supabase
  .from("projects")
  .select("*")
  .eq("workspace_id", workspaceId);
```

But still do not trust frontend `workspaceId`. RLS must verify user membership.

---

# 28. Workspace Membership Table Standard

```sql id="yjp3fm"
create table public.workspace_members (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  user_id uuid not null references auth.users(id) on delete cascade,
  role text not null check (role in ('owner', 'admin', 'manager', 'member', 'viewer')),
  created_at timestamptz not null default now(),
  unique (workspace_id, user_id)
);
```

This table becomes the base of most RLS policies.

---

# 29. Role Permission Matrix Documentation

Backend must document permissions like this:

| Action         | Owner | Admin | Manager |   Member | Viewer |
| -------------- | ----: | ----: | ------: | -------: | -----: |
| View projects  |   Yes |   Yes |     Yes |      Yes |    Yes |
| Create project |   Yes |   Yes |     Yes |       No |     No |
| Update project |   Yes |   Yes |     Yes | Own only |     No |
| Delete project |   Yes |    No |      No |       No |     No |
| Invite member  |   Yes |   Yes |      No |       No |     No |
| Change billing |   Yes |    No |      No |       No |     No |

Frontend uses this to hide/show actions, but backend/RLS remains the real authority.

---

# 30. Storage Security Standards

For Supabase Storage:

| Rule              | Standard                              |
| ----------------- | ------------------------------------- |
| Buckets           | Separate public/private buckets.      |
| Public Files      | Only for assets safe to expose.       |
| Private Files     | Use signed URLs.                      |
| Upload Policy     | User can upload only to allowed path. |
| File Path         | Include tenant/user scope.            |
| File Size         | Validate.                             |
| MIME Type         | Validate.                             |
| Delete Permission | Owner/admin only.                     |

Path standard:

```text id="2p0l6l"
workspace/{workspace_id}/projects/{project_id}/files/{file_id}.pdf
```

Never allow users to upload to arbitrary paths.

---

# 31. Storage RLS Policy Pattern

Example idea:

```sql id="hl2243"
create policy "Workspace members can view workspace files"
on storage.objects
for select
to authenticated
using (
  bucket_id = 'project-files'
  and exists (
    select 1
    from public.workspace_members wm
    where wm.user_id = auth.uid()
      and storage.foldername(name)[1] = 'workspace'
      and wm.workspace_id::text = storage.foldername(name)[2]
  )
);
```

---

