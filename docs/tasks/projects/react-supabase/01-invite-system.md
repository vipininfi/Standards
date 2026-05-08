# Project — Workspace Invite System

**Stack:** React (TypeScript) + Supabase
**Estimated Time:** 8 hours
**Difficulty:** Intermediate–Advanced

---

## What You Are Building

A workspace invite system. Admins invite people by email, assigning them a role. The invitee receives an email with a unique link. Clicking the link accepts the invite and adds the person as a workspace member. Admins can view pending invites, copy the invite link, resend the email, and revoke pending invites.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | `workspace_invites` table + RLS | 1h |
| 2 | Edge Function: `send-workspace-invite` | 1h 30m |
| 3 | RPC: `accept_workspace_invite` | 1h |
| 4 | React: Members + Invites page (tabs) | 1h |
| 5 | React: Send invite form | 45m |
| 6 | React: Accept invite page | 45m |
| 7 | Invite management (copy, resend, revoke) | 1h |

---

## 1. Database — `workspace_invites` Table

### Schema

| Column | Type | Rules |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE |
| email | text | NOT NULL |
| role | text | NOT NULL, one of: 'admin', 'member', default 'member' |
| token | text | NOT NULL, UNIQUE — a secure random token (gen_random_uuid()::text) |
| status | text | NOT NULL, default 'pending', one of: 'pending', 'accepted', 'revoked', 'expired' |
| invited_by | uuid | FK → auth.users.id |
| expires_at | timestamptz | NOT NULL, default: now() + interval '7 days' |
| accepted_at | timestamptz | nullable |
| created_at | timestamptz | default now() |

### Constraints

```sql
UNIQUE (workspace_id, email, status)  -- prevents duplicate pending invites for same email
CHECK (role IN ('admin', 'member'))
CHECK (status IN ('pending', 'accepted', 'revoked', 'expired'))
```

### Indexes

```sql
CREATE UNIQUE INDEX idx_invites_token ON workspace_invites (token);
CREATE INDEX idx_invites_workspace_status ON workspace_invites (workspace_id, status);
CREATE INDEX idx_invites_email ON workspace_invites (email);
```

### RLS Policies

```text
SELECT:
  — Admins/owners of the workspace can see all invites for their workspace
  — A user can see their own pending invite (WHERE email = auth.email())

INSERT:
  — Only workspace admins/owners can create invites

UPDATE:
  — Only workspace admins/owners can update (for revoke)
  — The Edge Function and RPC use SECURITY DEFINER to bypass RLS

DELETE:
  — Not allowed — use status = 'revoked' instead (soft delete pattern)
```

**Run checklist:** [New Table](../../../../checklists/supabase/new-table.md) · [RLS Policies](../../../../checklists/supabase/rls-policies.md)

---

## 2. Edge Function: `send-workspace-invite`

**Endpoint:** `POST /functions/v1/send-workspace-invite`

### Logic (10 steps)

```text
1.  Validate JWT — reject if no valid auth token
2.  Parse body: { workspace_id, email, role }
3.  Validate: email is valid format
4.  Validate: role is 'admin' or 'member'
5.  Check caller is admin/owner of workspace (query workspace_members)
6.  Check email is not already an active member of the workspace
7.  Check: no pending invite for this email in this workspace already exists
    (if exists: return 409 with existing invite details so frontend can offer "Resend")
8.  Create invite record: insert into workspace_invites with a generated token + 7-day expiry
9.  Send invite email via Resend:
    Subject: "[Admin Name] invited you to join [Workspace Name]"
    Body: includes workspace name, role, and the invite link:
    https://yourapp.com/accept-invite?token={token}
10. Return: { "data": { invite_id, email, role, expires_at } }
```

### Environment Variables Required

```text
RESEND_API_KEY
APP_BASE_URL
```

### Error Responses

