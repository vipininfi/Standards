# Frontend Standards — Forms & Validation

> Every form must handle every state, show every error, and behave identically whether adding or editing.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-form-structure) | Form Structure |
| [2](#2-field-standards) | Field Standards |
| [3](#3-required-fields) | Required Fields |
| [4](#4-validation-timing) | Validation Timing |
| [5](#5-error-display) | Error Display |
| [6](#6-addedit-consistency) | Add/Edit Consistency |
| [7](#7-password-fields) | Password Fields |
| [8](#8-phone-fields) | Phone Fields |
| [9](#9-file-upload-fields) | File Upload Fields |
| [10](#10-disabled-fields) | Disabled Fields |
| [11](#11-form-submission) | Form Submission |

---

# 1. Form Structure

```text
[ ] Form uses <form> element — not a <div> with a click handler
[ ] Submit triggered by button type="submit" — not an onClick handler alone
[ ] One primary action button per form
[ ] Cancel / secondary action on the left, primary action on the right
[ ] Form fields grouped logically — related fields together
[ ] Section headings used when form has more than 6 fields
[ ] Form width appropriate to content — not full-width on wide screens unless needed
```

---

# 2. Field Standards

```text
TEXT INPUTS
[ ] Label above the field — not placeholder-only (placeholder is not a label)
[ ] Placeholder text describes the expected format: "e.g., John Smith" — not repeating the label
[ ] Input autocomplete attributes set correctly:
      name="email" autocomplete="email"
      name="password" autocomplete="current-password" (login) / "new-password" (signup)
      name="name" autocomplete="name"

DATE FIELDS
[ ] Date picker used — not a plain text input
[ ] Format shown to user: "DD/MM/YYYY" or locale-appropriate
[ ] Min/max date constraints applied where relevant
[ ] Date stored as ISO 8601 (YYYY-MM-DD) — displayed in locale format

SELECT / DROPDOWN
[ ] Native <select> for simple cases; custom component for branded UI
[ ] Options loaded from backend — not hardcoded unless truly static
[ ] Placeholder option: "Select a status..."
[ ] Searchable when > 7 options

CHECKBOXES & RADIO
[ ] Label is clickable (htmlFor / label wrapping)
[ ] Radio groups: one option selected by default where appropriate
[ ] Checkbox groups: none selected by default is valid

TEXTAREA
[ ] Visible character count when there is a max length
[ ] Textarea resizes vertically — not horizontally

RICH TEXT EDITOR
[ ] Plain text fallback available if editor fails to load
[ ] Content sanitized before saving to prevent XSS
[ ] Max length enforced on the backend — editor max is UX only
```

---

# 3. Required Fields

```text
[ ] Required fields marked with a red asterisk (*)
[ ] Asterisk explained once: "* Required field" (near top of form or in form legend)
[ ] Optional fields are not marked — only required ones are marked
[ ] Backend validates required fields too — frontend required marker is UX only
[ ] aria-required="true" on required inputs
```

---

# 4. Validation Timing

```text
ON SUBMIT
[ ] All fields validated on submit — never skip validation because "it looks filled"
[ ] If errors exist: form does NOT submit, first error field receives focus
[ ] Error messages appear next to each field

ON BLUR (after user leaves a field)
[ ] Validate after user leaves a field — not while typing
[ ] Exception: real-time feedback for password strength (see Password Fields)
[ ] Do not validate on focus (on entering a field)

NEVER
× Validate an empty field on first focus
× Show errors before the user has touched the field (except after submit attempt)
× Clear errors while user is still typing (clear on blur or on submit only)
```

---

# 5. Error Display

```text
FIELD ERRORS
[ ] Error message directly below the field it belongs to
[ ] Red/danger color for text and border
[ ] Specific: "Email is required." not "Field required."
[ ] Specific: "Password must be at least 8 characters." not "Invalid password."
[ ] aria-describedby on the input pointing to the error element
[ ] role="alert" on the error element (or use aria-live="polite")

FORM-LEVEL ERRORS (API errors that don't map to a field)
[ ] Displayed in a banner above the submit button (or at top of form)
[ ] Red/danger styled banner — not a toast
[ ] Specific: "A project with this name already exists." not "Error."
[ ] Error does not disappear automatically — user must dismiss or correct

ERROR MESSAGE STANDARDS
| Situation | Example Message |
|-----------|----------------|
| Required field empty | "[Field] is required." |
| Email invalid | "Enter a valid email address." |
| Password too short | "Password must be at least 8 characters." |
| Passwords don't match | "Passwords do not match." |
| Name too long | "[Field] must be 100 characters or fewer." |
| Duplicate value | "A project with this name already exists." |
| API failure | "Something went wrong. Please try again." |
```

---

# 6. Add/Edit Consistency

This is a non-negotiable. Add and Edit for the same entity must be identical in every dimension:

```text
COMPONENT
[ ] Add and Edit use the exact same form component
[ ] Edit passes initial values as props — it does not re-implement the form
[ ] Validation schema is shared — not duplicated

FIELDS
[ ] Same fields in same order
[ ] Same labels, same placeholder text
[ ] Same required/optional markers
[ ] Same field types (both use a date picker — not one date picker, one text input)

VALIDATION
[ ] Same rules (same min/max, same regex)
[ ] Same error messages
[ ] No validation rules that apply only to Add or only to Edit

UI & LAYOUT
[ ] Same width, same spacing
[ ] Same button labels (except: Add = "Create Project", Edit = "Save Changes")
[ ] Same cancel behavior (both navigate back or close modal)

SUBMISSION
[ ] Both show loading state during submission
[ ] Both show success feedback after submission
[ ] Both show errors in the same location
[ ] Add redirects to list or new item; Edit returns to detail or list — both are intentional
```

---

# 7. Password Fields

```text
[ ] Password input type="password" — never type="text" for password
[ ] "Show/hide password" toggle (eye icon) — switches between type="password" and type="text"
[ ] New password: strength indicator shown while typing
    Weak    → red bar + "Weak — add uppercase, numbers, or symbols"
    Medium  → amber bar + "Medium — a bit stronger"
    Strong  → green bar + "Strong"
[ ] Minimum: 8 characters (enforce on both frontend and backend)
[ ] "Confirm password" field: validated against password field on blur
[ ] autocomplete="new-password" on new password fields
[ ] autocomplete="current-password" on current password / login fields
[ ] Password never logged, never included in error messages
```

---

# 8. Phone Fields

```text
[ ] Phone number stored and validated in E.164 format: +[country code][number]
    Example: +14155551234 (US), +447911123456 (UK)
[ ] Country code prefix selector shown to user (dropdown or dial code input)
[ ] Input masked/formatted for the user: (415) 555-1234 — stored as +14155551234
[ ] Validated for correct digit count for the selected country
[ ] Error: "Enter a valid phone number including country code."
[ ] Phone field uses type="tel" and autocomplete="tel"
```

---

# 9. File Upload Fields

```text
[ ] Accepted file types shown to user: "JPG, PNG, or PDF up to 5 MB"
[ ] File type validated by MIME type — not just by file extension
[ ] File size validated on frontend (before upload) and backend (after upload)
[ ] Progress bar shown for uploads > 1 second
[ ] Upload can be cancelled while in progress
[ ] Preview shown after selection (images: thumbnail; PDF: filename + icon)
[ ] Remove/change button available after selection
[ ] Error: "File must be JPG, PNG, or PDF."
[ ] Error: "File size must be under 5 MB."
[ ] Drag-and-drop zone with clear visual feedback on drag-over
[ ] Keyboard accessible: click to open file picker as fallback
```

---

# 10. Disabled Fields

```text
[ ] Disabled fields are visually distinct: reduced opacity, cursor not-allowed
[ ] Disabled fields are not editable — not just visually styled
[ ] If a field is disabled due to a permission: tooltip explains why
[ ] If a field is auto-populated (e.g., created by, date): shown as read-only, not disabled
[ ] Read-only vs disabled: disabled excluded from form submit; read-only is included
[ ] Never disable a field to hide a validation error — show the error instead
```

---

# 11. Form Submission

```text
[ ] Submit button disabled and shows loading spinner during submission
[ ] No double-submit possible — button disabled or debounced
[ ] Form does not clear while submitting (user can see what they entered)
[ ] On success:
    — Add form: redirect to new item or list, show success toast
    — Edit form: return to view, show "Saved." toast or inline indicator
    — Modal form: close modal, show success toast, list refreshes
[ ] On error:
    — Form stays open with data intact
    — Error shown in UI — not just in console
    — User can correct and resubmit
[ ] After success: scroll to top of page if there is a page transition
```

---

## Practice Task

**→ [Registration Form](../../../tasks/frontend/01-registration-form.md)**
Build a registration form with 7 fields, password strength, phone input with country code, and all loading/error/success states.
