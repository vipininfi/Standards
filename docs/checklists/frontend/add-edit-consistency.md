# Add / Edit Consistency Checklist

> **Core Rule:** Add and Edit for the same entity must be identical in behavior, validation, layout, and error handling. A user should not be able to do something in Edit that Add prevents, or vice versa.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-consistency-rule) | The Consistency Rule |
| [2](#2-shared-component-requirement) | Shared Component Requirement |
| [3](#3-field-consistency) | Field Consistency |
| [4](#4-validation-consistency) | Validation Consistency |
| [5](#5-ui-and-layout-consistency) | UI and Layout Consistency |
| [6](#6-state-consistency) | State Consistency |
| [7](#7-permission-consistency) | Permission Consistency |
| [8](#8-pre-fill-behavior-edit-only) | Pre-Fill Behavior (Edit Only) |
| [9](#9-submit-behavior) | Submit Behavior |
| [10](#10-common-violations--anti-patterns) | Common Violations & Anti-Patterns |
| [11](#11-consistency-checklist--before-marking-done) | Consistency Checklist — Before Marking Done |

---

# 1. The Consistency Rule

A team member looking at an Add form and an Edit form for the same entity (e.g., Add Project / Edit Project) must see:

- The **same fields**
- The **same validation rules**
- The **same error messages**
- The **same layout and structure**
- The **same loading and success behavior**

The only differences allowed:

| Difference | Allowed | Why |
|------------|---------|-----|
| Page/modal title ("Add Project" vs "Edit Project") | Yes | Context label |
| Submit button label ("Create" vs "Save Changes") | Yes | Action label |
| Pre-filled values in Edit | Yes | Existing data loaded |
| Some fields disabled in Edit | Only if intentional business rule | Documented reason required |
| Additional fields in Edit only | Only if they don't exist at creation time | e.g., "Created at", "Status" after creation |

---

# 2. Shared Component Requirement

The Add and Edit forms for the same entity **must use the same component** — not two copies.

```text
Good:
  components/forms/ProjectForm.tsx
    ↳ Used in AddProjectModal.tsx (mode="add")
    ↳ Used in EditProjectPage.tsx (mode="edit")

Bad:
  components/AddProjectForm.tsx    ← different validation
  components/EditProjectForm.tsx   ← different layout
```

## How to structure a shared form component

```tsx
// ProjectForm.tsx
interface ProjectFormProps {
  mode: 'add' | 'edit';
  initialValues?: Partial<Project>;  // only provided in edit mode
  onSubmit: (data: ProjectFormData) => Promise<void>;
}

// Same fields, same validation schema, same layout for both modes
// Only title and submit label differ based on mode
```

## Shared validation schema

```ts
// validations/project.schema.ts
export const projectSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  description: z.string().max(500).optional(),
  status: z.enum(['active', 'inactive']),
});

// Used in both AddProject and EditProject — never duplicated
```

---

# 3. Field Consistency

## Required vs Optional

```text
[ ] A field marked required in Add must be required in Edit
[ ] A field optional in Add must be optional in Edit
[ ] Exception: fields that only exist after creation (e.g., ID, created_at) are display-only in Edit
```

## Field Presence

```text
[ ] Every field shown in Add is shown in Edit (unless there is a documented reason it should not be)
[ ] Every field shown in Edit was present in Add (edit should not introduce new "hidden" fields)
[ ] If a field is intentionally absent in Edit — document the business rule in the ticket
```

## Field Order

```text
[ ] Fields appear in the same order in both Add and Edit
[ ] Groups and sections are in the same order
```

---

# 4. Validation Consistency

This is the most critical consistency requirement. Validation that differs between Add and Edit allows users to bypass rules.

## Rules

```text
[ ] Same validation schema object is used for both Add and Edit
[ ] Min/max character limits are identical in both modes
[ ] Required/optional rules are identical in both modes
[ ] Format rules (email, phone, URL) are identical in both modes
[ ] Cross-field validation (e.g., end date after start date) is identical in both modes
[ ] Backend validation is identical — backend must not skip validation on updates
```

## Common Violations

| Violation | Impact |
|-----------|--------|
| Add requires name min 3 chars, Edit allows 1 | User can save invalid name via Edit |
| Add validates phone format, Edit does not | Invalid phones saved via Edit |
| Add requires category, Edit makes it optional | Data integrity broken |
| Add validates end date > start date, Edit does not | Invalid date ranges saved via Edit |

---

# 5. UI and Layout Consistency

```text
[ ] Same field labels (not "Project Name" in Add and "Name" in Edit)
[ ] Same placeholder text
[ ] Same helper text and tooltips
[ ] Same required field indicators (asterisk placement)
[ ] Same field order within groups
[ ] Same visual grouping (if Add groups fields into sections, Edit uses same sections)
[ ] Same button placement (primary action in same position)
[ ] Same modal size if displayed in a modal (not Add in small modal, Edit in large modal)
[ ] Same spacing and padding between fields
[ ] Mobile layout is identical (same stacking, same input sizes)
```

---

# 6. State Consistency

```text
[ ] Both Add and Edit show a loading spinner while submitting
[ ] Both Add and Edit disable the submit button during submission
[ ] Both Add and Edit show inline field errors in the same position
[ ] Both Add and Edit show API/server errors in the same way (banner or inline)
[ ] Both Add and Edit show a success state (toast, redirect, or confirmation)
[ ] Both Add and Edit handle network errors with the same message
[ ] Edit shows a loading state while pre-filling data (not an empty form flash)
```

---

# 7. Permission Consistency

```text
[ ] If a role can access Add, they can access Edit (unless there is a business rule)
[ ] If a field is read-only for a role in Edit, the field does not exist in Add for that role
[ ] If a role cannot edit a specific field, it is clearly displayed as disabled (not hidden and silently ignored)
[ ] Backend permission checks are identical for create and update operations
[ ] Frontend does not enforce different permissions in Add vs Edit — backend handles all permission enforcement
```

---

# 8. Pre-Fill Behavior (Edit Only)

## Loading Pre-Fill Data

```text
[ ] Show skeleton/spinner while loading existing data — do NOT render an empty form then fill it in
[ ] If data fails to load — show an error state, not an empty form ready for overwrite
[ ] Pre-fill ALL form fields with existing values, not just some
[ ] Preserve field types: dates as date objects, numbers as numbers (not strings)
[ ] Normalize data before pre-fill: trim strings, convert nulls to empty strings, etc.
```

## Controlled Fields in Edit

If some fields should not be editable after creation:

```text
[ ] Field is shown as read-only/disabled (not hidden)
[ ] Field has a tooltip or helper text explaining why it cannot be changed
[ ] Disabled field is NOT submitted in the form payload (backend ignores or rejects it anyway)
[ ] Example: "Email cannot be changed after account creation"
```

## Example: Pre-Fill Pattern (React)

```tsx
// In Edit mode — load first, then render
const { data: project, isLoading } = useProject(projectId);

if (isLoading) return <Skeleton />;
if (!project) return <ErrorState message="Project not found." />;

return (
  <ProjectForm
    mode="edit"
    initialValues={project}
    onSubmit={handleUpdate}
  />
);
```

---

# 9. Submit Behavior

| Behavior | Add | Edit |
|----------|-----|------|
| HTTP method | POST | PATCH or PUT |
| Success message | "Project created successfully." | "Project updated successfully." |
| Redirect after success | To list or new record | Stay on page or back to list |
| Submit button label | "Create Project" | "Save Changes" |
| Confirmation before submit | Rarely needed | Sometimes needed if destructive change |

## Rules

```text
[ ] Success messages use past tense and name the entity ("Project created", not "Done")
[ ] Error messages are identical in both modes
[ ] On success in Add: redirect or close modal — do not leave empty form open
[ ] On success in Edit: show confirmation — do not silently close without feedback
[ ] On failure: form stays open with values intact — user should not re-type everything
```

---

# 10. Common Violations & Anti-Patterns

| Violation | Why It's Wrong | Fix |
|-----------|---------------|-----|
| Two separate form components (AddForm, EditForm) | Validation diverges over time | Single shared form component |
| Edit form skips required field validation | Users bypass data quality rules | Same schema for both |
| Add shows 5 fields, Edit shows 8 | Users can't find fields they expect | Same field set |
| Add has real-time validation, Edit has submit-only | Inconsistent UX causes confusion | Same validation timing |
| Edit form shows blank while loading data | User thinks data was deleted | Skeleton/loading state |
| Success toast says "Saved" in Edit, "Created" in Add | Not wrong, but inconsistent | Use specific messages |
| Add form in modal, Edit form as full page | Different UX patterns for same action | Agree on one pattern, use it both |
| Edit allows empty name (overrides required) | Data integrity broken | Required = required in both |
| Disabled field in Edit silently submitted | Backend may reject or mishandle | Never submit disabled fields |
| Phone validated on Add, not on Edit | Invalid data saved via Edit | Identical validation schema |

---

# 11. Consistency Checklist — Before Marking Done

## Structural

```text
[ ] Add and Edit use the same form component (not two copies)
[ ] The same validation schema (Zod, Yup, etc.) is imported in both
[ ] Field list is identical (with documented exceptions)
[ ] Field order is identical
[ ] Labels, placeholders, and helper text are identical
```

## Validation

```text
[ ] Every required field in Add is required in Edit
[ ] Every optional field in Add is optional in Edit
[ ] Character limits are the same in both
[ ] Format rules (email, phone, URL, date) are the same in both
[ ] Cross-field rules (end > start) are enforced in both
[ ] Backend validates both create and update endpoints equally
```

## UI/UX

```text
[ ] Same loading state (spinner + disabled button) in both
[ ] Same inline error position in both
[ ] Same API error display in both
[ ] Same button placement in both
[ ] Mobile layout is identical in both
[ ] Edit shows skeleton while loading pre-fill data (not empty form flash)
```

## Pre-Fill (Edit)

```text
[ ] All fields pre-filled with existing values
[ ] Date/number types preserved correctly
[ ] Disabled fields shown but not submitted
[ ] Disabled fields have explanatory tooltip
[ ] Failed data load shows error state, not empty form
```

## Submit

```text
[ ] Add uses POST, Edit uses PATCH or PUT
[ ] Success messages are appropriate for each mode
[ ] Form stays intact on error (user does not re-type)
[ ] Success redirects/closes correctly for each mode
[ ] No double-submit possible in either mode
```

## Final

```text
[ ] Manually tested: Add form flow complete (submit, success, error)
[ ] Manually tested: Edit form flow complete (load, change, submit, success, error)
[ ] Manually tested: Same validation rules hit in both (try submitting invalid data in each)
[ ] Mobile tested for both Add and Edit
[ ] Ticket updated with evidence that both were tested
```

---

## Practice Task

Apply what you learned by building a single shared form component used in both Add and Edit modes.

**→ [Task 03: Build a Shared Add / Edit Project Form](../../tasks/frontend/03-add-edit-project-form.md)**

Covers: one `ProjectForm` component with a `mode` prop, one shared validation schema imported in both modes, skeleton loader for pre-fill, identical field order and error messages, correct HTTP method per mode.
