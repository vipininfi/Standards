# Login & Signup Standards Checklist

> **Core Rule:** Auth flows are the first thing users see. Every error must be shown. Every field must be validated. Every state must be handled. No silent failures.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-login-form) | Login Form |
| [2](#2-signup--registration-form) | Signup / Registration Form |
| [3](#3-forgot-password-flow) | Forgot Password Flow |
| [4](#4-reset-password-form) | Reset Password Form |
| [5](#5-email-verification) | Email Verification |
| [6](#6-social--oauth-login) | Social / OAuth Login |
| [7](#7-session--token-handling) | Session & Token Handling |
| [8](#8-security-rules) | Security Rules |
| [9](#9-ux-standards) | UX Standards |
| [10](#10-auth-checklist--before-marking-done) | Auth Checklist — Before Marking Done |

---

# 1. Login Form

## Fields

| Field | Type | Required |
|-------|------|----------|
| Email | Email input | Yes |
| Password | Password input (masked) | Yes |
| Remember Me | Checkbox | No |

## Validation Rules

```text
[ ] Email — required, valid email format, trimmed, lowercase before submission
[ ] Password — required, minimum 1 character (backend validates length, not frontend)
[ ] No client-side password complexity check on login (only on signup)
[ ] Trim whitespace from email before submitting
[ ] Do NOT trim password (spaces in passwords are valid)
```

## Error Messages

| Scenario | Message Shown to User |
|----------|-----------------------|
| Empty email | "Email is required." |
| Invalid email format | "Enter a valid email address." |
| Empty password | "Password is required." |
| Wrong email or password | "Incorrect email or password." |
| Account not found | "Incorrect email or password." (same message — do not reveal whether the email exists) |
| Account disabled/locked | "Your account has been suspended. Contact support." |
| Too many failed attempts | "Too many failed attempts. Try again in X minutes." |
| Network/API error | "Something went wrong. Please try again." |

> **Security rule:** Never reveal whether the email exists. Use the same message for "wrong password" and "account not found."

## States

```text
[ ] Default state — form visible, submit enabled
[ ] Loading state — submit button shows spinner, button disabled, no double submit
[ ] Error state — inline field errors or banner error shown
[ ] Success state — redirect to dashboard or intended page
[ ] Rate-limited state — submit disabled with countdown or message
```

## UI Requirements

```text
[ ] Password field has show/hide toggle (eye icon)
[ ] "Forgot password?" link is visible near the password field
[ ] Submit button is full-width or prominently placed
[ ] Enter key submits the form
[ ] No autocomplete="off" on email (allow browser autofill)
[ ] Password field has autocomplete="current-password"
[ ] Form does not reset on error (user should not re-type everything)
```

---

# 2. Signup / Registration Form

## Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| First Name | Text | Yes | Min 1 char |
| Last Name | Text | Yes | Min 1 char |
| Email | Email | Yes | Unique check on backend |
| Password | Password | Yes | Complexity enforced |
| Confirm Password | Password | Yes | Must match |
| Phone Number | Tel | Depends on app | E.164 format |
| Terms & Conditions | Checkbox | Yes | Must be checked |
| Marketing Emails | Checkbox | No | Optional opt-in |

## Validation Rules

```text
[ ] First name — required, min 1 char, max 100 chars, no special chars except hyphens and apostrophes
[ ] Last name — required, min 1 char, max 100 chars
[ ] Email — required, valid format, trimmed, lowercased before submission
[ ] Password — required, minimum 8 chars, at least 1 uppercase, 1 lowercase, 1 number, 1 special char
[ ] Confirm Password — required, must exactly match password field (client-side check)
[ ] Phone — if required: E.164 format, country code included, validated on backend
[ ] Terms checkbox — must be checked before submission allowed
[ ] Show password strength indicator as user types (weak / fair / strong)
```

## Password Strength Indicator

```text
Weak:   Less than 8 chars OR only one character type
Fair:   8+ chars, 2 character types
Strong: 8+ chars, 3+ character types (upper, lower, number, special)
```

Show strength as a progress bar or label — do NOT block submission with a custom strength rating. Only enforce the rules in the validation list above.

## Error Messages

| Scenario | Message Shown to User |
|----------|-----------------------|
| Empty first name | "First name is required." |
| Empty last name | "Last name is required." |
| Empty email | "Email is required." |
| Invalid email | "Enter a valid email address." |
| Email already registered | "An account with this email already exists. Log in instead?" |
| Weak password | "Password must be at least 8 characters with uppercase, lowercase, number, and special character." |
| Passwords don't match | "Passwords do not match." |
| Terms not checked | "You must accept the Terms & Conditions to continue." |
| API error | "Something went wrong. Please try again." |

## States

```text
[ ] Default — form empty, submit disabled until required fields valid
[ ] Typing — inline validation shows after field is blurred (not while typing)
[ ] Loading — submit spinner, button disabled
[ ] Success — redirect to email verification page or dashboard
[ ] Duplicate email — show error with "Log in instead?" link
```

## UI Requirements

```text
[ ] Password field has show/hide toggle
[ ] Confirm Password field has show/hide toggle
[ ] Password strength indicator visible while typing
[ ] Terms & Conditions link opens in new tab (not navigates away from form)
[ ] Clear visual indicator for required fields (asterisk)
[ ] "Already have an account? Log in" link visible
[ ] Enter key does NOT submit form if confirm password is not filled (tab order matters)
[ ] autocomplete="new-password" on both password fields
[ ] autocomplete="email" on email field
```

---

# 3. Forgot Password Flow

## Steps

```text
1. User clicks "Forgot password?" on login page
2. User is shown a single email input field
3. User submits email
4. Backend sends reset email if account exists (same response whether email exists or not)
5. User sees confirmation message
6. User receives email → clicks link → lands on Reset Password form
```

## Fields

| Field | Type | Required |
|-------|------|----------|
| Email | Email | Yes |

## Validation Rules

```text
[ ] Email — required, valid email format, trimmed, lowercased
[ ] No indication of whether account exists with that email (security rule)
```

## Messages

| Scenario | Message Shown to User |
|----------|-----------------------|
| Valid submission (email exists or not) | "If an account exists with that email, you'll receive a password reset link shortly." |
| Empty email | "Email is required." |
| Invalid email format | "Enter a valid email address." |
| API error | "Something went wrong. Please try again." |
| Already sent recently | "A reset link was already sent. Check your inbox or try again in a few minutes." |

## States

```text
[ ] Default — email input, submit button
[ ] Loading — spinner, button disabled
[ ] Success — confirmation message shown (NOT same page as form, or form hidden)
[ ] Error — appropriate error message
```

## UI Requirements

```text
[ ] "Back to login" link visible
[ ] Success state clearly shows next step ("Check your inbox")
[ ] Resend link available on success state (with rate limit)
[ ] Email link expires (backend: typically 15–60 minutes)
```

---

# 4. Reset Password Form

## Accessed via: Link in email (contains token)

## Fields

| Field | Type | Required |
|-------|------|----------|
| New Password | Password | Yes |
| Confirm New Password | Password | Yes |

## Validation Rules

```text
[ ] New password — same complexity as signup (8+ chars, upper, lower, number, special)
[ ] Confirm password — must exactly match new password
[ ] Token from URL — validated on backend before allowing form submission
[ ] If token is invalid or expired — show error before displaying form
[ ] Password cannot be the same as the previous password (enforced on backend, shown to user)
```

## Error Messages

| Scenario | Message Shown to User |
|----------|-----------------------|
| Token invalid or expired | "This reset link is invalid or has expired. Request a new one." |
| Token already used | "This reset link has already been used. Request a new one." |
| Empty new password | "New password is required." |
| Weak password | "Password must be at least 8 characters with uppercase, lowercase, number, and special character." |
| Passwords don't match | "Passwords do not match." |
| Same as old password | "Your new password must be different from your previous password." |
| API error | "Something went wrong. Please try again." |

## States

```text
[ ] Loading (validating token) — spinner while token is checked before rendering form
[ ] Invalid token — error message + link to forgot password page (form NOT shown)
[ ] Form visible — ready to enter new password
[ ] Submitting — spinner, button disabled
[ ] Success — password updated, redirect to login with success message
```

## UI Requirements

```text
[ ] Token validated immediately on page load before rendering form
[ ] Both password fields have show/hide toggles
[ ] Show same password strength indicator as signup
[ ] "Back to login" link
[ ] On success: auto-redirect to login after 3 seconds OR show "Go to login" button
[ ] Do NOT auto-login after password reset — require explicit login
```

---

# 5. Email Verification

## When Required

- After signup (if email verification is required before dashboard access)
- After changing email address

## Flow

```text
1. After signup → show "Check your email" page
2. User clicks link in email
3. Backend validates token
4. User redirected to dashboard (or login with success message)
```

## States

```text
[ ] "Check your email" page — clearly tells user to check email, shows which email was used
[ ] Verifying — token validated on page load (spinner)
[ ] Success — "Email verified. Welcome!" or redirect to dashboard
[ ] Invalid/expired token — show error + resend button
[ ] Resend triggered — success confirmation + rate limit enforced
```

## Messages

| Scenario | Message |
|----------|---------|
| Initial page | "We sent a verification link to [email]. Check your inbox." |
| Token valid | "Email verified successfully." |
| Token expired | "This verification link has expired. Click below to resend." |
| Token invalid | "This verification link is invalid. Try requesting a new one." |
| Resend success | "Verification email resent. Check your inbox." |
| Resend rate limited | "Please wait before requesting another verification email." |

## UI Requirements

```text
[ ] Show the email address the link was sent to
[ ] "Resend email" button visible
[ ] "Wrong email? Sign up again" link visible
[ ] Resend button disabled for 60 seconds after sending (cooldown visible)
[ ] Backend: tokens expire after configurable time (e.g., 24 hours)
[ ] Backend: tokens are single-use
```

---

# 6. Social / OAuth Login

## Common Providers

- Google
- Apple (required for iOS apps that offer social login)
- Microsoft
- GitHub (developer apps)

## Standards

```text
[ ] OAuth handled entirely on backend — frontend only initiates the redirect
[ ] Backend creates or updates user record on first login
[ ] Role assigned on backend from database — never from OAuth payload
[ ] If account already exists with email → link accounts or show "log in with [provider]" message
[ ] If account disabled → block login even via OAuth
[ ] Always store OAuth provider + user ID on backend for future reference
[ ] Never store OAuth access token in localStorage (use secure HTTP-only cookie or session)
```

## Error Handling

| Scenario | Message |
|----------|---------|
| OAuth cancelled by user | Return to login page silently (or "Login cancelled") |
| OAuth provider error | "Could not connect to [Provider]. Try again or use email." |
| Email already registered (different method) | "An account with this email exists. Log in with [original method] instead." |
| Account disabled | "Your account has been suspended. Contact support." |

## UI Requirements

```text
[ ] Social login buttons show provider logo + name ("Continue with Google")
[ ] Social buttons styled per provider brand guidelines (Google has strict guidelines)
[ ] Clear visual divider between social login and email/password ("or")
[ ] Loading spinner shown while OAuth redirect is processing
```

---

# 7. Session & Token Handling

```text
[ ] Auth tokens stored in HTTP-only cookies OR memory (NOT localStorage)
[ ] Refresh token rotation: issue new refresh token each time it is used
[ ] Access token short expiry (15 minutes recommended)
[ ] Refresh token longer expiry (7–30 days)
[ ] On logout: invalidate token on backend (do not just delete client-side)
[ ] On session expiry: redirect to login page with "Session expired. Please log in again."
[ ] On 401 from API: trigger refresh token flow before showing error
[ ] On refresh token failure: log out user, redirect to login
[ ] Remember Me: extends refresh token expiry (e.g., 30 days vs 1 day)
[ ] Concurrent session handling defined (allow or revoke others on new login)
```

---

# 8. Security Rules

```text
[ ] Passwords are NEVER logged — not in console, not in API logs
[ ] Passwords are NEVER sent in query string (always in POST body)
[ ] Reset/verification tokens are NEVER exposed in URLs beyond their intended use
[ ] Auth endpoint has rate limiting (block after N failed attempts)
[ ] Lockout after too many failures with clear user feedback
[ ] HTTPS required — no auth flow over HTTP
[ ] CSRF protection enabled on all auth endpoints
[ ] Service role key NEVER in frontend code
[ ] JWT secret NEVER in frontend code
[ ] Email enumeration prevented: same response for "user not found" and "wrong password"
[ ] Frontend validation is UX only — backend validates and enforces everything
```

---

# 9. UX Standards

```text
[ ] Loading state on every auth button (spinner, no double-click)
[ ] Enter key submits the focused form
[ ] Tab order is logical (first name → last name → email → password → submit)
[ ] Error messages shown inline next to the relevant field
[ ] General/API errors shown as a banner above or below the form
[ ] Form does NOT reset on submission error
[ ] Redirect after login goes to the intended page (not always dashboard)
  — Store intended URL before redirecting to login, restore after
[ ] "Back" link always available
[ ] "Already have an account? Log in" and "Don't have an account? Sign up" links are mutually visible
[ ] Mobile: keyboard does not cover submit button
[ ] Mobile: input type="email" shows email keyboard, type="password" shows secure keyboard
```

---

# 10. Auth Checklist — Before Marking Done

## Login Form

```text
[ ] Email and password validated
[ ] Loading state on submit
[ ] Error shown in UI (not just console)
[ ] Incorrect credentials: same message for wrong password and unknown email
[ ] Password show/hide toggle works
[ ] Forgot password link visible and functional
[ ] Redirect to intended page after login
[ ] Mobile: tested on phone
```

## Signup Form

```text
[ ] All required fields validated
[ ] Password complexity rules enforced and communicated clearly
[ ] Confirm password match checked on client
[ ] Terms & Conditions checkbox required
[ ] Duplicate email error shown with "Log in instead?" link
[ ] Loading state on submit
[ ] Success: redirect or email verification flow initiated
[ ] Mobile: tested on phone
```

## Forgot Password

```text
[ ] Email validated
[ ] Same message regardless of whether email exists
[ ] Success state shown clearly
[ ] Backend: email sent if account exists
[ ] Backend: reset link expires
[ ] Backend: rate limit on requests
```

## Reset Password

```text
[ ] Token validated on page load before rendering form
[ ] Invalid/expired token shows error (not broken form)
[ ] New password complexity enforced
[ ] Confirm password match checked
[ ] Same password as old: blocked on backend, shown to user
[ ] Success: redirect to login (not auto-login)
```

## Session & Security

```text
[ ] Tokens not in localStorage
[ ] Logout invalidates token on backend
[ ] Session expiry handled gracefully (redirect to login)
[ ] Rate limiting on login and forgot password
[ ] No password or token in logs
[ ] Backend enforces all auth rules — frontend only displays
```

---

## Practice Task

Apply what you learned by building the full auth flow: login, forgot password, and reset password.

**→ [Task 02: Build a Login Form with Forgot & Reset Password](../../tasks/frontend/02-login-form.md)**

Covers: login error handling, email enumeration prevention, forgot password confirmation state, reset password with token validation on page load before rendering the form.