| Scenario | HTTP | Code |
|----------|------|------|
| No auth | 401 | AUTH_REQUIRED |
| Not admin | 403 | FORBIDDEN |
| Already a member | 409 | DUPLICATE |
| Invite already pending | 409 | DUPLICATE |
| Invalid email | 400 | VALIDATION_ERROR |

**Run checklist:** [Edge Function](../../../../checklists/supabase/edge-function.md)

---

## 3. RPC: `accept_workspace_invite`

```sql
CREATE OR REPLACE FUNCTION accept_workspace_invite(
  p_token  text
)
RETURNS json
LANGUAGE plpgsql
SECURITY DEFINER  -- needs to write workspace_members even if RLS blocks it
AS $$
DECLARE
  v_user_id   uuid := auth.uid();
  v_invite    workspace_invites%ROWTYPE;
BEGIN
  -- 1. Auth check
  IF v_user_id IS NULL THEN
    RAISE EXCEPTION 'AUTH_REQUIRED: authentication required';
  END IF;

  -- 2. Find invite by token
  SELECT * INTO v_invite FROM workspace_invites WHERE token = p_token;
  IF NOT FOUND THEN
    RAISE EXCEPTION 'NOT_FOUND: invite not found or already used';
  END IF;

  -- 3. Check status is pending
  IF v_invite.status != 'pending' THEN
    RAISE EXCEPTION 'BUSINESS_RULE_VIOLATION: this invite has already been % ', v_invite.status;
  END IF;

  -- 4. Check not expired
  IF v_invite.expires_at < now() THEN
    UPDATE workspace_invites SET status = 'expired' WHERE id = v_invite.id;
    RAISE EXCEPTION 'BUSINESS_RULE_VIOLATION: this invite has expired';
  END IF;

  -- 5. Check caller email matches invite email
  IF (SELECT email FROM auth.users WHERE id = v_user_id) != v_invite.email THEN
    RAISE EXCEPTION 'FORBIDDEN: this invite was sent to a different email address';
  END IF;

  -- 6. Check not already a member
  IF EXISTS (SELECT 1 FROM workspace_members WHERE workspace_id = v_invite.workspace_id AND user_id = v_user_id) THEN
    RAISE EXCEPTION 'DUPLICATE: you are already a member of this workspace';
  END IF;

  -- 7. Create workspace_member record
  INSERT INTO workspace_members (workspace_id, user_id, role, status)
  VALUES (v_invite.workspace_id, v_user_id, v_invite.role, 'active');

  -- 8. Mark invite as accepted
  UPDATE workspace_invites
  SET status = 'accepted', accepted_at = now()
  WHERE id = v_invite.id;

  RETURN json_build_object(
    'data', json_build_object('workspace_id', v_invite.workspace_id, 'role', v_invite.role),
    'message', 'Invite accepted. Welcome to the workspace!'
  );
END;
$$;
```

**Run checklist:** [RPC Function](../../../../checklists/supabase/rpc-function.md)

---

## 4. React: Members & Invites Page

**URL:** `/workspaces/{id}/members`

### Two Tabs

**Tab 1: Members**
```text
[ ] List of active workspace members
[ ] Columns: Avatar · Name · Email · Role badge · Joined Date · Actions (admin: change role, remove)
[ ] Role change: inline dropdown → save immediately with "Saving..." indicator
[ ] Remove member: confirmation dialog "Remove [Name] from workspace?"
[ ] Loading: skeleton rows · Empty: "No members yet." · Error: retry button
```

**Tab 2: Pending Invites** (admin/owner only — tab hidden for members)
```text
[ ] List of pending invites
[ ] Columns: Email · Role · Invited By · Expires On · Actions
[ ] Actions: Copy Link · Resend · Revoke
[ ] Copy Link: copies full accept URL to clipboard, "Link copied." toast
[ ] Resend: calls Edge Function again, "Invite resent." toast
[ ] Revoke: confirmation "Revoke invite to [email]?" → updates status to 'revoked'
[ ] Expired invites shown with "Expired" badge (not action buttons)
[ ] Loading: skeleton rows · Empty: "No pending invites." · Error: retry
```

