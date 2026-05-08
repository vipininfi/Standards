# Project — Task Board

**Stack:** WeWeb + Supabase
**Estimated Time:** 8 hours
**Difficulty:** Intermediate

---

## What You Are Building

A workspace task board where members can create, assign, and track tasks. Tasks have a status, priority, due date, and assignee. Admins can delete tasks. All members can create and update task status. The list supports search, filtering by status and assignee, sorting, and pagination. Bulk status update is available for admins.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | `tasks` table + RLS + RPC functions | 1h 30m |
| 2 | Task list page (search, filters, sort, pagination) | 1h 30m |
| 3 | Add / Edit task modal (shared form) | 1h 30m |
| 4 | Inline status update (per row) | 30m |
| 5 | Delete task (confirmation, admin only) | 30m |
| 6 | Bulk status update (select + action bar) | 1h |
| 7 | All states + permissions | 30m |

---

## 1. Database

### `tasks` Table

| Column | Type | Rules |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE |
| title | text | NOT NULL, max 200 chars |
| description | text | nullable |
| status | text | NOT NULL, default 'open', one of: 'open', 'in-progress', 'done', 'cancelled' |
| priority | text | NOT NULL, default 'medium', one of: 'low', 'medium', 'high', 'urgent' |
| assignee_id | uuid | FK → auth.users.id, ON DELETE SET NULL, nullable |
| due_date | date | nullable |
| created_by | uuid | FK → auth.users.id, ON DELETE SET NULL |
| created_at | timestamptz | default now() |
| updated_at | timestamptz | default now(), updated by trigger |

### Constraints

```sql
CHECK (status IN ('open', 'in-progress', 'done', 'cancelled'))
CHECK (priority IN ('low', 'medium', 'high', 'urgent'))
```

### Indexes

```sql
CREATE INDEX idx_tasks_workspace_id ON tasks (workspace_id);
CREATE INDEX idx_tasks_status ON tasks (workspace_id, status);
CREATE INDEX idx_tasks_assignee ON tasks (workspace_id, assignee_id);
CREATE INDEX idx_tasks_due_date ON tasks (workspace_id, due_date);
```

### RLS Policies

```text
SELECT: workspace members can view all tasks in their workspace
INSERT: workspace members can insert (created_by set via RLS WITH CHECK)
UPDATE: workspace members can update tasks in their workspace
DELETE: only workspace admins/owners can delete
```

### RPC Functions

**`create_task`** — creates a task and writes to audit_logs

```sql
Parameters: p_workspace_id uuid, p_title text, p_description text DEFAULT NULL,
            p_priority text DEFAULT 'medium', p_assignee_id uuid DEFAULT NULL,
            p_due_date date DEFAULT NULL
Validation:
  1. Auth check
  2. title not blank, max 200 chars
  3. priority IN ('low','medium','high','urgent')
  4. workspace membership check
  5. assignee_id exists in workspace_members if provided
Returns: json — { "data": { task row } }
```

**`update_task_status`** — updates status of one or more tasks atomically

```sql
Parameters: p_task_ids uuid[], p_new_status text
Validation:
  1. Auth check
  2. p_new_status is valid
  3. all task_ids exist in the caller's workspace
Returns: json — { "updated_count": N }
```

**Run checklist:** [New Table](../../../../checklists/supabase/new-table.md) · [RLS Policies](../../../../checklists/supabase/rls-policies.md) · [RPC Function](../../../../checklists/supabase/rpc-function.md)

---

## 2. Task List Page

**URL:** `/workspace/{id}/tasks`

### Requirements

```text
HEADER
[ ] Page title: "Tasks"
[ ] "Add Task" button — visible to all members

SEARCH & FILTERS
[ ] Search input: searches task title — debounced 300ms — sent to Supabase
[ ] Filter: Status — dropdown (All / Open / In Progress / Done / Cancelled)
[ ] Filter: Priority — dropdown (All / Low / Medium / High / Urgent)
[ ] Filter: Assignee — dropdown (All / Me / specific members from backend)
[ ] Active filters shown as chips above the table: "Status: Open ×"
[ ] "Clear all filters" button appears when any filter is active
[ ] All filters sent to Supabase — never client-side array filtering
[ ] Filter state preserved in URL: ?status=open&priority=high&assignee=me

SORT
[ ] Default sort: created_at DESC (newest first)
[ ] Sortable columns: Title, Priority, Due Date, Created At
[ ] Sort state preserved in URL: ?sort=due_date&order=asc

TABLE COLUMNS
[ ] Checkbox (for bulk) · Title · Status badge · Priority badge · Assignee avatar · Due Date · Actions

BADGES
[ ] Status: open=grey, in-progress=blue, done=green, cancelled=grey-strikethrough
[ ] Priority: low=grey, medium=amber, high=orange, urgent=red

PAGINATION
[ ] 25 per page, "Showing X–Y of Z tasks"
[ ] Page state in URL: ?page=2

STATES
[ ] Loading: skeleton rows
[ ] Empty (no tasks): "No tasks yet." + "Add your first task" button
[ ] Empty (filter/search): "No tasks match your filters." + clear button
[ ] Error: "Could not load tasks." + retry
```

**Run checklist:** [WeWeb Page Build](../../../../checklists/website/weweb-page.md) · [Search, Filters & Pagination](../../../../checklists/frontend/search-filters-pagination.md)

---

## 3. Add / Edit Task Modal

One shared component. Add mode = empty form. Edit mode = pre-filled.

### Form Fields

