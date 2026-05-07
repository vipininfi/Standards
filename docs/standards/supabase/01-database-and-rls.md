# Developer Responsibilities, Database Schema & RLS

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Section 1 (Responsibilities), Sections 4–12 (Database schema, naming, constraints, indexes, RLS policies, auth standards, service role rules)

---

# 1. Backend Supabase Developer Responsibilities

## P0 — Mandatory Responsibilities

| Area            | Responsibility                                                                    |
| --------------- | --------------------------------------------------------------------------------- |
| Database Schema | Create clean tables, constraints, indexes, relationships, timestamps.             |
| RLS Policies    | Implement row-level access rules for every exposed table.                         |
| API Layer       | Decide whether API is direct Supabase query, RPC, or Edge Function.               |
| Auth Rules      | Define who can call what: anonymous, authenticated, owner, admin, org member.     |
| Validation      | Validate payload on frontend and backend/server/database.                         |
| Error Handling  | Return predictable errors, never silent failure.                                  |
| Documentation   | Write clear frontend-facing API contract for every API.                           |
| Testing         | Test success, validation error, unauthorized, forbidden, duplicate, missing data. |
| Security        | No service-role key exposed, no open table without RLS, no unsafe policies.       |
| Observability   | Log important actions, failed requests, webhook retries, background job failures. |

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

# 4. Database Schema Standards

## P0 — Every Table Should Have

| Column                             | Standard                                        |
| ---------------------------------- | ----------------------------------------------- |
| `id`                               | UUID primary key.                               |
| `created_at`                       | Timestamp with timezone, default `now()`.       |
| `updated_at`                       | Timestamp with timezone, updated automatically. |
| `created_by`                       | User UUID where ownership matters.              |
| `updated_by`                       | User UUID where audit matters.                  |
| `workspace_id` / `organization_id` | Required for multi-tenant SaaS.                 |
| `deleted_at`                       | Soft delete if record recovery is needed.       |
| `status`                           | Use enum/check constraint for workflow records. |

Example:

```sql id="70uj97"
create table public.projects (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  name text not null,
  description text,
  status text not null default 'active' check (status in ('active', 'archived')),
  created_by uuid not null references auth.users(id),
  updated_by uuid references auth.users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  deleted_at timestamptz
);
```

---

# 5. Naming Standards

## Table Naming

| Good                    | Bad                   |
| ----------------------- | --------------------- |
| `projects`              | `Project`             |
| `project_members`       | `projectMember`       |
| `workspace_invitations` | `workspaceInviteData` |
| `audit_logs`            | `logsTable`           |

Use:

```text id="42q5wh"
snake_case
plural table names
clear foreign key names
predictable column names
```

## Column Naming

| Purpose     | Standard                                               |
| ----------- | ------------------------------------------------------ |
| Foreign key | `user_id`, `project_id`, `workspace_id`                |
| Boolean     | `is_active`, `is_verified`, `has_access`               |
| Timestamp   | `created_at`, `updated_at`, `deleted_at`, `expires_at` |
| Count       | `message_count`, `member_count`                        |
| JSON        | `metadata`, `settings`, `config`                       |

---

# 6. Constraint Standards

A Supabase developer should not depend only on application code.

## P0 Constraints

| Constraint    | Use For                                  |
| ------------- | ---------------------------------------- |
| `not null`    | Required fields.                         |
| `unique`      | Email, slug, invite token, external IDs. |
| `foreign key` | Relationships.                           |
| `check`       | Status, amount, enum-like values.        |
| `default`     | IDs, timestamps, default status.         |

Example:

```sql id="nvcyei"
alter table public.projects
add constraint projects_name_length_check
check (char_length(name) between 2 and 100);
```

---

# 7. Indexing Standards

## P0 Indexes

| Scenario            | Index                                    |
| ------------------- | ---------------------------------------- |
| Tenant filtering    | `workspace_id`                           |
| Ownership filtering | `created_by`                             |
| Search by status    | `status`                                 |
| Date sorting        | `created_at desc`                        |
| Foreign keys        | All FK columns                           |
| Soft delete queries | Partial index where `deleted_at is null` |

Example:

```sql id="2plp4r"
create index projects_workspace_id_idx
on public.projects(workspace_id);

create index projects_workspace_active_idx
on public.projects(workspace_id, created_at desc)
where deleted_at is null;
```

---

# 8. RLS Standards

## P0 Rules

| Rule                  | Standard                                               |
| --------------------- | ------------------------------------------------------ |
| Enable RLS            | Every exposed table.                                   |
| No open policies      | Never use `true` unless table is intentionally public. |
| Separate policies     | Separate select, insert, update, delete policies.      |
| Use `auth.uid()`      | For user ownership checks.                             |
| Use membership tables | For team/workspace apps.                               |
| Use `with check`      | Mandatory for insert/update ownership validation.      |
| Test with real users  | Test owner, member, outsider, anonymous.               |

