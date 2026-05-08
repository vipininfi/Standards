# Project — Team Member Directory

**Stack:** WeWeb + Supabase
**Estimated Time:** 8 hours
**Difficulty:** Beginner–Intermediate

---

## What You Are Building

A team member directory for a workspace. Admins can add, edit, and remove members. All workspace members can view the directory. Each member has a profile with a name, role, department, bio, and avatar photo stored in Supabase Storage.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | `members` table + RLS policies | 1h |
| 2 | `member-avatars` storage bucket + RLS | 30m |
| 3 | Members list page (search + department filter + pagination) | 1h 30m |
| 4 | Member detail page (full profile view) | 45m |
| 5 | Add / Edit member modal (shared form with avatar upload) | 2h |
| 6 | Delete member (confirmation dialog, admin only) | 45m |
| 7 | Loading, empty, and error states on all pages | 30m |

---

## 1. Database — `members` Table

### Schema

| Column | Type | Rules |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE |
| full_name | text | NOT NULL, max 100 chars |
| email | text | NOT NULL, unique per workspace |
| role | text | NOT NULL, one of: 'owner', 'admin', 'member' |
| department | text | nullable |
| bio | text | nullable, max 500 chars |
| avatar_path | text | nullable — path in storage bucket |
| status | text | NOT NULL, default 'active', one of: 'active', 'inactive' |
| created_at | timestamptz | default now() |
| updated_at | timestamptz | default now(), updated by trigger |

### Constraints

```sql
-- Unique email per workspace
UNIQUE (workspace_id, email)

-- Status must be valid
CHECK (status IN ('active', 'inactive'))

-- Role must be valid
CHECK (role IN ('owner', 'admin', 'member'))
```

### Indexes

```sql
CREATE INDEX idx_members_workspace_id ON members (workspace_id);
CREATE INDEX idx_members_department ON members (workspace_id, department);
CREATE INDEX idx_members_status ON members (workspace_id, status);
```

### Trigger

Create a `set_timestamps` trigger that sets `updated_at = now()` on every UPDATE.

### RLS Policies

```text
SELECT: workspace members can view all members in their workspace
INSERT: only workspace admins/owners can insert
UPDATE: only workspace admins/owners can update
DELETE: only workspace admins/owners can delete
```

**Run checklist:** [New Table](../../../../checklists/supabase/new-table.md) · [RLS Policies](../../../../checklists/supabase/rls-policies.md)

---

## 2. Storage — `member-avatars` Bucket

- Visibility: **Private**
- Allowed MIME types: `image/jpeg`, `image/png`, `image/webp`
- Max file size: 2 MB
- Path structure: `{workspace_id}/{member_id}/avatar.{ext}`

### RLS on storage.objects

```text
SELECT: workspace members can view avatars for their workspace (first path segment = workspace_id)
INSERT: workspace admins/owners can upload
UPDATE: workspace admins/owners can replace
DELETE: workspace admins/owners can delete
```

**Run checklist:** [Storage Bucket](../../../../checklists/supabase/storage-bucket.md)

---

## 3. Members List Page

**URL:** `/workspace/{id}/members`

### Requirements

```text
[ ] Page header: "Team Members" + "Add Member" button (admin only — hidden for non-admins)
[ ] Search input: searches by full_name and email — debounced 300ms — sent to Supabase
[ ] Filter: Department dropdown (all departments from backend, not hardcoded)
[ ] Filter: Status toggle (Active / All)
[ ] Results shown in a table or card grid (your choice — be consistent)
[ ] Columns (table): Avatar · Full Name · Role badge · Department · Status badge · Actions
[ ] Actions column: Edit (admin only) · Delete (admin only)
[ ] Role badge: owner=purple, admin=blue, member=grey
[ ] Status badge: active=green, inactive=grey
[ ] Pagination: 20 per page, "Showing X–Y of Z members"

STATES:
[ ] Loading: skeleton rows (matching page size)
[ ] Empty (no members): "No team members yet." + "Add your first member" button
[ ] Empty (filter/search): "No members match your search." + clear button
[ ] Error: "Could not load members." + retry button
```

**Run checklist:** [WeWeb Page Build](../../../../checklists/website/weweb-page.md) · [Loading States & Skeletons](../../../../checklists/frontend/loading-states-skeletons.md)

---

## 4. Member Detail Page

**URL:** `/workspace/{id}/members/{member_id}`

### Requirements

```text
[ ] Shows avatar (or initials fallback if no avatar)
[ ] Full name, role badge, department, status badge
[ ] Bio text (or "No bio added." if empty — not a blank space)
[ ] Email (clickable mailto link)
[ ] "Edit Member" button (admin only)
[ ] "Back to Members" link
[ ] Loading skeleton while data loads
[ ] Error state if member not found: "Member not found." + back link
```

---

## 5. Add / Edit Member Modal

A **single shared modal component** used for both Add and Edit. The only difference is the title and submit button label.

### Form Fields

