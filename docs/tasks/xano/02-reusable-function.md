# Task — Xano Reusable Function: `check_workspace_permission`

**Platform:** Xano
**Checklist to Run:** [Reusable Function Checklist](../../checklists/xano/reusable-function.md)
**Standard:** [Xano Standards](../../standards/xano-standards.md)

---

## Scenario

Across your Xano API, you have at least 5 endpoints that all need to verify the same thing: "Is the authenticated user a member of the requested workspace, and what is their role?" Right now, each endpoint duplicates the same 4-step database lookup and permission check. This needs to be extracted into a reusable function.

---

## What to Build

A Xano Custom Function named `check_workspace_permission`.

---

## Requirements

### Function Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| workspace_id | integer | Yes | The workspace to check membership for |
| user_id | integer | Yes | The authenticated user's ID (from auth token) |
| required_role | text | No | Minimum required role: 'member', 'editor', 'admin', 'owner'. If not provided, any role is accepted. |

### Function Logic (4 Steps)

**Step 1 — Query workspace_members**
```
Query: workspace_members
WHERE workspace_id = inputs.workspace_id
  AND user_id = inputs.user_id
  AND status = 'active'
LIMIT 1
```

**Step 2 — Check if member exists**
```
IF result is empty:
  RAISE error:
    code: "FORBIDDEN"
    message: "You are not a member of this workspace."
    http_status: 403
```

**Step 3 — Check role if required_role is provided**
```
Role hierarchy: member < editor < admin < owner

IF required_role is provided:
  IF user's role does not meet the minimum required_role:
    RAISE error:
      code: "FORBIDDEN"
      message: "You need [required_role] access to perform this action."
      http_status: 403
```

**Step 4 — Return the member record**
```
RETURN the workspace_member record:
{
  "id": ...,
  "workspace_id": ...,
  "user_id": ...,
  "role": "admin",
  "status": "active"
}
```

### Role Hierarchy

The role check in Step 3 must implement a hierarchy. Requiring "admin" means "admin" OR "owner" — not only exactly "admin".

| Required Role | Accepted Roles |
|--------------|---------------|
| member | member, editor, admin, owner |
| editor | editor, admin, owner |
| admin | admin, owner |
| owner | owner only |

---

## How It Gets Called

After extraction, these 5 endpoints will call `check_workspace_permission` as a step (after auth token validation):

1. **Create Project** — `required_role: 'editor'`
2. **Delete Project** — `required_role: 'admin'`
3. **Invite Member** — `required_role: 'admin'`
4. **Update Workspace Settings** — `required_role: 'admin'`
5. **Get Project Details** — no required_role (any member)

Each endpoint still validates its own auth token first. This function receives `user_id` from the endpoint — not from the token directly.

---

## What You Should NOT Do

```text
× Put the auth token validation inside this reusable function
× Hardcode workspace IDs or user IDs inside the function
× Return null or false on permission failure — always raise an error
× Use the wrong role comparison (treat roles as equal instead of hierarchical)
× Duplicate this logic in any endpoint after extracting it
```

---

## Checklist to Run

Before marking done, run: [Reusable Function Checklist](../../checklists/xano/reusable-function.md)

---

## Done When

```text
[ ] Function named check_workspace_permission
[ ] Inputs defined: workspace_id (integer), user_id (integer), required_role (text, optional)
[ ] Step 1: queries workspace_members correctly with status = 'active'
[ ] Step 2: raises FORBIDDEN with correct message if member not found
[ ] Step 3: implements role hierarchy — not just equality check
[ ] Step 4: returns the member record on success
[ ] All 5 existing endpoints updated to call this function (duplicate logic removed)
[ ] All 5 endpoints retested — no regressions
[ ] Function documented: inputs, output shape, all error cases
```