Enable RLS:

```sql id="rt1m4r"
alter table public.projects enable row level security;
```

---

# 9. RLS Policy Types

## SELECT Policy

Controls what rows user can read.

```sql id="bz4fqs"
create policy "Users can view projects in their workspace"
on public.projects
for select
to authenticated
using (
  exists (
    select 1
    from public.workspace_members wm
    where wm.workspace_id = projects.workspace_id
      and wm.user_id = auth.uid()
  )
);
```

## INSERT Policy

Controls what rows user can create.

```sql id="6mdwod"
create policy "Workspace members can create projects"
on public.projects
for insert
to authenticated
with check (
  created_by = auth.uid()
  and exists (
    select 1
    from public.workspace_members wm
    where wm.workspace_id = projects.workspace_id
      and wm.user_id = auth.uid()
      and wm.role in ('owner', 'admin', 'manager')
  )
);
```

## UPDATE Policy

Controls what rows user can update.

```sql id="k5c2u9"
create policy "Admins and managers can update projects"
on public.projects
for update
to authenticated
using (
  exists (
    select 1
    from public.workspace_members wm
    where wm.workspace_id = projects.workspace_id
      and wm.user_id = auth.uid()
      and wm.role in ('owner', 'admin', 'manager')
  )
)
with check (
  exists (
    select 1
    from public.workspace_members wm
    where wm.workspace_id = projects.workspace_id
      and wm.user_id = auth.uid()
      and wm.role in ('owner', 'admin', 'manager')
  )
);
```

## DELETE Policy

Prefer soft delete over hard delete.

```sql id="gj60i5"
create policy "Only owners can delete projects"
on public.projects
for delete
to authenticated
using (
  exists (
    select 1
    from public.workspace_members wm
    where wm.workspace_id = projects.workspace_id
      and wm.user_id = auth.uid()
      and wm.role = 'owner'
  )
);
```

---

# 10. RLS Anti-Patterns

Avoid these:

```sql id="dp9sxi"
using (true)
```

```sql id="t2b7iz"
with check (true)
```

```sql id="8fzvep"
created_by = auth.uid()
```

without also checking workspace membership.

```sql id="mvvzg5"
auth.jwt() -> 'user_metadata' ->> 'role' = 'admin'
```

Do not use user-editable metadata for authorization. Supabase specifically warns that `raw_user_meta_data` can be updated by authenticated users, while `raw_app_meta_data` is safer for authorization data. ([Supabase][1])

---

# 11. Auth Standards

## P0 Auth Requirements

| API Type      | Auth Standard                                        |
| ------------- | ---------------------------------------------------- |
| Public API    | No auth, but rate-limited and sanitized.             |
| User API      | Requires Supabase access token.                      |
| Workspace API | Requires authenticated user + workspace membership.  |
| Admin API     | Requires admin role from secure source.              |
| Webhook API   | Requires signature verification.                     |
| Edge Function | Must validate JWT or use secure server-side pattern. |

Frontend must send:

```ts id="s5kb29"
Authorization: Bearer <supabase_access_token>
```

For Edge Functions, current Supabase guidance is that developers should explicitly own and control auth handling patterns rather than relying on hidden implicit verification, especially with newer JWT signing key patterns. ([Supabase][3])

---

# 12. Service Role Key Rules

## Absolute Rule

```text id="zezosz"
The service_role key must never be used in frontend code.
```

## Service Role Can Be Used Only In

| Place                         | Allowed?       |
| ----------------------------- | -------------- |
| Browser frontend              | No             |
| Mobile app                    | No             |
| Public GitHub repo            | No             |
| Supabase Edge Function secret | Yes            |
| Private backend server        | Yes            |
| Secure cron job               | Yes            |
| Local development `.env`      | Yes, carefully |

## Service Role Use Cases

| Use Case             | Example                           |
| -------------------- | --------------------------------- |
| Admin-only operation | Suspend user.                     |
| Webhook processing   | Stripe subscription update.       |
| Background job       | Process uploaded file.            |
| System automation    | Create default workspace/profile. |

Even when using service role, the developer must manually check authorization because service role bypasses RLS.

---

# 13. API Design Standards

## Every API Must Define

```text id="yh0r4v"
1. API name
2. Purpose
3. Endpoint / Supabase table / RPC / Edge Function name
4. Auth required or not
5. Required role
6. Required headers
7. Request payload
8. Validation rules
9. Success response
10. Error responses
11. RLS policies involved
12. Frontend implementation notes
13. Test cases
14. Example request
15. Example response
```

---