```text
[ ] "Invite Member" button at top right — admin only
[ ] Tab "Pending Invites (3)" — shows count of pending invites
[ ] Tab URL param: ?tab=members (default) or ?tab=invites
```

---

## 5. React: Send Invite Modal

```text
Fields:
[ ] Email: email input, required
[ ] Role: dropdown — Member / Admin

Behavior:
[ ] Submit "Send Invite" → calls Edge Function
[ ] Loading state during call
[ ] On success: "Invite sent to [email]." toast, invites list refreshes
[ ] If already pending: "An invite to this email is already pending. Resend it?" with Resend button
[ ] If already a member: "This email is already a member of this workspace."
[ ] API error: banner above submit
```

---

## 6. React: Accept Invite Page

**URL:** `/accept-invite?token={token}`

```text
LOADING STATE:
[ ] Page loads, token extracted from URL
[ ] API call to validate the invite (GET /invites?token=xxx or check via RPC)
[ ] While loading: spinner + "Verifying your invite..."

VALID INVITE:
[ ] Show workspace name + invited role
[ ] Show invited email (user must be logged in with this email)
[ ] "Accept Invite" button → calls accept_workspace_invite RPC
[ ] On success: redirect to workspace dashboard with "Welcome to [Workspace]!" toast
[ ] If user not logged in: "Please sign in with [email] to accept this invite." + sign-in link

INVALID INVITE:
[ ] Token not found: "This invite link is invalid or has already been used."
[ ] Expired: "This invite has expired. Ask your admin to send a new one."
[ ] Wrong email: "This invite was sent to a different email address."
[ ] Already a member: "You are already a member of this workspace." + link to workspace
```

---

## What You Should NOT Do

```text
× Store the invite token as a sequential ID (use UUID for tokens)
× Use SECURITY INVOKER for accept_workspace_invite — it needs to write workspace_members
× Let the Edge Function skip the "already a member" check
× Let accept_workspace_invite succeed for an expired or already-accepted token
× Show the Pending Invites tab to member-role users
× Generate signed URLs for the invite link — it should be a plain token URL
× Skip the email match check (user must accept with the same email the invite was sent to)
```

---

## Checklists to Run (in order)

```text
[ ] New Table — workspace_invites table
[ ] RLS Policies — workspace_invites
[ ] Edge Function — send-workspace-invite
[ ] RPC Function — accept_workspace_invite
[ ] React Page Build — Members & Invites page
[ ] Modals & Dialogs — Send Invite modal
[ ] Loading States & Skeletons — members list + invites list
[ ] Error Handling — accept invite page (all invalid states)
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — Invites tab hidden for members
[ ] Delete & Destructive Actions — revoke invite, remove member
```

---

## Done When

```text
[ ] workspace_invites table: all columns, unique token index, correct RLS
[ ] Edge Function: validates JWT, checks admin role, prevents duplicates, sends email
[ ] Edge Function: returns 409 with invite details when invite already pending
[ ] accept_workspace_invite RPC: SECURITY DEFINER with auth.uid() as first line
[ ] RPC: validates token, status, expiry, email match — all 8 validation steps
[ ] RPC: creates workspace_member AND marks invite as accepted atomically
[ ] Members tab: role change saved immediately, remove member with confirmation
[ ] Invites tab: visible only to admin/owner
[ ] Invites tab: copy link, resend, revoke all work
[ ] Revoked invite: status = 'revoked' (not deleted)
[ ] Expired invites: show "Expired" badge, no action buttons
[ ] Accept invite page: handles all 5 states (loading/valid/expired/invalid/wrong-email)
[ ] Accept invite page: redirects to workspace on success
[ ] Accept invite page: prompts login if not authenticated
[ ] Tab state in URL: ?tab=members and ?tab=invites
[ ] Mobile tested at 375px and 768px
[ ] No silent failures — all errors shown in UI
```
