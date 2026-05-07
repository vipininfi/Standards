# Modals & Dialogs Checklist

> **Core Rule:** A modal must be openable, closeable, and accessible. It must trap focus inside, respond to the Escape key, block background scroll, and handle loading/error states for any async action inside it.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-opening-a-modal) | Opening a Modal |
| [2](#2-closing-a-modal) | Closing a Modal |
| [3](#3-modal-content-structure) | Modal Content Structure |
| [4](#4-confirmation-modals) | Confirmation Modals |
| [5](#5-form-modals) | Form Modals |
| [6](#6-accessibility) | Accessibility |
| [7](#7-stacking--nested-modals) | Stacking & Nested Modals |
| [8](#8-mobile) | Mobile |
| [9](#9-modal-checklist--before-marking-done) | Modal Checklist — Before Marking Done |

---

# 1. Opening a Modal

```text
[ ] Modal is triggered by a clear user action (button click) — not on page load unless it is an intentional interruption (e.g., cookie consent)
[ ] Opening animation is present (fade in or slide up — keep it under 200ms)
[ ] Background (backdrop) darkens to signal the modal is blocking
[ ] Background scroll is locked when modal is open (body overflow: hidden)
[ ] Focus moves to the modal on open — not left on the trigger button behind the backdrop
[ ] First focusable element inside the modal receives focus on open
[ ] Screen reader announces the modal is open (role="dialog", aria-modal="true", aria-labelledby)
```

---

# 2. Closing a Modal

```text
[ ] Close button (X) visible in the top-right corner of the modal
[ ] Clicking the backdrop closes the modal (unless the modal contains an incomplete, unsaved form)
[ ] Pressing Escape key closes the modal
[ ] Closing animation matches the open animation (fade out / slide down)
[ ] Focus returns to the trigger button that opened the modal on close
[ ] Background scroll is restored on close
[ ] If the modal contains unsaved changes: show a "Discard changes?" confirmation before closing
[ ] If a loading operation is in progress: Escape and backdrop click are disabled until it completes
```

---

# 3. Modal Content Structure

```text
[ ] Modal has a visible title (h2 or h3 — not just bold text)
[ ] Title is connected to the modal via aria-labelledby
[ ] Modal has a clear primary action button and a cancel/close button
[ ] Primary action is on the right, cancel on the left (or bottom)
[ ] Destructive primary actions are styled in red/danger color
[ ] Modal content does not overflow the viewport — scrollable content area inside the modal, not the page
[ ] Modal width is appropriate for content (confirm dialogs: narrow; forms: medium; preview: wider)
[ ] Modal is centered both vertically and horizontally
[ ] Modal does not exceed the viewport height without an internal scroll area
```

---

# 4. Confirmation Modals

Used before any destructive or irreversible action (delete, archive, remove member, etc.).

```text
[ ] Title states what is being confirmed: "Delete Project?" not "Are you sure?"
[ ] Body explains the consequence: "This will permanently delete the project and all its tasks."
[ ] Body names the specific item being deleted: "Delete 'Marketing Campaign 2024'?"
[ ] Primary (destructive) button label states the action: "Delete Project" not "Confirm" or "OK"
[ ] Cancel button is clearly visible and labeled "Cancel"
[ ] Primary button is styled in danger/red
[ ] Loading state on primary button while the action is in progress
[ ] On success: modal closes, success toast shown, list refreshes
[ ] On error: modal stays open, error message shown inside the modal
```

### Levels of Confirmation

| Risk Level | Example | Confirmation Type |
|------------|---------|------------------|
| Low | Archive project | Simple confirm modal |
| Medium | Delete project (recoverable) | Confirm modal with consequence explanation |
| High | Delete account / delete all data | Type the name to confirm (see [Delete Checklist](delete-destructive-actions.md)) |

---

# 5. Form Modals

Modal that contains a form (e.g., Add Project, Invite Member).

```text
[ ] Same component used for Add and Edit (see [Add/Edit Consistency](add-edit-consistency.md))
[ ] Form validation runs before submission — inline errors shown inside the modal
[ ] API/server errors shown inside the modal (not in a toast that disappears immediately)
[ ] Loading state on submit button — button disabled during submission
[ ] No double submit possible
[ ] On success: modal closes, success toast shown, parent list refreshes
[ ] On error: modal stays open, form stays populated, error shown
[ ] Cancel button closes modal — if form has unsaved input, confirm "Discard changes?"
[ ] Modal scrolls internally if form is long (not the page behind it)
[ ] Tab order is logical — loops inside the modal, does not escape to background
```

---

# 6. Accessibility

```text
[ ] role="dialog" on the modal root element
[ ] aria-modal="true" on the modal root element
[ ] aria-labelledby pointing to the modal title element
[ ] aria-describedby pointing to the modal description/body (if applicable)
[ ] Focus is trapped inside the modal — Tab and Shift+Tab cycle through modal elements only
[ ] Escape key closes the modal
[ ] Focus returns to the trigger element when modal closes
[ ] Background content has aria-hidden="true" while modal is open (so screen readers only read modal content)
[ ] All interactive elements inside modal are keyboard-accessible
```

---

# 7. Stacking & Nested Modals

Avoid nesting modals whenever possible — re-think the UX first.

```text
[ ] Nested modals avoided where possible (use inline confirm inside the parent modal instead)
[ ] If nested: second modal has higher z-index, second backdrop darkens on top of first
[ ] Escape closes only the topmost modal (not all modals at once)
[ ] Focus management works correctly: moves to second modal on open, returns to first modal on close
[ ] Max nesting depth: 2 levels — never 3+
```

---

# 8. Mobile

```text
[ ] Modal is full-screen or bottom sheet on mobile (< 768px) — not a small centered box
[ ] Bottom sheet slides up from the bottom on mobile (preferred for action confirmation)
[ ] Scrollable content area within the sheet for long forms
[ ] Close handle visible at the top of the bottom sheet (drag indicator bar)
[ ] Tap outside / swipe down closes the modal (with unsaved changes protection)
[ ] Keyboard does not push the modal content out of view on mobile (viewport height management)
[ ] Touch targets inside the modal are at least 44×44px
```

---

# 9. Modal Checklist — Before Marking Done

```text
OPEN
[ ] Triggered by user action
[ ] Backdrop darkens
[ ] Page scroll locked
[ ] Focus moves to modal on open

CLOSE
[ ] X button visible and works
[ ] ESC key closes modal
[ ] Clicking backdrop closes modal (unless unsaved form)
[ ] Focus returns to trigger on close
[ ] Scroll restored on close
[ ] Unsaved changes: confirmation before closing

CONTENT
[ ] Modal has a visible, descriptive title
[ ] Primary and cancel buttons both present
[ ] Content scrolls inside modal (not the page)

CONFIRMATION MODAL (if applicable)
[ ] Title names the action
[ ] Body names the specific item and consequence
[ ] Primary button label is the action ("Delete", not "OK")
[ ] Destructive button is red/danger styled
[ ] Loading state on primary button

FORM MODAL (if applicable)
[ ] Same component as non-modal form (if same entity)
[ ] Validation errors shown inside modal
[ ] API errors shown inside modal
[ ] Success: closes modal, refreshes list, shows toast
[ ] Error: modal stays open, form stays populated

ACCESSIBILITY
[ ] role="dialog" and aria-modal="true"
[ ] aria-labelledby on title
[ ] Focus trapped inside modal
[ ] Background aria-hidden while modal open

MOBILE
[ ] Full-screen or bottom sheet on mobile
[ ] Touch targets ≥ 44×44px
[ ] Keyboard does not hide modal content
```
