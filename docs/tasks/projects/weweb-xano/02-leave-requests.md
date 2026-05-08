# Project — Leave Request System

**Stack:** WeWeb + Xano
**Estimated Time:** 8 hours
**Difficulty:** Intermediate

---

## What You Are Building

A leave request system where employees submit leave requests and managers approve or reject them. Employees see their own requests. Managers (admin role) see all pending requests and can approve or reject. The request has a status that moves through: draft → submitted → approved / rejected. Cancelled by employee before approval is also allowed.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Xano tables + endpoints (submit, list, approve, reject, cancel) | 2h |
| 2 | Employee: multi-step submit form (WeWeb) | 2h |
| 3 | Employee: My Requests list page | 1h |
| 4 | Manager: All Requests list page (with approve/reject) | 1h 30m |
| 5 | Status badges + all states | 30m |

---

## 1. Xano Database Tables

### `leave_requests` Table

| Column | Type | Rules |
|--------|------|-------|
| id | int (auto) | PK |
| workspace_id | int | FK → workspaces.id |
| employee_id | int | FK → users.id, set from auth token |
| leave_type | enum | required: 'annual', 'sick', 'personal', 'unpaid' |
| start_date | date | required |
| end_date | date | required, must be >= start_date |
| total_days | int | computed: end_date - start_date + 1 |
| reason | text | optional, max 500 chars |
| status | enum | required: 'submitted', 'approved', 'rejected', 'cancelled', default: 'submitted' |
| reviewed_by | int | FK → users.id, nullable — set on approve/reject |
| reviewed_at | timestamp | nullable |
| review_notes | text | optional — manager note on approve/reject |
| created_at | timestamp | auto |

### `leave_types` Reference Table (optional — or use hardcoded enum)

```text
annual    → Annual Leave
sick      → Sick Leave
personal  → Personal Leave
unpaid    → Unpaid Leave
```

---

## 2. Xano Endpoints

### `POST /api/leave-requests` — Submit Request

```text
Auth: Required (any workspace member)
Body: { workspace_id, leave_type, start_date, end_date, reason }

Validation (in order):
  1. Auth check
  2. Workspace membership check (any member via check_workspace_permission)
  3. leave_type is valid
  4. start_date is not in the past
  5. end_date >= start_date
  6. total_days > 0
  7. No overlapping approved/submitted request for same employee:
     check no existing request where status IN ('submitted','approved')
     AND date ranges overlap

Logic:
  Set employee_id from auth token
  Set total_days = end_date - start_date + 1
  Set status = 'submitted'
  Insert record

Error codes: VALIDATION_ERROR, BUSINESS_RULE_VIOLATION
Response: { "data": { request } }
```

### `GET /api/leave-requests/my` — My Requests

```text
Auth: Required
Query params: status (optional filter), page, per_page (default 20)
Logic:
  1. Auth check
  2. Return requests WHERE employee_id = auth_user_id, ordered by created_at DESC
Response: { "data": [...], "total": N }
```

### `GET /api/leave-requests` — All Requests (Admin/Manager)

```text
Auth: Required — admin/owner role only
Query params: workspace_id, status (optional), employee_id (optional), page, per_page
Logic:
  1. Auth check
  2. check_workspace_permission(required_role: 'admin')
  3. Return all requests for workspace with filters
Response: { "data": [...], "total": N }
```

### `POST /api/leave-requests/{id}/approve` — Approve

```text
Auth: Required — admin/owner only
Body: { review_notes (optional) }
Validation:
  1. Auth + workspace admin permission
  2. Request exists and belongs to workspace
  3. Status must be 'submitted' (cannot approve already-approved or rejected)
Logic:
  Update status = 'approved', reviewed_by = auth_user_id, reviewed_at = now()
Response: { "data": { updated request } }
Error: BUSINESS_RULE_VIOLATION if not in 'submitted' status
```

### `POST /api/leave-requests/{id}/reject` — Reject

```text
Same structure as approve, but status = 'rejected'
Body: { review_notes (required on reject — reason must be given) }
Validation: review_notes required for rejection
```

### `POST /api/leave-requests/{id}/cancel` — Cancel (Employee)

```text
Auth: Required
Validation:
  1. Auth check
  2. Request belongs to auth user (not just workspace — only own requests)
  3. Status must be 'submitted' (cannot cancel approved/rejected)
Logic: Update status = 'cancelled'
Error: FORBIDDEN if not own request; BUSINESS_RULE_VIOLATION if not submitted
```

**Run checklist:** [New API Endpoint (Xano)](../../../../checklists/xano/new-endpoint.md) for each endpoint · [Reusable Function](../../../../checklists/xano/reusable-function.md)

---

## 3. Employee: Multi-Step Submit Form

A 3-step WeWeb form to submit a leave request.

### Step 1 — Leave Type & Dates

```text
Fields:
  [ ] Leave Type: radio buttons or dropdown (Annual / Sick / Personal / Unpaid)
  [ ] Start Date: date picker, min: today
  [ ] End Date: date picker, min: start_date (dynamically updates when start_date changes)
  [ ] Total Days: computed display (end - start + 1), shown read-only

Validation before Next:
  [ ] Leave type selected
  [ ] Start date not in the past
  [ ] End date >= start date
```

