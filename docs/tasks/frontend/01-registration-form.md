# Task: Build a User Registration Form

**Platform:** React (or WeWeb)
**Covers:** [Form Field Validation](../../checklists/frontend/form-field-validation.md) · [Frontend & UI Standards](../../standards/frontend-ui-standards.md)

---

## Scenario

You are building the signup page for a project management app called **WorkFlow**. New users register with their name, email, password, and an optional phone number. They must accept the Terms & Conditions before submitting.

---

## What to Build

A registration form component with the following fields:

| Field | Type | Required |
|-------|------|----------|
| First Name | Text input | Yes |
| Last Name | Text input | Yes |
| Email | Email input | Yes |
| Password | Password input | Yes |
| Confirm Password | Password input | Yes |
| Phone Number | Tel input with country code selector | No |
| Terms & Conditions checkbox | Checkbox | Yes |

---

## Requirements

### Fields
- First name and last name: minimum 1 character, maximum 100 characters
- Email: valid email format, trimmed and lowercased before submission
- Password: minimum 8 characters, at least 1 uppercase letter, 1 lowercase letter, 1 number, 1 special character
- Confirm password: must exactly match the password field — validated client-side in real time once both fields have values
- Phone (optional): if filled, must be a valid international number in E.164 format — include a country code dropdown
- Terms checkbox: must be checked before the form can submit

### Validation Behavior
- Show a password strength indicator (Weak / Fair / Strong) while the user types in the password field
- Validate fields on blur (when the user leaves the field) — not while typing
- Confirm password validates in real time once both fields have values
- Required field asterisks shown next to labels
- Show character count on first name and last name fields (max 100)

### Error Messages
Use exact wording from the [Login & Signup Standards](../../checklists/frontend/login-signup.md#error-messages-1):
- Empty required field: "Field name is required."
- Invalid email: "Enter a valid email address."
- Weak password: "Password must be at least 8 characters with uppercase, lowercase, number, and special character."
- Passwords don't match: "Passwords do not match."
- Terms not checked: "You must accept the Terms & Conditions to continue."

### States
- **Default** — form fields empty, submit button disabled until all required fields are valid
- **Loading** — submit button shows spinner and is disabled, no double submit possible
- **Error** — inline field errors shown below each field, form stays populated
- **Duplicate email** — show "An account with this email already exists. Log in instead?" with a link to login
- **Success** — show "Check your inbox to verify your email." message

### UI Details
- Password and Confirm Password fields each have a show/hide toggle (eye icon)
- Terms & Conditions text links to the terms page — opens in a new tab
- "Already have an account? Log in" link visible below the form
- `autocomplete="new-password"` on both password fields
- `autocomplete="email"` on the email field
- Form does not reset when a submission error occurs

---

## What You Should NOT Do

- Do not put validation logic directly in the submit handler — use a schema (Zod, Yup, or similar)
- Do not call the API without running full validation first
- Do not show the success message while still on the same form view without clearing it
- Do not accept the role or user ID from the form payload — backend assigns them
- Do not skip the loading state

---

## Checklist to Run When Done

Use the [Form Field Validation Checklist](../../checklists/frontend/form-field-validation.md) and the [Login & Signup Checklist](../../checklists/frontend/login-signup.md#10-auth-checklist--before-marking-done) — Signup Form section.

Then run the [Frontend Checklist](../../checklists/frontend-checklist.md).

---

## Done When

```text
[ ] All fields render correctly with correct input types
[ ] Validation fires on blur for each field
[ ] Password strength indicator updates while typing
[ ] Confirm password validates in real time once both fields have values
[ ] All error messages match the standard wording
[ ] Submit button disabled until form is valid
[ ] Loading state shows during submission (no double submit)
[ ] Duplicate email shows "Log in instead?" link
[ ] Success state shown after submission
[ ] Phone field accepts country code + number and validates E.164 format
[ ] Terms link opens in new tab
[ ] "Already have an account?" link is visible
[ ] Form stays populated when submission fails
[ ] Tested on mobile (no layout breakage, keyboard does not obscure submit button)
```
