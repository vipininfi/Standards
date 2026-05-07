# Frontend & UI Standards

Core rule:

> Build consistent, accessible, and user-friendly interfaces. Every page, form, and component must follow the same standards — the user should never feel like different parts of the product were built by different teams.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-ui-consistency-standard) | UI Consistency — Add vs Edit |
| [2](#2-reusability-standard) | Reusability Standard |
| [3](#3-form-standards) | Form Standards |
| [4](#4-required-star-standard) | Required Star Standard |
| [5](#5-password--security-validation-standard) | Password & Security Validation |
| [6](#6-phone-field-standard) | Phone Field Standard |
| [7](#7-disabled--enabled-field-standard) | Disabled / Enabled Field |
| [8](#8-tooltip-standard) | Tooltip Standard |
| [9](#9-error-handling-standard) | Error Handling |
| [10](#10-notification-standard) | Notification Standard |
| [11](#11-draft-save--session-recovery-standard) | Draft Save & Session Recovery |
| [12](#12-loading-state-standard) | Loading State Standard |
| [13](#13-delete-confirmation-standard) | Delete Confirmation |
| [14](#14-testing-your-own-work) | Testing Your Own Work |
| [15](#15-design-system-standard) | Design System Standard |
| [16](#16-backend--frontend-error-contract) | Backend + Frontend Error Contract |

---

# 1. UI Consistency Standard

Similar pages must follow the same UI and behavior.

## Add/Edit Form Rule

If Add and Edit pages are for the same entity, they must use the same:

| Area | Standard |
|------|----------|
| Field order | Same |
| Labels | Same |
| Validation | Same |
| Required stars | Same |
| Tooltips | Same |
| Button placement | Same |
| Error messages | Same |
| Loading state | Same |
| Section spacing | Same |
| Mobile layout | Same |

## Example

Bad:

```text
Add Project has phone validation.
Edit Project allows any text.
```

Good:

```text
Both Add and Edit Project use the same ProjectForm component and validation schema.
```

---

# 2. Reusability Standard

Avoid duplicate code and duplicate UI.

## Reuse When

| Scenario | Standard |
|----------|----------|
| Same form used in Add/Edit | Shared form component. |
| Same validation rules | Shared validation schema. |
| Same table style | Shared table component. |
| Same button styles | Shared button component. |
| Same API error handling | Shared error handler. |
| Same notification logic | Shared toast/notification utility. |
| Same modal behavior | Shared modal component. |

## Good Reusable Structure

```text
components/
  forms/
    ProjectForm.tsx
  modals/
    DeleteConfirmationModal.tsx
  ui/
    Button.tsx
    Input.tsx
    Tooltip.tsx

validations/
  project.schema.ts

services/
  project.service.ts

utils/
  api-error-handler.ts
```

## Anti-Pattern

Copy-pasting the same form into Add and Edit pages. This causes bugs because one page gets updated and the other does not.

---

# 3. Form Standards

Every form must have proper validation, loading, error, and success behavior.

## Mandatory Form Standards

```text
[ ] Every field has a label.
[ ] Required fields have a star (*).
[ ] Required fields have validation.
[ ] Optional fields are clearly optional where needed.
[ ] Field-level errors are shown near the field.
[ ] Submit button has loading state.
[ ] Submit button disables during request.
[ ] Form values are not cleared on error.
[ ] Success message is shown after successful submit.
[ ] API error is shown properly.
[ ] Mobile layout is checked.
```

## Required Field Standard

Good:

```text
Project Name *
[ Enter project name ]
```

Bad — using only placeholder text with no label:

```text
[ Enter name ]
```

---

# 4. Required Star Standard

Use star only for required fields.

```text
Name *
Email *
Phone Number *
```

Optional fields can use:

```text
Description (optional)
```

Do not put stars randomly. Do not forget stars on required fields.

---

# 5. Password & Security Validation Standard

For signup, reset password, change password, and admin user creation.

## Password Rules

| Rule | Standard |
|------|----------|
| Minimum length | 8 minimum, 12 preferred |
| Uppercase | Required for secure apps |
| Lowercase | Required |
| Number | Required |
| Symbol | Recommended |
| Common password block | Recommended |
| Confirm password | Required for reset/signup |
| Show/hide password | Required |
| Strength indicator | Recommended |

## Password Error Example

```text
Password must contain at least 8 characters, one uppercase letter, one lowercase letter, and one number.
```

## Security Rules

```text
[ ] Do not expose whether email exists during forgot password.
[ ] Reset password token must expire.
[ ] Login must have rate limiting.
[ ] Password must never be stored in plain text.
[ ] Sensitive forms must not silently fail.
```

---

# 6. Phone Field Standard

Phone fields must not be simple text fields.

## Required Standard

| Item | Standard |
|------|----------|
| Country code | Required |
| Country selector | Required |
| Number input | Digits only |
| Length validation | Based on country |
| Storage format | E.164 format |
| Example | `+919876543210` |

## UI Standard

```text
Phone Number *
[ 🇮🇳 +91 ] [ 9876543210 ]
```

## Error

```text
Enter a valid phone number.
```

---

# 7. Disabled / Enabled Field Standard

Disabled fields must be visually clear.

## Disabled Field UI

| Area | Standard |
|------|----------|
| Background | Light grey / muted |
| Text | Muted but readable |
| Cursor | Not allowed |
| Tooltip | Explain why disabled if not obvious |
| Label | Still visible |
| Validation | Do not show active validation on disabled field |

## Example Tooltip

```text
This field cannot be edited after project creation.
```

## Bad

A disabled field that looks like a normal editable field.

## Good

A disabled field that visually communicates it is locked, and why.

---

# 8. Tooltip Standard

Tooltips should be used only where they add clarity.

## Use Tooltip For

| Scenario | Example |
|----------|---------|
| Complex field | "Workspace ID is automatically assigned." |
| Disabled action | "Only admins can delete this project." |
| Security action | "Changing this will affect user permissions." |
| Technical setting | "Webhook URL receives project update events." |
| Billing/plan restriction | "Upgrade required to enable this feature." |

## Do Not Use Tooltip For

- Obvious fields like Name or Email.
- Replacing proper labels.
- Long documentation.
- Critical errors.

## Tooltip Text Standard

Good:

```text
Only workspace owners can change this setting.
```

Bad:

```text
Info.
```

---

# 9. Error Handling Standard

Frontend must show errors to the user properly. The user will not open the browser console.

## Mandatory Error Handling

```text
[ ] Show API error in UI.
[ ] Show field-level validation errors.
[ ] Show permission errors clearly.
[ ] Show network/server error clearly.
[ ] Do not silently fail.
[ ] Do not only console.log(error).
[ ] Preserve form data on error.
[ ] Allow retry where possible.
```

## Error Color Standard

| Type | Color |
|------|-------|
| Error | Red |
| Success | Green |
| Warning | Yellow/Orange |
| Info | Blue/Grey |
| Disabled | Grey |

## Error Message Examples

Bad:

```text
Something went wrong.
```

Better:

```text
We could not save the project. Please check the required fields and try again.
```

Bad:

```text
Error 500.
```

Better:

```text
Server error occurred while saving. Please try again in a moment.
```

---

# 10. Notification Standard

Use consistent notifications.

## Notification Types

| Type | Use For | Color |
|------|---------|-------|
| Success | Created, updated, deleted, saved | Green |
| Error | Failed action, validation, API error | Red |
| Warning | Risk, incomplete action, attention needed | Yellow/Orange |
| Info | Neutral update | Blue/Grey |

## Examples

Success:

```text
Project created successfully.
```

Error:

```text
Project could not be created. Please try again.
```

Warning:

```text
You have unsaved changes.
```

Info:

```text
Changes are being processed.
```

---

# 11. Draft Save & Session Recovery Standard

For long forms, important forms, or multi-step forms, draft recovery should be considered.

## Use Draft Saving For

| Form Type | Draft Needed? |
|-----------|---------------|
| Simple contact form | Usually no |
| Signup form | Usually no |
| Long project setup form | Yes |
| Multi-step onboarding | Yes |
| Quote/invoice builder | Yes |
| Application form | Yes |
| AI prompt/config form | Yes |
| Blog/editor/content form | Yes |

## Draft Standards

```text
[ ] Auto-save draft after changes.
[ ] Show "Draft saved" status.
[ ] Recover fields after refresh/session loss.
[ ] Warn before leaving unsaved changes.
[ ] Clear draft after successful submission.
```

## Example UI

```text
Draft saved 2 minutes ago.
```

```text
You have unsaved changes. Are you sure you want to leave?
```

---

# 12. Loading State Standard

Every async action must show a loading state.

## Required Loading States

| Action | Standard |
|--------|----------|
| Form submit | Button shows loading and disables |
| Table fetch | Skeleton rows or loader |
| Delete action | Delete button shows deleting |
| File upload | Progress indicator |
| API sync | Syncing status |
| Page load | Skeleton/loader |

## Button Example

Normal:

```text
Save Project
```

Loading:

```text
Saving...
```

Success:

```text
Saved
```

Delete loading:

```text
Deleting...
```

---

# 13. Delete Confirmation Standard

Any destructive action must have confirmation.

## Low-Risk Delete

```text
Are you sure you want to delete this item?
This action cannot be undone.

[Cancel] [Delete]
```

## High-Risk Delete

```text
Type DELETE to confirm.
```

## Very High-Risk Delete (workspace/account/data)

```text
Type the workspace name to confirm deletion.
```

Also consider password or 2FA confirmation.

---

# 14. Testing Your Own Work

Every member must test their own work before marking complete. QA should not find basic issues that the developer could easily check.

## Self-Testing Checklist

```text
[ ] Main success flow works.
[ ] Required field validation works.
[ ] Invalid input validation works.
[ ] API failure is shown in UI.
[ ] Loading state works.
[ ] Empty state works.
[ ] Mobile view checked.
[ ] Similar Add/Edit pages checked.
[ ] Permission-based behavior checked.
[ ] Console checked for obvious errors.
[ ] No broken layout.
[ ] No silent failure.
```

## Completion Comment Must Mention Testing

Example:

```text
Testing Done:
- Created project successfully.
- Checked required field validation.
- Checked API failure handling.
- Verified Add/Edit UI consistency.
- Checked mobile layout.
```

---

# 15. Design System Standard

Every project should follow a consistent design system.

## Required Design System Areas

| Area | Standard |
|------|----------|
| Colors | Primary, secondary, success, error, warning, info |
| Typography | Heading, body, small text |
| Spacing | Consistent section and field spacing |
| Buttons | Primary, secondary, danger, disabled |
| Inputs | Normal, focused, error, disabled |
| Cards | Border, radius, shadow |
| Modals | Header, body, footer actions |
| Tables | Header, row, empty, loading |
| Forms | Label, required star, help text, error |
| Notifications | Success/error/warning/info |

## Rule

Do not create a new design style for every page. Similar pages must feel like they belong to the same product.

---

# 16. Backend + Frontend Error Contract

Frontend and backend must agree on error format before building.

## Backend Should Return

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please fix the highlighted fields.",
    "fields": {
      "email": "Enter a valid email address."
    }
  }
}
```

## Frontend Should Handle

| Backend Code | Frontend Behavior |
|--------------|-------------------|
| `VALIDATION_ERROR` | Show field-level errors |
| `UNAUTHENTICATED` | Redirect to login |
| `FORBIDDEN` | Show permission message |
| `NOT_FOUND` | Show not found state |
| `CONFLICT` | Show duplicate/conflict message |
| `INTERNAL_ERROR` | Show retry message |

## Bad Frontend Behavior

```ts
console.log(error);
```

without showing anything to the user.
