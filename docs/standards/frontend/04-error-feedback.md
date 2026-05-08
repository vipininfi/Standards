# Frontend Standards — Error & Feedback

> No error is silent. No action is invisible. Every failure tells the user what happened and what to do next.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-error-types--display-locations) | Error Types & Display Locations |
| [2](#2-field-level-errors) | Field-Level Errors |
| [3](#3-form-api-errors) | Form API Errors |
| [4](#4-page-level-errors) | Page-Level Errors |
| [5](#5-api-error-code-mapping) | API Error Code Mapping |
| [6](#6-empty-states) | Empty States |
| [7](#7-destructive-action-confirmation) | Destructive Action Confirmation |
| [8](#8-network-errors) | Network Errors |
| [9](#9-auth-errors) | Auth Errors |
| [10](#10-message-standards) | Message Standards |
| [11](#11-no-silent-failures) | No Silent Failures |

---

# 1. Error Types & Display Locations

| Error Source | What It Is | Where to Show It |
|-------------|-----------|-----------------|
| Client-side validation | Required field, format, length | Below the field (field error) |
| API field error | Duplicate, constraint failure | Below the field + form banner |
| API business rule | Permission, state conflict | Form banner or inline alert |
| API server error | 500, unexpected | Form banner or page error banner |
| Page load failure | Data could not be fetched | Full page error state with retry |
| Network failure | No connection, timeout | Toast or inline banner |
| Auth expiry | Session expired | Redirect to login (not a blank page) |

---

# 2. Field-Level Errors

```text
[ ] Message appears directly below the field it belongs to
[ ] Red text + red border on the field
[ ] Specific message: "Email is required." — not "Required."
[ ] aria-describedby on the input pointing to the error element
[ ] Error cleared when user corrects the field and moves on (on blur/re-submit)
[ ] Never clear an error while the user is still typing
[ ] Never show the same error twice in the same form
```

---

# 3. Form API Errors

```text
[ ] API errors shown in a red banner above the submit button (or top of form)
[ ] Banner stays visible — does not auto-dismiss
[ ] Form stays open with data intact — user can correct and retry
[ ] If the API error maps to a specific field: show it as a field error, not just in banner
[ ] Banner message is specific: "A project with this name already exists."
[ ] Never show raw API error strings, status codes, or stack traces to the user
[ ] Banner dismissed when user successfully resubmits
```

---

# 4. Page-Level Errors

```text
[ ] If page data fails to load: show an error state (not a blank page)
[ ] Error state contains:
    — Friendly message: "Could not load projects. Please try again."
    — Retry button (re-triggers the original fetch)
[ ] If data was previously loaded and a refresh fails: show error banner above the existing data
[ ] Never replace loaded data with an error state — show the banner, keep the data
[ ] 404 Not Found: show a "This page doesn't exist." state with a link back to the list
[ ] 403 Forbidden: show a "You don't have access to this page." state — not a blank page
[ ] 401 Unauthorized: redirect to login (not a blank screen)
```

---

# 5. API Error Code Mapping

Map backend error codes to user-facing messages on the frontend. Never show raw codes.

| Backend Code | User-Facing Message |
|-------------|-------------------|
| `VALIDATION_ERROR` | Show the specific field message from the error detail |
| `DUPLICATE` | "A [entity] with this name already exists." |
| `NOT_FOUND` | "This item was not found. It may have been deleted." |
| `FORBIDDEN` | "You don't have permission to do this." |
| `AUTH_REQUIRED` | Redirect to login |
| `BUSINESS_RULE_VIOLATION` | Show the specific reason from the error detail |
| `SERVER_ERROR` | "Something went wrong. Please try again." |
| Network / no response | "Connection lost. Check your internet connection." |

```ts
// Standard pattern for parsing Supabase/backend error codes
const { data, error } = await supabase.rpc('create_project', params);

if (error) {
  const code = error.message.split(':')[0].trim();
  const msg  = error.message.split(':').slice(1).join(':').trim();

  switch (code) {
    case 'VALIDATION_ERROR':          return showFieldError(msg);
    case 'DUPLICATE':                 return showFormError('A project with this name already exists.');
    case 'FORBIDDEN':                 return showFormError("You don't have permission to do this.");
    case 'NOT_FOUND':                 return showFormError('This item was not found. It may have been deleted.');
    case 'BUSINESS_RULE_VIOLATION':   return showFormError(msg);
    default:                          return showFormError('Something went wrong. Please try again.');
  }
}
```

---

# 6. Empty States

```text
NO DATA EXISTS
[ ] Illustration or icon — not just text
[ ] Specific message: "No projects yet." — not "No data." or "Empty."
[ ] Call to action: "Create your first project" button or link

SEARCH RETURNED NO RESULTS
[ ] "No results for '[search term]'."
[ ] "Try a different search term."
[ ] Clear search button visible

FILTER RETURNED NO RESULTS
[ ] "No [entities] match your current filters."
[ ] Shows what filters are active
[ ] "Clear all filters" button prominent

COMBINATION (search + filters active)
[ ] "No results for '[term]' with these filters."
[ ] Both "Clear search" and "Clear filters" available

RULE: Never show a blank table body or an empty list without a message.
```

---

# 7. Destructive Action Confirmation

Before any irreversible action, show a confirmation dialog.

```text
LOW RISK (Archive, soft actions)
  Title:  "Archive Project?"
  Body:   What happens + whether reversible: "This project will be hidden from your workspace. You can restore it later."
  Buttons: "Cancel" (left), amber or red action button (right): "Archive"

MEDIUM RISK (Delete a record)
  Title:  "Delete Project?"
  Body:   Item name + consequence: "'Marketing Q1' will be permanently deleted along with all its tasks."
  Body:   "This action cannot be undone."
  Buttons: "Cancel" (left), "Delete Project" — red/danger (right)

HIGH RISK (Delete account, all data, bulk delete)
  Same as medium +
  Type-to-confirm input: "Type DELETE to confirm"
  Submit button: disabled until input matches exactly (case-sensitive)
  Buttons: "Cancel" (left), "Delete Everything" — red/danger, disabled until confirmed (right)
```

---

# 8. Network Errors

```text
[ ] Show an error when a request fails due to no connection or timeout
[ ] "Connection lost. Check your internet connection." — not a blank response
[ ] Retry button where appropriate (especially for page loads)
[ ] If the user is filling a form: do not clear the form on network error
[ ] Request timeout: treat the same as an error — show the error state
[ ] Do not silently swallow network errors and show a success toast
```

---

# 9. Auth Errors

```text
[ ] Expired session: redirect to login — do not show a blank screen
[ ] After login redirect: return user to the page they were on (use redirect param)
[ ] 401 from an API call mid-session: show "Your session has expired. Please log in again." and redirect
[ ] Do not show the user a raw "401 Unauthorized" message
[ ] Forbidden (403) action: show inline message — do not redirect unless the whole page is inaccessible
[ ] Token refresh: handled silently — user should not see a flash or error during refresh
```

---

# 10. Message Standards

```text
TONE
[ ] Friendly and specific — not technical
[ ] No "Error", "Failure", "Exception" as the full message
[ ] No exclamation marks on error messages
[ ] Past tense for completed: "Project saved." "Changes discarded."
[ ] Present tense for ongoing: "Saving..." "Deleting..."
[ ] No ALL CAPS in messages

SPECIFICITY
Good:  "Password must be at least 8 characters."
Bad:   "Invalid input."

Good:  "A project named 'Website Redesign' already exists in this workspace."
Bad:   "Duplicate error."

Good:  "Could not load team members. Please try again."
Bad:   "Error 500"

NO TECHNICAL DETAILS TO USERS
[ ] No error codes (SQLSTATE, HTTP status codes)
[ ] No stack traces
[ ] No variable names or internal identifiers
[ ] No "undefined", "null", "NaN" in user-facing messages
```

---

# 11. No Silent Failures

These are the most dangerous violations — the user thinks something worked when it didn't.

```text
NEVER:
× console.log(error) without showing anything in the UI
× Catch an error and return undefined / null without showing a message
× Show a success toast after a failed operation
× Let a form submit without feedback (no spinner, no error, no success)
× Show "Saved" when the save actually failed
× Catch errors in a catch block with an empty body: catch (e) {}
× Swallow a network error and show stale data as if fresh
× Not handle the error state of a Promise or async/await call

ALWAYS:
✓ Every try/catch must show an error in the UI if the catch fires
✓ Every async call must have an error handler
✓ Every error handler must update UI state — not just log to console
✓ If you show a loading spinner, you must also handle the error that cancels it
```

---

## Practice Task

**→ [Add/Edit Project Form](../../../tasks/frontend/03-add-edit-project-form.md)**
Build a project form that correctly handles field errors, API errors (duplicate, forbidden), and network failures — with no silent failures.
