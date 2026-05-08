# Project — Contact Book

**Stack:** WeWeb + Xano
**Estimated Time:** 8 hours
**Difficulty:** Beginner–Intermediate

---

## What You Are Building

A workspace contact book (light CRM). Members can add, view, edit, and delete contacts. Contacts have a name, email, phone, company, status, and notes. The list is searchable, filterable, and paginated. All backend logic lives in Xano with proper auth, permission checks, and error handling.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Xano tables + `check_workspace_permission` reusable function | 1h |
| 2 | Xano endpoints: list, get, create, update, delete | 1h 30m |
| 3 | Contacts list page (search, filter, sort, pagination) | 1h 30m |
| 4 | Add / Edit contact modal (shared form) | 1h 30m |
| 5 | Contact detail view (side panel or page) | 30m |
| 6 | Delete contact (confirmation, admin only) | 30m |
| 7 | All states + permissions | 30m |

---

## 1. Xano Database Tables

### `contacts` Table

| Column | Type | Rules |
|--------|------|-------|
| id | int (auto) | PK |
| workspace_id | int | FK → workspaces.id, required |
| full_name | text | required, max 100 chars |
| email | text | optional, valid email format if provided |
| phone | text | optional, E.164 format if provided |
| company | text | optional, max 100 chars |
| status | enum | required, one of: 'lead', 'active', 'inactive', default: 'lead' |
| notes | text | optional, max 1000 chars |
| created_by | int | FK → users.id, set from auth token |
| created_at | timestamp | auto |
| updated_at | timestamp | auto |

---

## 2. Xano Reusable Function

### `check_workspace_permission`

Before building endpoints, extract this reusable function:

```text
Inputs:
  workspace_id (integer, required)
  user_id (integer, required)
  required_role (text, optional) — 'member', 'admin', 'owner'

Logic:
  1. Query workspace_members WHERE workspace_id = input AND user_id = input AND status = 'active'
  2. If not found: RAISE FORBIDDEN "You are not a member of this workspace."
  3. If required_role provided: check role hierarchy — RAISE FORBIDDEN if insufficient
  4. Return member record

Role hierarchy: member < admin < owner
```

**Run checklist:** [Reusable Function](../../../../checklists/xano/reusable-function.md)

---

## 3. Xano Endpoints

All endpoints follow this structure:
1. **Auth Token** — validate as the first precondition
2. **check_workspace_permission** — called with workspace_id from the request
3. **Input validation** — validate all required fields
4. **Business logic** — query/mutate the database
5. **Response** — standard format

### `GET /api/contacts` — List Contacts

```text
Auth: Required
Input query params:
  workspace_id (required)
  search (optional) — searches full_name, email, company
  status (optional) — filter by status
  sort (optional) — 'full_name', 'created_at', 'company' (default: created_at DESC)
  page (optional, default: 1)
  per_page (optional, default: 25, max: 100)

Logic:
  1. Auth + workspace permission check (any member)
  2. Build query with filters applied
  3. Return paginated results

Response:
  { "data": [...], "total": 143, "page": 1, "per_page": 25 }
```

### `GET /api/contacts/{id}` — Get Single Contact

```text
Auth: Required
Logic:
  1. Auth + workspace permission (any member)
  2. Verify contact belongs to workspace (NOT_FOUND if not)
  3. Return contact
```

### `POST /api/contacts` — Create Contact

```text
Auth: Required
Body: { workspace_id, full_name, email, phone, company, status, notes }
Validation (in order):
  1. full_name: required, max 100 chars
  2. email: valid format if provided
  3. phone: E.164 format if provided
  4. status: one of valid values
  5. Workspace permission (any member)
  6. No duplicate email in same workspace (if email provided)
Logic: Insert contact, set created_by from auth token
Response: { "data": { contact } }
Error codes: VALIDATION_ERROR, DUPLICATE, FORBIDDEN
```

### `PUT /api/contacts/{id}` — Update Contact

```text
Auth: Required
Same validation as create (except workspace check — verify contact ownership)
Logic: Update contact, set updated_at
Response: { "data": { contact } }
```

### `DELETE /api/contacts/{id}` — Delete Contact

```text
Auth: Required
Permission: admin or owner role only
Logic: Verify contact in workspace, delete record
Response: { "message": "Contact deleted." }
Error codes: FORBIDDEN, NOT_FOUND
```

**Run checklist:** [New API Endpoint (Xano)](../../../../checklists/xano/new-endpoint.md) for each endpoint

---

## 4. Contacts List Page

**URL:** `/workspace/{id}/contacts`

### Requirements