| Field | Type | Rules |
|-------|------|-------|
| Full Name | text input | Required, max 100 chars |
| Email | email input | Required, valid email format, unique in workspace (check on submit) |
| Role | dropdown | Required, options: Admin / Member (Owner cannot be set via form) |
| Department | text input | Optional, max 100 chars |
| Bio | textarea | Optional, max 500 chars, show character count |
| Avatar | file upload | Optional, JPG/PNG/WebP only, max 2 MB, show preview |
| Status | toggle (Active/Inactive) | Default: Active, visible in Edit mode only |

### Behavior

```text
ADD MODE:
[ ] Modal title: "Add Member"
[ ] All fields empty on open
[ ] Submit button: "Add Member" — shows loading state during submit
[ ] On success: close modal, show "Member added." toast, list refreshes

EDIT MODE:
[ ] Modal title: "Edit Member"
[ ] All fields pre-filled with current member data
[ ] Avatar shows current avatar (or initials)
[ ] Submit button: "Save Changes" — shows loading state during submit
[ ] On success: close modal, show "Changes saved." toast, list refreshes

BOTH MODES:
[ ] X close button + Escape key close the modal
[ ] If email already exists: show "A member with this email already exists." error below email field
[ ] API error: show banner above submit button (not a toast)
[ ] Loading during submit: disable all fields and close button
```

**Run checklist:** [Add/Edit Consistency](../../../../checklists/frontend/add-edit-consistency.md) · [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md) · [Notifications & Toasts](../../../../checklists/frontend/notifications-toasts.md)

---

## 6. Delete Member

```text
[ ] Delete button visible only to admins (hidden for member role)
[ ] Clicking Delete opens a confirmation modal:
    Title:  "Remove [Full Name]?"
    Body:   "[Full Name] will be removed from the workspace and lose access immediately."
    Body:   "This action cannot be undone."
    Buttons: "Cancel" (left) · "Remove Member" red (right)
[ ] Confirm button shows loading state during delete
[ ] On success: modal closes, "Member removed." toast shown, row disappears without page refresh
[ ] On error: show error banner in the modal, keep modal open
```

**Run checklist:** [Delete & Destructive Actions](../../../../checklists/frontend/delete-destructive-actions.md)

---

## 7. Permissions Summary

```text
| Action | Member | Admin | Owner |
|--------|--------|-------|-------|
| View list | ✓ | ✓ | ✓ |
| View detail | ✓ | ✓ | ✓ |
| Add member | ✗ | ✓ | ✓ |
| Edit member | ✗ | ✓ | ✓ |
| Delete member | ✗ | ✓ | ✓ |
| Set role to Admin | ✗ | ✓ | ✓ |

[ ] "Add Member" button hidden for member role — not just disabled
[ ] Edit and Delete buttons hidden in table rows for member role
[ ] Backend (RLS) enforces all of the above — frontend is display only
```

**Run checklist:** [Permissions & Role-Based UI](../../../../checklists/frontend/permissions-role-ui.md)

---

## What You Should NOT Do

```text
× Hardcode department options — fetch from the backend (distinct departments query)
× Use a public storage bucket for avatars — they contain user data
× Store signed avatar URLs in the database — generate them at render time
× Duplicate the Add and Edit form — one shared component only
× Filter members client-side — all search/filter must go to Supabase
× Show a blank area for empty states — always show a message
× Skip loading skeletons — every data fetch needs a skeleton, not a spinner
× Let the delete work without a confirmation dialog
× Show the Add/Edit/Delete buttons to a member-role user
```

---

## Checklists to Run (in order)

```text
[ ] New Table — members table
[ ] Database Trigger — updated_at trigger
[ ] RLS Policies — members table
[ ] Storage Bucket — member-avatars bucket
[ ] WeWeb Page Build — members list page
[ ] WeWeb Page Build — member detail page
[ ] Add/Edit Consistency — member modal
[ ] Modals & Dialogs — member modal
[ ] Loading States & Skeletons — list and detail pages
[ ] Delete & Destructive Actions — delete member
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — all pages and actions
```

---

## Done When

```text
[ ] members table created with all columns, constraints, indexes
[ ] updated_at trigger works: UPDATE sets updated_at automatically
[ ] RLS: member can read; only admin/owner can write — tested for both roles
[ ] member-avatars storage bucket: private, MIME restricted, RLS enforced
[ ] Members list: search and department filter sent to Supabase (not client-side)
[ ] Members list: all 4 states work (loading/empty/empty-filter/error)
[ ] Members list: pagination shows "Showing X–Y of Z"
[ ] Member detail: loading skeleton and not-found error state work
[ ] Add/Edit: same component used for both modes
[ ] Add/Edit: pre-fill works in edit mode
[ ] Add/Edit: duplicate email error shown as field error (not toast)
[ ] Add/Edit: loading state during submit, button disabled
[ ] Avatar upload: MIME type validated, 2 MB limit enforced, preview shown
[ ] Signed URL generated for avatar display (not stored public URL)
[ ] Delete: confirmation dialog shown, loading state on confirm button
[ ] Delete: row removed from list without page refresh
[ ] Add/Edit/Delete buttons hidden for member-role users
[ ] Backend RLS tested: member cannot INSERT/UPDATE/DELETE via Supabase directly
[ ] Mobile tested at 375px and 768px
[ ] No silent failures — every error shown in UI
```
