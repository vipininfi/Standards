# Error Handling Checklist

> **Core Rule:** Every error must be shown to the user. No error stays only in the console. No form resets after a failed submission. Every error has a human-readable message and a clear next step.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-error-types--where-they-appear) | Error Types & Where They Appear |
| [2](#2-field-level-validation-errors) | Field-Level Validation Errors |
| [3](#3-form-level-api-errors) | Form-Level API Errors |
| [4](#4-page-level-errors) | Page-Level Errors |
| [5](#5-network--connectivity-errors) | Network & Connectivity Errors |
| [6](#6-permission-errors) | Permission Errors |
| [7](#7-session--auth-errors) | Session & Auth Errors |
| [8](#8-error-message-standards) | Error Message Standards |
| [9](#9-what-never-to-do-with-errors) | What Never to Do with Errors |
| [10](#10-error-handling-checklist--before-marking-done) | Error Handling Checklist — Before Marking Done |

---

# 1. Error Types & Where They Appear

| Error Type | Source | Where to Show |
|------------|--------|--------------|
| Required field empty | Client validation | Inline below the field |
| Invalid format (email, phone) | Client validation | Inline below the field |
| Business rule failure (duplicate, date logic) | API response | Inline on the field or form banner |
| API server error (500) | API response | Form banner or error toast |
| Not found (404) | API response | Page-level error state or toast |
| Permission denied (403) | API response | Inline banner or redirect |
| Not authenticated (401) | API response | Redirect to login |
| Network failure | Browser/fetch | Toast or inline banner |
| Session expired | API response | Redirect to login with message |

---

# 2. Field-Level Validation Errors

```text
[ ] Shown directly below the field they apply to — not in a summary at the top
[ ] Shown after the user has left (blurred) the field — not while they are typing
[ ] Exception: confirm password field — validate in real time once both fields have values
[ ] Field border/outline changes to red when it has an error
[ ] Field label changes to red (or stays neutral — be consistent throughout the app)
[ ] Error message is specific: "Enter a valid email address." not just "Invalid."
[ ] Error message uses the field name: "Phone number is required." not "Required."
[ ] Error clears when the user starts correcting the field
[ ] Error re-checks on blur (not on every keystroke)
[ ] Multiple errors on one field: show the most important one (not a list)
[ ] Scroll to the first error field automatically on form submit if any field has an error
[ ] Required field asterisk visible on label (so user understands which fields are mandatory)
```

---

# 3. Form-Level API Errors

Errors returned from the backend after the form is submitted.

```text
[ ] Form does NOT reset after a failed API call — user input is preserved
[ ] Specific field errors from API: mapped to the correct field (shown inline below the field)
[ ] General errors from API (not field-specific): shown as a banner above the submit button
[ ] API error banner has a visible X to dismiss
[ ] Banner message is human-readable: "Could not save project. The name is already taken."
[ ] Error code from API mapped to a human message — never show raw error codes to users
[ ] Loading state on submit button removed after error (button re-enabled)
[ ] User can correct and re-submit without refreshing the page
[ ] If multiple field errors returned from API: all shown simultaneously (not one at a time)
```

### API Error Code → User Message Mapping

| Error Code | Message to User |
|------------|----------------|
| `VALIDATION_ERROR` | Show field-specific messages |
| `DUPLICATE` | "A [entity] with this [field] already exists." |
| `NOT_FOUND` | "This [entity] was not found. It may have been deleted." |
| `FORBIDDEN` | "You don't have permission to [action]." |
| `AUTH_REQUIRED` | Redirect to login |
| `BUSINESS_RULE_VIOLATION` | Specific message per rule |
| `SERVER_ERROR` | "Something went wrong. Please try again." |

---

# 4. Page-Level Errors

When the page itself fails to load its data.

```text
[ ] Error state shown in the content area — not a blank page
[ ] Error message explains what went wrong: "Could not load projects."
[ ] "Try again" button that re-fetches the data
[ ] If partial data loads: show what loaded, with an error for what didn't
[ ] 404 (page not found): custom 404 page with helpful links
[ ] 403 (forbidden): "You don't have access to this page." with a link back to home
[ ] 500: "Something went wrong." with a link to go back or contact support
[ ] Never show a blank white page to the user — always show a meaningful state
```

---

# 5. Network & Connectivity Errors

```text
[ ] Network failure during form submit: show "Connection lost. Check your internet and try again." — do not lose form data
[ ] Network failure during page load: show error state with retry button
[ ] Request timeout: treat the same as a network failure — show error, allow retry
[ ] Slow connection: loading state must remain visible until the request completes or times out
[ ] After network error: form stays populated — user does not re-type
```

---

# 6. Permission Errors

```text
[ ] 403 on an action (delete, edit): show inline error message near the action — "You don't have permission to delete this project."
[ ] 403 on a page: render a "You don't have access" state in the page content area — not a blank page
[ ] Do not silently hide the button or content without telling the user why
[ ] If a user loses permission mid-session (role changed): show a clear message on next action
[ ] Do not expose sensitive details in permission error messages (what data exists, who has access)
```

---

# 7. Session & Auth Errors

```text
[ ] 401 from any API call: redirect to login page immediately
[ ] On redirect: show "Your session has expired. Please log in again." on the login page (not silently)
[ ] The page the user was on is saved and restored after re-login (intended URL redirect)
[ ] Token refresh attempted before showing 401 (if refresh token pattern is implemented)
[ ] Token refresh failure: log out user, redirect to login with session expired message
[ ] Do not show a 401 error toast — always redirect to login
```

---

# 8. Error Message Standards

```text
[ ] Written in plain language — no codes, no jargon, no stack traces
[ ] Specific — names the entity and field where possible
[ ] Helpful — tells the user what to do next (not just what went wrong)
[ ] Calm — no aggressive language, no ALL CAPS, no exclamation on error
[ ] Consistent — the same error from the same action always shows the same message
[ ] Respectful — does not blame the user ("Invalid input" not "You entered wrong data")
```

### Message Tone Examples

| Scenario | Good | Bad |
|----------|------|-----|
| Required field | "Email is required." | "MISSING FIELD" |
| Duplicate | "A project with this name already exists." | "Unique constraint violation" |
| Server error | "Something went wrong. Please try again." | "Error 500: Internal Server Error" |
| Permission | "You don't have permission to edit this." | "403: Access Denied" |
| Network | "Connection lost. Check your internet." | "TypeError: Failed to fetch" |

---

# 9. What Never to Do with Errors

```text
× console.log(error) without showing the user anything
× Reset the form after a failed submission (user loses their input)
× Show raw API error messages or stack traces to users
× Show "Success" when the action actually failed
× Silently ignore a failed API call
× Show a blank page when data fails to load
× Use an error toast for a 401 (always redirect to login)
× Show "Error" or "Something went wrong" without any way for the user to resolve it
× Show field errors only at the top of the form (also show them inline)
× Block the form permanently after an error (re-enable the submit button)
```

---

# 10. Error Handling Checklist — Before Marking Done

```text
FIELD ERRORS
[ ] Shown inline below the field on blur
[ ] Field outline changes to red on error
[ ] Message is specific and uses the field name
[ ] Scroll to first error on submit
[ ] Error clears when user starts correcting

FORM API ERRORS
[ ] Form stays populated after failed submission
[ ] Field-specific API errors shown inline
[ ] General API errors shown as banner above form
[ ] Banner is dismissable
[ ] Submit button re-enabled after error
[ ] API error codes mapped to human messages

PAGE ERRORS
[ ] Error state shown (not blank page)
[ ] Retry button present
[ ] 403: meaningful "no access" message
[ ] 404: custom not found page
[ ] 500: fallback error with navigation option

NETWORK ERRORS
[ ] Connection failure shows message to user
[ ] Form data preserved on network failure
[ ] Retry available

AUTH ERRORS
[ ] 401 → redirect to login with message
[ ] Intended page restored after re-login

STANDARDS
[ ] No console.log(error) without UI feedback
[ ] No raw error codes shown to users
[ ] No blank pages
[ ] All errors have a next step for the user
```