```text
HEADER
[ ] "Contacts" title + "Add Contact" button (all members can add)

SEARCH & FILTERS
[ ] Search: searches full_name, email, company — debounced 300ms — sent to Xano
[ ] Filter: Status dropdown (All / Lead / Active / Inactive)
[ ] Active filter chips shown with X to remove
[ ] "Clear all" button when any filter is active
[ ] Filter/search state in URL params: ?search=acme&status=active

SORT
[ ] Sort dropdown or column headers: Name A–Z / Name Z–A / Newest / Oldest
[ ] Sort state in URL: ?sort=full_name&order=asc

TABLE COLUMNS
[ ] Full Name · Company · Email · Phone · Status badge · Actions

STATUS BADGES
[ ] lead=blue, active=green, inactive=grey

PAGINATION
[ ] 25 per page, "Showing X–Y of Z contacts"
[ ] Page in URL: ?page=2
[ ] Pagination resets to 1 when search/filter changes

STATES
[ ] Loading: skeleton rows
[ ] Empty (no contacts): "No contacts yet." + "Add your first contact" button
[ ] Empty (filter/search): "No contacts match your search." + clear button
[ ] Error: "Could not load contacts." + retry button
```

**Run checklist:** [WeWeb Page Build](../../../../checklists/website/weweb-page.md) · [Search, Filters & Pagination](../../../../checklists/frontend/search-filters-pagination.md)

---

## 5. Contact Detail Panel / Page

A side panel or separate page showing the full contact details.

```text
[ ] All fields shown (including notes)
[ ] "Edit Contact" button (any member)
[ ] "Delete Contact" button (admin/owner only — hidden for member role)
[ ] Loading skeleton while fetching
[ ] Error if contact not found: "Contact not found." + back link
```

---

## 6. Add / Edit Contact Modal

One shared WeWeb component used for both Add and Edit.

### Form Fields

| Field | Type | Rules |
|-------|------|-------|
| Full Name | text | Required, max 100 chars |
| Email | email | Optional, valid format |
| Phone | tel + country code | Optional, E.164 stored |
| Company | text | Optional, max 100 chars |
| Status | dropdown | Required, default: Lead |
| Notes | textarea | Optional, max 1000 chars, show character count |

### Behavior

```text
ADD MODE: title "Add Contact", submit "Add Contact"
EDIT MODE: title "Edit Contact", pre-filled fields, submit "Save Changes"

BOTH:
[ ] Duplicate email error shown below email field (not a toast)
[ ] Submit button: loading state, disabled during request
[ ] X + Escape close modal (confirm if dirty)
[ ] On success: toast notification, list refreshes
[ ] On API error: red banner above submit button
```

**Run checklist:** [Add/Edit Consistency](../../../../checklists/frontend/add-edit-consistency.md) · [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## 7. Delete Contact

```text
[ ] Admin/owner only — button hidden for member role
[ ] Confirmation dialog:
    Title:  "Delete Contact?"
    Body:   "'[Full Name]' will be permanently deleted."
    Buttons: Cancel · "Delete Contact" (red)
[ ] Loading state on confirm button
[ ] On success: row removed, "Contact deleted." toast
```

**Run checklist:** [Delete & Destructive Actions](../../../../checklists/frontend/delete-destructive-actions.md)

---

## What You Should NOT Do

```text
× Put permission logic in WeWeb — check_workspace_permission runs in Xano
× Filter contacts client-side — all search/filter sent to Xano
× Use separate Add and Edit components
× Let DELETE endpoint succeed for a member-role user
× Skip the phone format — store E.164 even though display is formatted
× Show delete button to member-role users
× Return 500 for a DUPLICATE email — return a specific error code
```

---

## Checklists to Run (in order)

```text
[ ] Reusable Function — check_workspace_permission
[ ] New API Endpoint (Xano) — GET /api/contacts (list)
[ ] New API Endpoint (Xano) — POST /api/contacts (create)
[ ] New API Endpoint (Xano) — PUT /api/contacts/{id} (update)
[ ] New API Endpoint (Xano) — DELETE /api/contacts/{id} (delete)
[ ] WeWeb Page Build — contacts list page
[ ] Search, Filters & Pagination — contacts list
[ ] Add/Edit Consistency — contact modal
[ ] Modals & Dialogs — contact modal
[ ] Loading States & Skeletons — list page
[ ] Delete & Destructive Actions — delete contact
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — delete visibility
```

---

## Done When

```text
[ ] contacts table created in Xano with all fields
[ ] check_workspace_permission reusable function works with role hierarchy
[ ] Auth token validation is the FIRST step in every endpoint
[ ] List endpoint: search, filter, sort, pagination all work from Xano
[ ] Create endpoint: validation in correct order, duplicate email caught, created_by set from token
[ ] Update endpoint: validates, checks contact is in caller's workspace
[ ] Delete endpoint: FORBIDDEN for member role
[ ] Contacts list: search + filters go to Xano (not client-side)
[ ] Contacts list: URL params preserved for search/filter/sort/page
[ ] Contacts list: pagination resets to page 1 on filter change
[ ] Contacts list: all 4 states work (loading/empty/empty-filter/error)
[ ] Add/Edit: one shared WeWeb component — pre-fill in edit mode works
[ ] Duplicate email: shown as field error below email input
[ ] Delete: admin/owner only, confirmation dialog required
[ ] Phone: E.164 format enforced in Xano, formatted display in WeWeb
[ ] Mobile tested at 375px and 768px
[ ] No silent failures — every Xano error shown in UI
```
