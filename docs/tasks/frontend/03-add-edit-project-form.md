# Task: Build a Shared Add / Edit Project Form

**Platform:** React (or WeWeb)
**Covers:** [Add / Edit Consistency](../../checklists/frontend/add-edit-consistency.md) · [Form Field Validation](../../checklists/frontend/form-field-validation.md) · [Code Reusability Standards](../../standards/code-reusability-standards.md)

---

## Scenario

You are building the Project feature for **WorkFlow**. Projects can be created by workspace members (Add Project) and updated by the project creator or an admin (Edit Project). The Add and Edit flows must use the same form component and the same validation schema — no exceptions.

---

## What to Build

### 1. A shared form component: `ProjectForm`

This single component is used in both Add and Edit modes. It accepts a `mode` prop (`"add"` or `"edit"`) and an optional `initialValues` prop for pre-filling in edit mode.

### 2. An Add Project modal

Uses `ProjectForm` in `mode="add"`. Triggered by an "Add Project" button on the projects list page. Submits via `POST /projects` (or your backend equivalent).

### 3. An Edit Project page (or modal)

Uses `ProjectForm` in `mode="edit"`. Loads existing project data, pre-fills all fields, then submits via `PATCH /projects/:id`.

---

## Form Fields

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| Project Name | Text | Yes | Min 3 chars, max 100 chars |
| Description | Textarea | No | Max 500 chars, show character counter |
| Status | Dropdown | Yes | Values: Active, On Hold, Archived |
| Start Date | Date picker | Yes | Cannot be in the past (Add only — validate on submit) |
| End Date | Date picker | No | If provided, must be after Start Date |
| Tags | Multi-select | No | Free-entry tags, max 10 tags |

---

## Requirements

### Shared Component (`ProjectForm`)

```
props:
  mode: "add" | "edit"
  initialValues?: { name, description, status, startDate, endDate, tags }
  onSubmit: (data) => Promise<void>
```

- Identical field layout in both modes
- Same validation schema (Zod or Yup) imported and used in both modes
- Only these things differ between modes:
  - Page/modal title: "Add Project" vs "Edit Project"
  - Submit button label: "Create Project" vs "Save Changes"
  - HTTP method: POST vs PATCH
  - Start date validation: "cannot be in past" applies only in Add mode

### Validation Schema (same object — not duplicated)

```text
name:        required, min 3, max 100 characters
description: optional, max 500 characters
status:      required, must be one of: active, on-hold, archived
startDate:   required, valid date
             — In add mode: must not be in the past
             — In edit mode: can be any valid date (already started)
endDate:     optional, if provided must be after startDate
tags:        optional, max 10 items
```

### Error Messages

| Field | Message |
|-------|---------|
| Name empty | "Project name is required." |
| Name too short | "Project name must be at least 3 characters." |
| Name too long | "Project name must be 100 characters or fewer." |
| Status not selected | "Status is required." |
| Start date empty | "Start date is required." |
| Start date in past (Add only) | "Start date cannot be in the past." |
| End date before start | "End date must be after the start date." |
| Too many tags | "You can add up to 10 tags." |

### Edit Mode — Pre-Fill Behavior

- Show a skeleton loader while project data is loading — do NOT flash an empty form
- If data fails to load: show an error state with a "Try again" button — do NOT show an empty form
- All fields pre-filled with existing values when data loads
- If a field is disabled in edit mode (none in this task, but in general): show it as read-only with a tooltip explaining why

### States (Both Modes)

- **Default** — form ready, submit enabled
- **Loading (submit)** — spinner on button, button disabled, no double submit
- **Success (Add)** — close modal, show "Project created successfully." toast, refresh project list
- **Success (Edit)** — show "Project saved." toast, stay on page
- **Error** — inline field errors, API errors in a banner above the form, form stays populated

---

## What You Should NOT Do

- Do not create `AddProjectForm.tsx` and `EditProjectForm.tsx` as two separate components
- Do not duplicate the validation schema — one schema file, imported in both
- Do not make End Date required in Edit if it was optional in Add
- Do not let the Edit form render empty while data is loading
- Do not allow different error messages in Add vs Edit for the same field
- Do not call different API routes without changing the HTTP method (PATCH vs POST) appropriately

---

## Reusability Check Before You Start

```text
[ ] Does a ProjectForm component already exist? (if yes — extend it, do not create a new one)
[ ] Does a project validation schema already exist? (if yes — import it, do not duplicate)
[ ] Does a useProject hook already exist for loading project data? (if yes — use it)
[ ] Does a DeleteConfirmModal already exist? (you will need one for a delete button in Edit)
```

---

## Checklist to Run When Done

Use the [Add / Edit Consistency Checklist](../../checklists/frontend/add-edit-consistency.md#11-consistency-checklist--before-marking-done) in full.

Then run the [Frontend Checklist](../../checklists/frontend-checklist.md).

---

## Done When

```text
STRUCTURE
[ ] Single ProjectForm component used in both Add and Edit
[ ] Same validation schema imported in both modes
[ ] mode prop controls title, button label, and HTTP method only

FIELDS
[ ] All 6 fields present in both modes
[ ] Same field order in Add and Edit
[ ] Same labels, placeholders, and helper text

VALIDATION
[ ] name: required, min 3, max 100
[ ] description: optional, max 500 with character counter
[ ] status: required, only allowed values
[ ] startDate: required; past dates blocked in Add only
[ ] endDate: optional; if set, must be after startDate
[ ] tags: optional, max 10 items
[ ] Same error messages in both modes

EDIT MODE
[ ] Skeleton shown while loading data
[ ] Error state shown if data load fails (not empty form)
[ ] All fields pre-filled correctly
[ ] Dates and numbers pre-fill as correct types (not strings)

STATES
[ ] Loading spinner on submit button in both modes
[ ] No double submit possible in either mode
[ ] Success toast shown in both modes
[ ] Form stays populated on error in both modes
[ ] API error shown as banner in both modes

MOBILE
[ ] Add modal tested on mobile
[ ] Edit page tested on mobile
[ ] Date pickers usable on mobile
```
