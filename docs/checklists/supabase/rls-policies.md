# Supabase — RLS Policy Checklist

> **Core Rule:** Every table exposed to the frontend must have RLS enabled. No table is open by default. Every policy must be tested across every role before it is considered done.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-before-writing-policies) | Before Writing Policies |
| [2](#2-rls-policy-types) | RLS Policy Types |
| [3](#3-policy-patterns-by-access-type) | Policy Patterns by Access Type |
| [4](#4-role-permission-matrix) | Role Permission Matrix |
| [5](#5-common-mistakes--anti-patterns) | Common Mistakes & Anti-Patterns |
| [6](#6-testing-rls-policies) | Testing RLS Policies |
| [7](#7-rls-checklist--before-marking-done) | RLS Checklist — Before Marking Done |

---

# 1. Before Writing Policies

Answer all of these before writing a single policy:

```text
[ ] What is the access model? (workspace-scoped / user-scoped / public / admin-only)
[ ] Who should be able to SELECT rows? (everyone / authenticated / members / owner / admin)
[ ] Who should be able to INSERT rows? (authenticated / members / admin only)
[ ] Who should be able to UPDATE rows? (owner / admin / members with role)
[ ] Who should be able to DELETE rows? (owner / admin only / nobody)
[ ] Are there role-based differences within the same table? (admin can do more than member)
[ ] Does anonymous access apply to any operation on this table?
[ ] Is this a multi-tenant table? (workspace_id must be checked, not just user_id)
```

---

# 2. RLS Policy Types

| Policy Keyword | Controls | When to Use |
|----------------|----------|-------------|
| `FOR SELECT USING (...)` | Read access | Who can see rows |
| `FOR INSERT WITH CHECK (...)` | Write access — new rows | Who can add rows |
| `FOR UPDATE USING (...) WITH CHECK (...)` | Update access | Who can change existing rows |
| `FOR DELETE USING (...)` | Delete access | Who can remove rows |
| `FOR ALL USING (...)` | All operations | Only for simple cases — prefer explicit policies |

> **Rule:** Always write explicit policies for SELECT, INSERT, UPDATE, DELETE separately. Using `FOR ALL` hides intent and makes auditing harder.

---

# 3. Policy Patterns by Access Type

## Public Read (No Auth Required)

```sql
-- Anyone (including anonymous) can read
CREATE POLICY "public_select" ON blog_posts
  FOR SELECT USING (is_published = true);
```

## Authenticated Users Only

```sql
-- Any logged-in user can read
CREATE POLICY "authenticated_select" ON user_profiles
  FOR SELECT USING (auth.uid() IS NOT NULL);
```

## Owner Only

```sql
-- User can only see/edit their own rows
CREATE POLICY "owner_select" ON user_settings
  FOR SELECT USING (user_id = auth.uid());

CREATE POLICY "owner_update" ON user_settings
  FOR UPDATE USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());
```

## Workspace Members (Multi-Tenant)

```sql
-- Any member of the workspace can read
CREATE POLICY "workspace_member_select" ON projects
  FOR SELECT USING (
    workspace_id IN (
      SELECT workspace_id FROM workspace_members
      WHERE user_id = auth.uid()
    )
  );

-- Insert: must be a member, created_by must be current user
CREATE POLICY "workspace_member_insert" ON projects
  FOR INSERT WITH CHECK (
    workspace_id IN (
      SELECT workspace_id FROM workspace_members
      WHERE user_id = auth.uid()
    )
    AND created_by = auth.uid()
  );
```

## Role-Based Within Workspace (Admin vs Member)

```sql
-- Only admins can delete
CREATE POLICY "admin_delete" ON projects
  FOR DELETE USING (
    EXISTS (
      SELECT 1 FROM workspace_members
      WHERE workspace_id = projects.workspace_id
        AND user_id = auth.uid()
        AND role = 'admin'
    )
  );

-- Admins or the record creator can update
CREATE POLICY "owner_or_admin_update" ON projects
  FOR UPDATE USING (
    created_by = auth.uid()
    OR EXISTS (
      SELECT 1 FROM workspace_members
      WHERE workspace_id = projects.workspace_id
        AND user_id = auth.uid()
        AND role IN ('admin', 'owner')
    )
  );
```

## Intentionally Block an Operation

```sql
-- Nobody can delete (use soft delete instead)
-- Simply do NOT create a DELETE policy
-- OR explicitly block it:
CREATE POLICY "no_delete" ON important_records
  FOR DELETE USING (false);
```

---

# 4. Role Permission Matrix

Document this for every table. Example:

| Role | SELECT | INSERT | UPDATE | DELETE |
|------|--------|--------|--------|--------|
| Anonymous | Only published rows | No | No | No |
| Authenticated (non-member) | No | No | No | No |
| Workspace Member | Yes (own workspace) | Yes (own workspace) | Own records only | No |
| Workspace Admin | Yes (own workspace) | Yes (own workspace) | All records | Yes |
| Workspace Owner | Yes (own workspace) | Yes (own workspace) | All records | Yes |
| Service Role | Yes (all) | Yes (all) | Yes (all) | Yes (all) |

> Service role bypasses RLS entirely — it is only used for server-side operations, never in frontend code.

---

# 5. Common Mistakes & Anti-Patterns

| Mistake | Impact | Fix |
|---------|--------|-----|
| RLS not enabled on a table | Any authenticated user can read/write all rows | `ALTER TABLE x ENABLE ROW LEVEL SECURITY` |
| Using `FOR ALL` instead of per-operation policies | Hidden access surface | Write separate SELECT/INSERT/UPDATE/DELETE policies |
| Checking `workspace_id = $workspace_id` from payload | User can pass any workspace_id | Check via subquery: `workspace_id IN (SELECT workspace_id FROM workspace_members WHERE user_id = auth.uid())` |
| Setting `created_by` from request payload | User can set any user as creator | Use `WITH CHECK (created_by = auth.uid())` in INSERT policy |
| Role read from request payload | User can claim to be admin | Read role from `workspace_members` table, not from request |
| Service role key in frontend code | Anyone can bypass RLS | Service role key only ever on server/Edge Functions |
| Not testing as non-member or anonymous | Hidden data leaks undetected | Always test all roles |
| Using `auth.jwt()` claims for roles | JWTs can be outdated | Read role from database in policy, not from JWT |
| Overly permissive `USING (true)` | Open access to all rows | Always scope with auth.uid() check |
| No policy on a table with sensitive data | Default deny is correct, but missing policies is fragile | Document intentionally blocked operations |

---

# 6. Testing RLS Policies

## Test Matrix (Run for Every Table)

| Test Scenario | Expected Result |
|---------------|-----------------|
| SELECT as anonymous | Blocked (or limited to public rows) |
| SELECT as authenticated non-member | Blocked |
| SELECT as workspace member | Only rows in their workspace |
| SELECT as workspace admin | Only rows in their workspace |
| INSERT as anonymous | Blocked |
| INSERT as member with wrong workspace_id | Blocked |
| INSERT as member with correct workspace_id | Allowed, created_by set to auth.uid() |
| UPDATE own record as member | Allowed |
| UPDATE someone else's record as member | Blocked |
| UPDATE any record as admin | Allowed |
| DELETE as member | Blocked |
| DELETE as admin | Allowed |
| Setting created_by to another user's ID | Blocked by INSERT policy WITH CHECK |
| Passing different workspace_id in payload | Blocked by INSERT/UPDATE policy |

## How to Test

```sql
-- Test as a specific user (in Supabase SQL editor)
SET LOCAL role TO authenticated;
SET LOCAL "request.jwt.claims" TO '{"sub": "user-uuid-here"}';

-- Now run your query — it will apply RLS as that user
SELECT * FROM projects;
```

Or use the Supabase client in a test environment with different user JWTs.

---

# 7. RLS Checklist — Before Marking Done

```text
ENABLED
[ ] RLS is enabled on the table (ALTER TABLE ... ENABLE ROW LEVEL SECURITY)
[ ] Default deny confirmed (no data accessible without a matching policy)

POLICIES WRITTEN
[ ] SELECT policy — correct access scope
[ ] INSERT policy — correct access scope + WITH CHECK enforces created_by
[ ] UPDATE policy — correct access scope + WITH CHECK prevents moving records
[ ] DELETE policy — correct scope, OR intentionally blocked with no policy

POLICY CORRECTNESS
[ ] All policies use auth.uid() — no hardcoded UUIDs or values
[ ] Workspace membership verified via subquery (not just workspace_id match)
[ ] Role read from workspace_members table — not from request payload
[ ] created_by set to auth.uid() in INSERT policy — not from request body
[ ] No policy uses USING (true) without a scoping condition

TESTING (EACH ROLE)
[ ] Anonymous: expected access only (or blocked)
[ ] Authenticated non-member: blocked
[ ] Workspace member: correct access to their workspace only
[ ] Workspace admin: correct elevated access
[ ] Workspace owner: correct access
[ ] Attempting wrong workspace_id: blocked
[ ] Attempting to set created_by to another user: blocked
[ ] Attempting to UPDATE another user's record as member: blocked

DOCUMENTATION
[ ] Role permission matrix documented for this table
[ ] RLS policies documented in API documentation or table notes
```

---

## Practice Task

Apply what you learned by writing all four RLS policies for a real table, then testing them across every role.

**→ [Task 02: Write RLS Policies for the Projects Table](../../tasks/supabase/02-rls-policies.md)**

Covers: SELECT (members only, admins see soft-deleted), INSERT (members only, created_by enforced), UPDATE (creator or admin, workspace_id cannot change), no DELETE policy, testing matrix for all roles including viewer, non-member, and anonymous.
