# Notifications & Toasts Checklist

> **Core Rule:** Every async action must give the user feedback. Success, error, and warning notifications must be clear, consistent, and never silently disappear before the user can read them.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-when-to-use-each-notification-type) | When to Use Each Notification Type |
| [2](#2-toast-notifications) | Toast Notifications |
| [3](#3-inline-banners) | Inline Banners |
| [4](#4-persistent-alerts) | Persistent Alerts |
| [5](#5-notification-content-standards) | Notification Content Standards |
| [6](#6-what-always-triggers-a-notification) | What Always Triggers a Notification |
| [7](#7-what-never-triggers-a-toast) | What Never Triggers a Toast |
| [8](#8-notification-checklist--before-marking-done) | Notification Checklist — Before Marking Done |

---

# 1. When to Use Each Notification Type

| Type | Use When | Duration |
|------|----------|----------|
| **Toast** | Async action completed (create, update, delete, send) | 4–5 seconds, auto-dismiss |
| **Inline Banner** | Form-level error from API, permission error on a page | Persistent until dismissed or resolved |
| **Persistent Alert** | Account-level issue requiring attention (payment overdue, email unverified) | Persistent, manual dismiss |
| **Field Error** | Specific form field validation failure | Until the field is corrected |
| **Confirmation Dialog** | Before a destructive action | User must respond |

---

# 2. Toast Notifications

### Types

| Type | When | Color/Icon |
|------|------|-----------|
| **Success** | Action completed: created, saved, deleted, sent | Green ✓ |
| **Error** | Action failed: API error, server error | Red ✕ |
| **Warning** | Action completed with a caveat | Amber ⚠ |
| **Info** | Neutral information, non-action update | Blue ℹ |

### Placement & Behavior

```text
[ ] Position: top-right corner (desktop) / top-center or bottom-center (mobile)
[ ] Z-index above all content including modals
[ ] Success toasts: auto-dismiss after 4 seconds
[ ] Error toasts: auto-dismiss after 6 seconds (longer — user needs time to read)
[ ] Warning/info toasts: auto-dismiss after 5 seconds
[ ] User can manually dismiss any toast (X button always visible)
[ ] Hovering over a toast pauses its auto-dismiss timer
[ ] Multiple toasts stack vertically (not overlap)
[ ] Maximum 3 toasts visible at once — queue others
[ ] New toasts appear at the top of the stack, older ones shift down
```

### Toast Content

```text
[ ] Message is specific: "Project created." not "Success!"
[ ] Message is past tense for completed actions: "Project deleted." not "Project will be deleted."
[ ] Error messages are human-readable: "Could not save project. Please try again." not "Error 500"
[ ] Optional action button in toast: "Undo" for deletions, "View" for created items
[ ] Undo action available for 5 seconds after delete (if backend supports soft delete)
[ ] No technical jargon or internal error codes visible to the user
```

---

# 3. Inline Banners

Displayed inside the page or form — not floating. Used for errors that require the user's attention before they can continue.

```text
[ ] Shown at the top of the form or page section it relates to
[ ] Stays visible until the user dismisses it or the error is resolved
[ ] Includes an X button to manually dismiss
[ ] Color-coded: red for error, amber for warning, blue for info, green for success
[ ] Message explains what happened and what the user should do: "You don't have permission to edit this project. Contact your workspace admin."
[ ] API errors that apply to the whole form use an inline banner (not a toast that disappears)
[ ] Permission errors on a page use an inline banner — not a toast
[ ] Rate limit / quota errors use an inline banner
[ ] Includes a link or button if the user can take action to resolve it
```

---

# 4. Persistent Alerts

Account-level or workspace-level issues shown at the top of every page until resolved.

```text
[ ] Shown in a fixed alert bar below the navigation
[ ] Cannot be permanently dismissed if the issue is unresolved (can be temporarily hidden per session)
[ ] Clearly explains the issue and what to do: "Your free trial ends in 3 days. Upgrade to continue." with an "Upgrade" button
[ ] Removed automatically once the issue is resolved (e.g., email verified, payment updated)
[ ] Not used for one-off notifications — only for ongoing account states
```

---

# 5. Notification Content Standards

```text
[ ] Written in plain language — no internal codes, no stack traces
[ ] Specific — names the entity involved: "Project 'Marketing Q1' deleted." not "Item deleted."
[ ] Actionable where relevant — tells the user what to do next if the action failed
[ ] Consistent — same event always produces the same message across the app
[ ] Correct tense — completed action = past tense; "Project saved." not "Saving project."
[ ] No exclamation marks on error messages — tone should be calm and helpful
[ ] No all-caps — "ERROR" is aggressive; use "Could not save project."
```

### Message Examples

| Scenario | Good Message | Bad Message |
|----------|-------------|-------------|
| Project created | "Project created successfully." | "Success!" |
| Project deleted | "Project deleted." | "Done." |
| Save failed | "Could not save project. Please try again." | "Error 500" |
| No permission | "You don't have permission to view this." | "403 Forbidden" |
| Network error | "Connection lost. Check your internet and try again." | "Network Error" |
| Session expired | "Your session has expired. Please log in again." | "401 Unauthorized" |

---

# 6. What Always Triggers a Notification

```text
[ ] Record created (success toast)
[ ] Record updated / saved (success toast)
[ ] Record deleted (success toast + optional undo)
[ ] File uploaded (success toast)
[ ] Invitation sent (success toast)
[ ] Any async action that fails (error toast or inline banner)
[ ] Permission denied when attempting an action (inline banner or toast)
[ ] Session expired (redirect to login with a message on the login page)
[ ] Backend validation error on form submission (inline field errors + optional banner)
```

---

# 7. What Never Triggers a Toast

```text
× Page load / data fetch (no "Data loaded" success toast)
× Navigation between pages
× Opening a modal
× Typing in a form field
× Client-side validation failure (use inline field errors instead)
× Background sync / auto-save (use a subtle "Saved" indicator, not a toast)
× Hover / focus events
```

---

# 8. Notification Checklist — Before Marking Done

```text
TOAST USAGE
[ ] Every async action (create/update/delete/send) shows a toast on success
[ ] Every async failure shows an error toast or inline banner
[ ] No action completes silently without user feedback
[ ] Toast position: top-right (desktop), top or bottom center (mobile)

TOAST BEHAVIOR
[ ] Success toasts: 4 seconds auto-dismiss
[ ] Error toasts: 6 seconds auto-dismiss
[ ] X button on every toast for manual dismiss
[ ] Hover pauses auto-dismiss timer
[ ] Multiple toasts stack, max 3 visible

CONTENT
[ ] Messages are specific (name the entity)
[ ] Messages are past tense for completed actions
[ ] Error messages are human-readable (no codes or jargon)
[ ] No exclamation marks on errors
[ ] Consistent — same event = same message everywhere

INLINE BANNERS
[ ] Whole-form API errors use inline banner (not a toast)
[ ] Permission errors use inline banner
[ ] Banner includes dismiss button
[ ] Banner explains what to do (not just what went wrong)

COVERAGE
[ ] create → success toast
[ ] update → success toast
[ ] delete → success toast
[ ] send/invite → success toast
[ ] any API error → error toast or banner
[ ] no silent failures anywhere
```