### Step 2 — Reason

```text
Fields:
  [ ] Reason: textarea, optional, max 500 chars, show character count
  [ ] Summary card: shows selected type, dates, total days (read-only recap)

Validation before Next:
  [ ] No validation required — reason is optional
```

### Step 3 — Review & Submit

```text
[ ] Full summary of all entered data
[ ] "Back" button goes to Step 2 (data preserved)
[ ] "Submit Request" button
[ ] Loading state on Submit button during API call
[ ] On success: redirect to My Requests page, "Leave request submitted." toast
[ ] On error (e.g., overlapping request): error banner on this step
```

### Progress Indicator

```text
[ ] 3 steps shown at top: Leave Details → Reason → Review
[ ] Active step highlighted
[ ] Completed steps shown as ticked
[ ] Clicking a previous step goes back (data preserved)
[ ] Step 3 cannot be navigated to directly from Step 1
```

**Run checklist:** [Multi-Step Forms](../../../../checklists/frontend/multi-step-forms.md) · [WeWeb Page Build](../../../../checklists/website/weweb-page.md)

---

## 4. Employee: My Requests Page

**URL:** `/workspace/{id}/my-leaves`

```text
[ ] List of the logged-in employee's own requests only
[ ] Columns: Leave Type · Start Date · End Date · Days · Status badge · Actions
[ ] Actions: Cancel button (only shown when status = 'submitted')
[ ] Filter: Status dropdown
[ ] Status badges: submitted=blue, approved=green, rejected=red, cancelled=grey
[ ] Cancel: confirmation dialog "Cancel this leave request?" + "Yes, Cancel" button
[ ] Pagination: 20 per page
[ ] Empty state: "You have no leave requests." + "Submit a Request" button

STATES:
[ ] Loading skeleton · Empty · Error with retry
```

---

## 5. Manager: All Requests Page

**URL:** `/workspace/{id}/admin/leaves`

```text
[ ] Visible only to admin/owner role — 403 state for members
[ ] List of all workspace requests
[ ] Filter: Status · Employee (dropdown of members) · Leave Type
[ ] Active filter chips with X to remove
[ ] Columns: Employee Name · Leave Type · Dates · Days · Submitted On · Status badge · Actions
[ ] Actions per row (when status = 'submitted'):
    "Approve" (green button) · "Reject" (red button)
[ ] Clicking Approve: confirmation "Approve [Name]'s [N]-day Annual Leave?"
[ ] Clicking Reject: opens a small modal asking for rejection reason (required)
[ ] After approve/reject: row status badge updates in place, action buttons disappear
[ ] Pagination: 25 per page
[ ] "Showing X–Y of Z requests" total count

STATES: Loading · Empty · Empty (filter) · Error
```

**Run checklist:** [Permissions & Role-Based UI](../../../../checklists/frontend/permissions-role-ui.md) · [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## What You Should NOT Do

```text
× Skip the overlapping request check — two approved requests for the same days must be blocked
× Let the cancel endpoint succeed for a request that belongs to another user
× Let the approve/reject endpoint succeed for a member-role user
× Let the employee see all workspace requests — only their own in "my" endpoint
× Submit the form all at once on Step 1 — each step must validate before advancing
× Clear form data when user clicks Back (data must be preserved)
× Show the manager view to a member-role WeWeb user (check role on page load)
```

---

## Checklists to Run (in order)

```text
[ ] Reusable Function — check_workspace_permission
[ ] New API Endpoint (Xano) — POST /api/leave-requests (submit)
[ ] New API Endpoint (Xano) — GET /api/leave-requests/my
[ ] New API Endpoint (Xano) — GET /api/leave-requests (admin)
[ ] New API Endpoint (Xano) — POST approve / reject / cancel
[ ] Multi-Step Forms — submit leave form
[ ] WeWeb Page Build — My Requests page
[ ] WeWeb Page Build — Manager All Requests page
[ ] Modals & Dialogs — reject modal, cancel confirmation
[ ] Loading States & Skeletons — all list pages
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — manager page access
```

---

## Done When

```text
[ ] leave_requests table created with all columns
[ ] check_workspace_permission used in all endpoints
[ ] Submit endpoint: overlapping request check works correctly
[ ] Submit endpoint: start_date not in past validated
[ ] Submit endpoint: end_date >= start_date validated
[ ] My Requests endpoint: returns only the auth user's own requests
[ ] All Requests endpoint: FORBIDDEN for member role
[ ] Approve/reject: only works when status = 'submitted'
[ ] Cancel: only works for own request AND only when status = 'submitted'
[ ] Multi-step form: 3 steps with progress indicator
[ ] Multi-step form: data preserved when navigating back
[ ] Multi-step form: Step 1 validates dates before advancing
[ ] My Requests: cancel only shown for submitted requests
[ ] Manager page: 403 state for member-role users
[ ] Manager page: approve/reject buttons disappear after action
[ ] Rejection: review_notes required in Xano endpoint
[ ] All list pages: 4 states work (loading/empty/empty-filter/error)
[ ] All filters sent to Xano — no client-side filtering
[ ] Mobile tested at 375px and 768px
```
