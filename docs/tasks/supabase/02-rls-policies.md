# Task: Write RLS Policies for the Projects Table

**Platform:** Supabase
**Covers:** [RLS Policy Checklist](../../checklists/supabase/rls-policies.md) · [Database & RLS Standards](../../standards/supabase/01-database-and-rls.md)

---

## Scenario

Continuing from [Task 01](01-create-projects-table.md), the `projects` table is created with RLS enabled but no policies yet. You will now write all four RLS policies following the access model defined below.

---

## Access Model

Before writing a single policy, this is the agreed access model for the `projects` table:

| Operation | Who Can Do It |
|-----------|--------------|
| SELECT | Any member of the workspace (all roles) — active projects only (deleted_at IS NULL) |
| INSERT | Any member of the workspace — created_by must be the current user |
| UPDATE | The project creator OR a workspace admin/owner |
| DELETE | Not allowed via RLS — soft delete only (UPDATE deleted_at) |

### Role Definitions

Roles are stored in the `workspace_members` table:

```sql
workspace_members (
  id           uuid,
  workspace_id uuid,
  user_id      uuid,
  role         text  -- 'owner', 'admin', 'member', 'viewer'
)
```

Viewers can read but not create or edit projects.

---

## What to Write

Write four separate RLS policies as SQL. Each must:

- Use `auth.uid()` — never hardcoded values
- Verify workspace membership via a subquery on `workspace_members` — never just matching the payload's `workspace_id`
- Match the access model above exactly

### Policy 1 — SELECT

Only workspace members can see projects in their workspace. Soft-deleted projects (`deleted_at IS NOT NULL`) are hidden from regular members. Admins and owners can see soft-deleted projects.

> Hint: you may need two SELECT policies or a combined USING clause with role logic.

### Policy 2 — INSERT

Any workspace member (role: owner, admin, member — not viewer) can create a project.

- `workspace_id` in the new row must be a workspace the user belongs to
- `created_by` in the new row must equal `auth.uid()` — user cannot create projects attributed to someone else

### Policy 3 — UPDATE

Only the project creator (`created_by = auth.uid()`) OR a workspace admin/owner can update a project.

- Users cannot move a project to a different workspace (the `workspace_id` in the updated row must match the original)
- Use both `USING` (existing row access) and `WITH CHECK` (updated row validation)

### Policy 4 — DELETE

No policy. Hard deletes are not permitted. Any attempt to DELETE will be rejected by the default deny.

---

## Testing Requirements

After writing all policies, verify each of the following manually or in a test:

| Test | Expected Result |
|------|-----------------|
| SELECT as anonymous | 0 rows returned |
| SELECT as authenticated non-member | 0 rows returned |
| SELECT as workspace member (role: member) | Own workspace active projects only |
| SELECT as workspace admin | Own workspace projects including soft-deleted |
| INSERT with correct workspace_id as member | Allowed, created_by = auth.uid() |
| INSERT with another user's workspace_id | Blocked |
| INSERT with created_by set to another user's UUID | Blocked (WITH CHECK fails) |
| INSERT as viewer | Blocked |
| UPDATE own project as member | Allowed |
| UPDATE another member's project as member | Blocked |
| UPDATE any project as admin | Allowed |
| UPDATE to change workspace_id | Blocked |
| DELETE as admin | Blocked (no DELETE policy) |

---

## What You Should NOT Do

- Do not use `FOR ALL` — write explicit policies per operation
- Do not check `workspace_id = input.workspace_id` from the request payload — always use a subquery on `workspace_members`
- Do not read the role from a JWT claim or request body — query `workspace_members` table in the policy
- Do not forget `WITH CHECK` on UPDATE — `USING` alone allows reading the row but not validating the new values
- Do not create a DELETE policy — soft delete is the only permitted deletion method

---

## Checklist to Run When Done

Use the [RLS Policy Checklist](../../checklists/supabase/rls-policies.md#7-rls-checklist--before-marking-done).

---

## Done When

```text
POLICIES WRITTEN
[ ] SELECT policy: members see active projects in their workspace
[ ] SELECT policy (admin/owner): can also see soft-deleted projects
[ ] INSERT policy: members (not viewers) can insert; created_by = auth.uid() enforced
[ ] UPDATE policy: creator OR admin/owner; workspace_id cannot change
[ ] No DELETE policy (intentionally absent)

POLICY CORRECTNESS
[ ] All policies use auth.uid()
[ ] Workspace membership verified via subquery on workspace_members
[ ] Role read from workspace_members — not from JWT or request
[ ] INSERT WITH CHECK enforces created_by = auth.uid()
[ ] UPDATE WITH CHECK prevents workspace_id change

TESTING
[ ] Anonymous: blocked
[ ] Non-member: blocked
[ ] Member: sees own workspace, can insert and update own projects
[ ] Viewer: can only select, cannot insert or update
[ ] Admin: sees all including soft-deleted, can update any
[ ] Wrong workspace_id in INSERT: blocked
[ ] Wrong created_by in INSERT: blocked
[ ] DELETE as admin: blocked
```