| Field | Type | Rules |
|-------|------|-------|
| Title | text | Required, max 200 chars |
| Description | textarea | Optional |
| Priority | dropdown | Required, default: Medium |
| Assignee | searchable dropdown | Optional — fetches workspace members |
| Due Date | date picker | Optional, min: today |
| Status | dropdown | Required — visible in Edit mode only |

### Behavior

```text
ADD MODE:
[ ] Title: "Add Task" · Submit: "Add Task"
[ ] Status field hidden (always starts as 'open')
[ ] On success: "Task added." toast, list refreshes with new task

EDIT MODE:
[ ] Title: "Edit Task" · Submit: "Save Changes"
[ ] All fields pre-filled from current task data
[ ] Status field visible — can be changed here
[ ] On success: "Changes saved." toast, list refreshes

BOTH MODES:
[ ] Loading state on submit button, all fields disabled during submit
[ ] API error: banner above submit button
[ ] Escape + X close button work
[ ] Focus moves to first field on open
```

**Run checklist:** [Add/Edit Consistency](../../../../checklists/frontend/add-edit-consistency.md) · [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## 4. Inline Status Update

```text
[ ] Status badge in each row is clickable — opens a small dropdown with status options
[ ] Selecting a new status calls update_task_status RPC immediately
[ ] Loading state: badge shows spinner while updating
[ ] On success: badge updates in place — no full page reload
[ ] On error: badge reverts to previous status, error toast shown
[ ] Disabled during bulk action operation
```

---

## 5. Delete Task (Admin Only)

```text
[ ] Delete action in the row's action menu — visible only to admins
[ ] Confirmation dialog:
    Title:  "Delete Task?"
    Body:   "'[Task Title]' will be permanently deleted."
    Body:   "This action cannot be undone."
    Buttons: Cancel · "Delete Task" (red)
[ ] On confirm: row removed from list, "Task deleted." toast
[ ] On error: modal stays open with error banner
```

**Run checklist:** [Delete & Destructive Actions](../../../../checklists/frontend/delete-destructive-actions.md)

---

## 6. Bulk Status Update

```text
SELECTION
[ ] Checkbox in header: select/deselect all on current page
[ ] Checkbox per row
[ ] Selection count shown: "5 selected"

BULK ACTION BAR
[ ] Appears above table only when rows are selected
[ ] "Change Status:" dropdown (Open / In Progress / Done / Cancelled)
[ ] "Apply to 5 tasks" button — shows count
[ ] "Cancel" button clears selection

BEHAVIOR
[ ] Calls update_task_status RPC with array of selected task IDs
[ ] Loading state on Apply button during operation
[ ] On success: "5 tasks updated." toast, table refreshes, selection cleared
[ ] On error: error banner in action bar, selection preserved
[ ] Bulk action bar visible to all members (not admin-only — status update is allowed for all)
```

**Run checklist:** [Tables & Data Lists](../../../../checklists/frontend/tables-data-lists.md)

---

## 7. Permissions Summary

```text
| Action | Member | Admin | Owner |
|--------|--------|-------|-------|
| View tasks | ✓ | ✓ | ✓ |
| Add task | ✓ | ✓ | ✓ |
| Edit task | ✓ | ✓ | ✓ |
| Update status (inline + bulk) | ✓ | ✓ | ✓ |
| Delete task | ✗ | ✓ | ✓ |

[ ] Delete button/action hidden for member role — not just disabled
[ ] RLS enforced: member cannot DELETE directly in Supabase
```

---

## What You Should NOT Do

```text
× Filter tasks client-side — all filters, search, sort go to Supabase
× Make create_task and update_task_status two separate frontend calls — use RPC
× Skip the RPC and do INSERT directly from WeWeb for task creation (no audit log)
× Hardcode status or priority options — use WeWeb variables/constants
× Show a full page spinner when loading — use skeleton rows
× Forget to reset pagination to page 1 when any filter changes
× Show the Delete action to member-role users
× Send p_task_ids to update_task_status as individual calls in a loop (use array)
```

---

## Checklists to Run (in order)

```text
[ ] New Table — tasks table
[ ] Database Trigger — updated_at trigger
[ ] RLS Policies — tasks table
[ ] RPC Function — create_task
[ ] RPC Function — update_task_status
[ ] WeWeb Page Build — task list page
[ ] Search, Filters & Pagination — task list
[ ] Add/Edit Consistency — task modal
[ ] Modals & Dialogs — task modal
[ ] Tables & Data Lists — bulk actions
[ ] Loading States & Skeletons — list page
[ ] Delete & Destructive Actions — delete task
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — delete action visibility
```

---

## Done When

```text
[ ] tasks table created with all columns, constraints, indexes
[ ] updated_at trigger fires on every UPDATE
[ ] RLS: all members can SELECT/INSERT/UPDATE; only admin/owner can DELETE — tested
[ ] create_task RPC: validates all inputs, writes audit log, returns task data
[ ] update_task_status RPC: accepts array of IDs, updates all atomically
[ ] Task list: search + all filters sent to Supabase, URL params preserved
[ ] Task list: filter chips shown with X to remove each filter
[ ] Task list: sort works, pagination resets to page 1 on filter/sort change
[ ] Task list: all 4 states (loading/empty/empty-filter/error) work
[ ] Add/Edit: single shared component, pre-fill works in edit mode
[ ] Add/Edit: loading state on submit, all fields disabled during submit
[ ] Inline status update: no full page reload, loading indicator on badge
[ ] Bulk: selection count shown, Apply button calls RPC with all IDs at once
[ ] Delete: only visible to admin/owner, confirmation dialog shown
[ ] Mobile tested at 375px (table collapses correctly)
[ ] All filters/sort/page state in URL params
[ ] No client-side filtering anywhere
[ ] No silent failures — every error shown in UI
```
