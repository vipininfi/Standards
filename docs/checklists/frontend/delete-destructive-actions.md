# Delete & Destructive Actions Checklist

> **Core Rule:** No destructive action executes without confirmation. No confirmation is vague. No deletion is silent. The user must always know what they are destroying and what the consequences are.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-risk-levels) | Risk Levels |
| [2](#2-confirmation-dialog-standards) | Confirmation Dialog Standards |
| [3](#3-low-risk-simple-confirm) | Low Risk — Simple Confirm |
| [4](#4-medium-risk--named-confirm) | Medium Risk — Named Confirm |
| [5](#5-high-risk--type-to-confirm) | High Risk — Type to Confirm |
| [6](#6-soft-delete-vs-hard-delete) | Soft Delete vs Hard Delete |
| [7](#7-bulk-delete) | Bulk Delete |
| [8](#8-after-deletion) | After Deletion |
| [9](#9-destructive-actions-checklist--before-marking-done) | Destructive Actions Checklist — Before Marking Done |

---

# 1. Risk Levels

Every destructive action falls into one of three risk levels. Choose the confirmation type accordingly.

| Risk Level | Examples | Confirmation Type |
|------------|---------|------------------|
| **Low** | Archive a project, remove a tag, clear a filter | Simple confirm modal |
| **Medium** | Delete a record (soft delete / recoverable), remove a member | Named confirm modal |
| **High** | Permanently delete account, delete all data, cancel a subscription | Type-to-confirm modal |

When in doubt: go one level higher. It is better to over-confirm than to allow accidental permanent data loss.

---

# 2. Confirmation Dialog Standards

Applies to all risk levels:

```text
[ ] Confirmation is a modal dialog — not a browser alert() or confirm()
[ ] Dialog blocks all background interaction (backdrop present, background aria-hidden)
[ ] Dialog opens immediately when the destructive action is triggered
[ ] Title of the dialog states the action: "Delete Project?" not "Are you sure?"
[ ] Body names the specific item: "Delete 'Marketing Q1'?" not "Delete this project?"
[ ] Body explains the consequence: "This will permanently delete the project and all 14 tasks."
[ ] Primary (destructive) button is red/danger styled
[ ] Primary button label is the action: "Delete Project" not "OK" or "Confirm"
[ ] Cancel button is clearly labeled "Cancel" and is the safe/default option
[ ] Cancel button is visually less prominent than the destructive action
[ ] Pressing Escape cancels the dialog (same as clicking Cancel)
[ ] Clicking the backdrop cancels the dialog
[ ] Loading state on the primary button while the deletion is in progress
[ ] Primary button disabled after clicking (prevents double-delete)
[ ] On success: dialog closes, success toast shown, UI updates
[ ] On error: dialog stays open, error shown inside the dialog, user can retry
```

---

# 3. Low Risk — Simple Confirm

Used for: archiving, removing a tag, removing a non-critical association.

```text
Title:   "Archive Project?"
Body:    "This project will be archived and hidden from the active list. You can restore it later."
Button:  "Archive Project" (amber/warning styled — not red, since it's reversible)
Cancel:  "Cancel"
```

```text
[ ] Dialog has a title with the specific action
[ ] Body explains the consequence and whether it is reversible
[ ] Primary button label = the action
[ ] If reversible: button is amber/warning, not red
[ ] If irreversible: button is red/danger
[ ] Escape and backdrop close the dialog
```

---

# 4. Medium Risk — Named Confirm

Used for: deleting a record, removing a workspace member, cancelling an order.

```text
Title:   "Delete Project?"
Body:    "Deleting 'Marketing Q1' will permanently remove it and all 14 tasks inside it."
         "This action cannot be undone."
Button:  "Delete Project" (red/danger styled)
Cancel:  "Cancel"
```

```text
[ ] Title names the action and entity type
[ ] Body names the specific item (its name/title — not "this project")
[ ] Body states the count of dependent records being deleted if applicable
[ ] "This action cannot be undone." included if it is truly irreversible
[ ] Red/danger button styling
[ ] Loading state on button during deletion
[ ] Error shown inside dialog if deletion fails
```

---

# 5. High Risk — Type to Confirm

Used for: permanent account deletion, deleting all workspace data, irreversible bulk operations.

```text
Title:   "Delete Your Account"
Body:    "This will permanently delete your account, all your workspaces, and all associated data.
          This cannot be recovered.
          
          Type DELETE to confirm:"
Input:   Text field (user must type "DELETE" or the account name)
Button:  "Delete Account" — disabled until the correct text is typed
Cancel:  "Cancel"
```

```text
[ ] Input field requires specific text before the primary button is enabled
  — Use "DELETE" for generic deletions
  — Use the entity name for named deletions ("type the project name to confirm")
[ ] Input match is case-sensitive
[ ] Primary button remains disabled until input matches exactly
[ ] Primary button enabled only when input matches — never on partial match
[ ] Input field has placeholder showing what to type: placeholder="Type DELETE to confirm"
[ ] Loading state on primary button after it is clicked (even with type-to-confirm)
[ ] Error shown inside dialog if deletion fails
[ ] If user closes dialog: input field is reset (so they have to type again on re-open)
```

---

# 6. Soft Delete vs Hard Delete

```text
[ ] Soft delete preferred for most user-facing deletions:
  — Record has a deleted_at column set to the current timestamp
  — Record is hidden from normal queries but still exists in the database
  — Provides a recovery path (admin or undo action)

[ ] Hard delete only when:
  — Legal/compliance requires data erasure (GDPR right to erasure)
  — Data is truly ephemeral (e.g., temporary session data)
  — Explicitly designed and documented as permanent

[ ] If soft delete: "Undo" action available in toast for 5–10 seconds after deletion
[ ] If soft delete: admin interface or recovery flow exists (not just hidden data)
[ ] If hard delete: "This action cannot be undone." clearly stated in confirmation
[ ] Backend enforces the deletion type — frontend never decides alone
```

---

# 7. Bulk Delete

```text
[ ] Confirmation shows count: "Delete 5 projects?" not just "Delete selected?"
[ ] Body explains the full consequence: "This will permanently delete 5 projects and all tasks inside them."
[ ] If any of the 5 cannot be deleted (permission): mention it: "3 of 5 projects will be deleted. 2 cannot be deleted due to insufficient permissions."
[ ] Loading state shown during bulk operation
[ ] Success: "5 projects deleted." toast shown
[ ] Partial success: "3 of 5 projects deleted. 2 could not be deleted." with details if possible
[ ] Error: clear message stating what failed and why
[ ] Table/list refreshes after bulk delete completes
[ ] Selection state cleared after operation
```

---

# 8. After Deletion

```text
[ ] Success toast shown: "Project deleted." (specific, named)
[ ] If undo is supported: "Project deleted. Undo" toast with 5-second countdown
[ ] If the deleted item was the current page (viewing project → deleted): redirect to the list page
[ ] If the deleted item was in a list: remove it from the list without a full page refresh
[ ] If deleting the last item in a list: show empty state
[ ] Selection cleared in tables after delete
[ ] Page/list count updated to reflect the deletion
[ ] No "ghost" row left in the table after deletion
```

---

# 9. Destructive Actions Checklist — Before Marking Done

```text
RISK LEVEL ASSIGNED
[ ] Action categorized as Low / Medium / High risk
[ ] Correct confirmation type chosen for the risk level

CONFIRMATION DIALOG
[ ] Title states the action ("Delete Project?" not "Confirm")
[ ] Body names the specific item (not "this item")
[ ] Body explains the full consequence (what is deleted, what else is affected)
[ ] "Cannot be undone" included if truly irreversible
[ ] Destructive button is red/danger styled
[ ] Destructive button label = the action ("Delete Project" not "OK")
[ ] Cancel button visible and labeled "Cancel"
[ ] Escape key and backdrop click = Cancel
[ ] Loading state on destructive button while in progress
[ ] Double-click prevented (button disabled after first click)
[ ] Error shown inside dialog on failure (dialog stays open)

HIGH RISK SPECIFIC
[ ] Type-to-confirm input present
[ ] Case-sensitive exact match required
[ ] Button disabled until match is correct
[ ] Input resets if dialog is closed and reopened

AFTER DELETION
[ ] Success toast shown (names the entity)
[ ] Undo option available if soft delete
[ ] List updates without full page refresh
[ ] Redirect if current page was deleted
[ ] Empty state shown if last item deleted

BACKEND
[ ] Backend performs the actual delete — not just frontend hiding
[ ] Soft delete vs hard delete decided and implemented on backend
[ ] Frontend does not trust itself to enforce deletion rules
```
