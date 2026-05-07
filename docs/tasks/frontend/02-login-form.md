# Task: Build a Login Form with Forgot & Reset Password

**Platform:** React (or WeWeb)
**Covers:** [Login & Signup Standards](../../checklists/frontend/login-signup.md) · [Frontend & UI Standards](../../standards/frontend-ui-standards.md)

---

## Scenario

You are building the login and password recovery flow for **WorkFlow**. This includes the login page, the forgot password page, and the reset password page (accessed via email link).

---

## What to Build

Three separate pages/views that work together as a complete auth flow:

1. **Login Page** — email + password, with a "Forgot password?" link
2. **Forgot Password Page** — email input, returns a confirmation regardless of whether the account exists
3. **Reset Password Page** — new password + confirm, accessed via token in the URL

---

## Part 1 — Login Page

### Fields

| Field | Type | Required |
|-------|------|----------|
| Email | Email input | Yes |
| Password | Password input (masked) | Yes |
| Remember Me | Checkbox | No |

### Requirements

- Email trimmed and lowercased before submission
- Password field: do NOT trim (spaces in passwords are valid)
- No password complexity check on login — only check that it is not empty
- Password field has show/hide toggle

### Error Messages

- Empty email: "Email is required."
- Invalid email format: "Enter a valid email address."
- Empty password: "Password is required."
- Wrong email or password: "Incorrect email or password." — same message for wrong password AND unknown email (never reveal whether the email exists)
- Account disabled: "Your account has been suspended. Contact support."
- API error: "Something went wrong. Please try again."

### States

- **Loading** — submit spinner, button disabled, cannot double-submit
- **Error** — message shown as a banner above the form (not inline for this case — it applies to the whole form)
- **Success** — redirect to dashboard (or to the page the user originally tried to reach)

### UI

- "Forgot password?" link near the password field
- "Don't have an account? Sign up" link below the form
- Enter key submits the form
- `autocomplete="current-password"` on password field
- Form does not reset on error

---

## Part 2 — Forgot Password Page

### Fields

| Field | Type | Required |
|-------|------|----------|
| Email | Email input | Yes |

### Requirements

- Email trimmed and lowercased before submission
- Same success message whether or not the account exists: "If an account exists with that email, you'll receive a password reset link shortly."
- Do NOT say "email not found" — email enumeration is a security issue

### States

- **Loading** — spinner, button disabled
- **Success** — hide the form, show the confirmation message and a "Resend" link
- **Resend** — button disabled for 60 seconds after sending (show countdown)

### UI

- "Back to login" link visible at all times
- Success state clearly shows: "Check your inbox" + which email the link was sent to

---

## Part 3 — Reset Password Page

Reached via a link in the reset email (`/reset-password?token=xxx`).

### Fields

| Field | Type | Required |
|-------|------|----------|
| New Password | Password input | Yes |
| Confirm New Password | Password input | Yes |

### Requirements

- On page load: validate the token from the URL **before** rendering the form
  - If token is invalid or expired: show error message + "Request a new link" button — do NOT show the form
- Password: same complexity rules as registration (8+ chars, upper, lower, number, special)
- Confirm password must match
- Same password as previous: backend blocks this — show "Your new password must be different from your previous password."
- On success: redirect to login page — do NOT auto-login the user

### States

- **Validating token** — spinner shown while token is being checked
- **Invalid token** — error shown, form never rendered
- **Form visible** — token valid, user can enter new password
- **Loading** — spinner on submit, button disabled
- **Success** — show confirmation + redirect to login after 3 seconds (or show "Go to login" button)

### UI

- Both password fields have show/hide toggles
- Show password strength indicator (same as registration)
- "Back to login" link visible

---

## What You Should NOT Do

- Do not reveal whether an email exists in the forgot password flow
- Do not auto-login the user after password reset
- Do not render the reset form before the token is validated
- Do not allow the form to be submitted twice (disable on submit)
- Do not store anything sensitive in localStorage

---

## Checklist to Run When Done

Use the [Login & Signup Checklist](../../checklists/frontend/login-signup.md#10-auth-checklist--before-marking-done) — all sections: Login Form, Forgot Password, Reset Password, and Session & Security.

Then run the [Frontend Checklist](../../checklists/frontend-checklist.md).

---

## Done When

```text
LOGIN
[ ] Email and password validated (not empty, valid email format)
[ ] Same error for wrong password and unknown email
[ ] Loading state on submit, no double submit
[ ] Password show/hide toggle works
[ ] "Forgot password?" link visible and goes to forgot password page
[ ] Enter key submits
[ ] Redirect to intended page after login
[ ] Mobile tested

FORGOT PASSWORD
[ ] Email validated
[ ] Same response regardless of whether account exists
[ ] Success state shows confirmation message with email address
[ ] Resend button disabled for 60 seconds
[ ] "Back to login" link works

RESET PASSWORD
[ ] Token validated on page load before form renders
[ ] Invalid/expired token shows error, not a broken form
[ ] Password complexity enforced
[ ] Confirm password match checked
[ ] Loading state on submit
[ ] On success: redirect to login (not auto-login)
[ ] Both show/hide toggles work
[ ] Mobile tested
```
