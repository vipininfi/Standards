# Settings & Profile Pages Checklist

> **Core Rule:** Settings pages must never silently save or silently fail. Every save action has a clear loading state, success confirmation, and error message. Destructive settings (delete account, disconnect integration) follow the destructive actions standard.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-page-structure) | Page Structure |
| [2](#2-general-settings-fields) | General Settings Fields |
| [3](#3-save-behavior) | Save Behavior |
| [4](#4-password-change-section) | Password Change Section |
| [5](#5-avatar--profile-photo) | Avatar & Profile Photo |
| [6](#6-notification-preferences) | Notification Preferences |
| [7](#7-connected-accounts--integrations) | Connected Accounts & Integrations |
| [8](#8-danger-zone) | Danger Zone |
| [9](#9-settings-checklist--before-marking-done) | Checklist — Before Marking Done |

---

# 1. Page Structure

```text
[ ] Settings divided into clear sections with headings (Profile, Security, Notifications, Integrations, Danger Zone)
[ ] Each section is visually distinct (separator, card, or panel per section)
[ ] Navigation: if many sections, use a sidebar or tab navigation rather than one long scrolling page
[ ] Active section is highlighted in the navigation
[ ] Page title: "Settings" or "Account Settings" — clearly labeled
[ ] Breadcrumb or back navigation available
[ ] Settings changes do not affect other users unless explicitly a team/workspace setting
[ ] Workspace settings and personal/account settings are separated clearly
```

---

# 2. General Settings Fields

```text
[ ] First name, last name: same validation as registration (min 1 char, max 100)
[ ] Email change: requires current password confirmation (security check)
[ ] Email change: sends a verification email to the new address before the change takes effect
[ ] Email change: confirmation email sent to the old address notifying of the change
[ ] Username (if applicable): unique, validated, alphanumeric + hyphens only
[ ] Timezone: timezone selector (not a freetext field)
[ ] Language/locale: dropdown with supported languages
[ ] All fields pre-filled with current values — never shown blank
[ ] All fields validated on the same rules as registration
```

---

# 3. Save Behavior

```text
[ ] Explicit Save button per section (preferred) OR auto-save with a visible "Saved" indicator
  — Never auto-save silently with no feedback
  — If auto-save: show "Saving..." while saving and "Saved ✓" after completion
[ ] Per-section save: only the section being edited is submitted — not the entire settings page
[ ] Save button shows loading state while saving
[ ] Save button disabled during loading
[ ] On success: show a success toast ("Profile updated.") OR inline "Saved ✓" near the button
[ ] On error: show error inline or as a banner — do NOT reset the field to the old value silently
[ ] Unsaved changes indicator: if user tries to navigate away with unsaved changes, show "Unsaved changes. Leave anyway?"
[ ] After save: fields stay populated with the new values (not reset to previous)
```

---

# 4. Password Change Section

```text
[ ] Separate section from profile details — never combined in the same form
[ ] Fields:
  — Current password (required)
  — New password (required, same complexity rules as registration)
  — Confirm new password (required, must match new password)
[ ] All three fields are password inputs with show/hide toggles
[ ] Current password validated on backend — frontend cannot bypass this
[ ] New password complexity rules shown clearly (same as registration)
[ ] Confirm new password validates in real time once both new password fields have values
[ ] New password = current password: blocked on backend, shown to user: "Your new password must be different."
[ ] Fields are separate from other settings — submitting password form does NOT submit profile form
[ ] On success: clear all three password fields, show success toast: "Password updated."
[ ] On error (wrong current password): show inline error on current password field: "Current password is incorrect."
[ ] After successful password change: optionally prompt user to log out other sessions
```

---

# 5. Avatar & Profile Photo

```text
[ ] Upload button triggers a file input (not a drag-and-drop-only zone — both preferred)
[ ] Accepted file types: JPEG, PNG, WebP — validated by MIME type (not just extension)
[ ] Maximum file size: 5MB (show error if exceeded: "Image must be smaller than 5MB.")
[ ] Minimum dimensions: 100×100px recommended — reject images too small
[ ] Maximum dimensions: enforce via server-side processing (resize on upload)
[ ] Preview shown before confirming the upload
[ ] Upload has a loading/progress indicator
[ ] On success: avatar updates immediately in the UI (including header/nav avatar)
[ ] Fallback: if no avatar set, show user's initials in a colored circle
[ ] Remove photo option: removes avatar and reverts to initials fallback
[ ] Backend strips EXIF data from uploaded images (privacy — removes GPS location)
```

---

# 6. Notification Preferences

```text
[ ] Toggles/checkboxes for each notification type — not a freetext field
[ ] Notification categories clearly labeled (e.g., "Project updates", "Mentions", "Weekly digest")
[ ] Each toggle saves immediately on change (no separate save button for toggles)
[ ] Saving a toggle shows a brief "Saved" confirmation near the toggle (not a toast — too disruptive for toggles)
[ ] Unsubscribing from all emails must include a confirmation: "You will not receive any emails from us."
[ ] Users can re-subscribe at any time from the same settings page
[ ] Email notification unsubscribe link in emails links to this section
[ ] Do NOT allow users to disable critical security emails (password change, account deletion)
```

---

# 7. Connected Accounts & Integrations

```text
[ ] Connected accounts shown with: provider logo, connected email/username, connection date
[ ] "Disconnect" button for each connected account
[ ] Disconnecting the last login method: warn "This is your only login method. Set a password before disconnecting."
[ ] OAuth token shown as "Connected" — never show the raw token
[ ] Reconnect option if token is expired or revoked
[ ] Third-party integrations (Slack, GitHub, etc.) show connection status, scopes, and last used date
[ ] Revoking a third-party integration: confirm "Disconnect [App]? This will remove its access to your account." before proceeding
[ ] Disconnect is a destructive action — follow [Destructive Actions Checklist](delete-destructive-actions.md)
```

---

# 8. Danger Zone

The Danger Zone section contains irreversible or high-impact actions.

```text
[ ] Danger Zone is visually separated from the rest of the settings (red border or distinct background)
[ ] Labeled clearly: "Danger Zone" or "Account Deletion"
[ ] Only irreversible or high-impact actions in this section:
  — Delete account
  — Leave workspace
  — Transfer workspace ownership
  — Cancel subscription
[ ] Every action in Danger Zone follows the [Destructive Actions Checklist](delete-destructive-actions.md)
[ ] Delete account: type-to-confirm required ("Type DELETE to confirm")
[ ] Delete account: explain what will be deleted: "All your data, workspaces, and files will be permanently deleted."
[ ] Workspace owner cannot delete account without transferring ownership first: "Transfer workspace ownership before deleting your account."
[ ] After successful account deletion: log out and redirect to homepage
```

---

# 9. Settings Checklist — Before Marking Done

```text
STRUCTURE
[ ] Sections clearly labeled and separated
[ ] Navigation between sections (sidebar or tabs) if multiple sections
[ ] Personal vs workspace settings separated

FIELDS
[ ] All fields pre-filled with current values
[ ] Same validation as registration
[ ] Email change: requires password confirmation and sends verification

SAVE BEHAVIOR
[ ] Explicit save button per section OR auto-save with visible indicator
[ ] Loading state on save button
[ ] Success: toast or inline "Saved ✓"
[ ] Error: shown inline, fields not reset silently
[ ] Unsaved changes: warning on navigation away

PASSWORD CHANGE
[ ] Separate from profile form
[ ] Current password + new + confirm
[ ] Show/hide toggles on all three
[ ] Complexity rules enforced and shown
[ ] Wrong current password: inline error
[ ] Success: fields cleared, toast shown

AVATAR
[ ] MIME type validated (not just extension)
[ ] File size limit enforced with clear error
[ ] Preview before confirm
[ ] Loading indicator during upload
[ ] Immediate UI update after success
[ ] Fallback initials if no avatar

DANGER ZONE
[ ] Visually separated
[ ] Every action has confirmation (medium or high risk level)
[ ] Delete account: type-to-confirm
[ ] Owner transfer required before delete if owner

MOBILE
[ ] All sections accessible on mobile
[ ] Toggle switches usable on touch
[ ] File upload works on mobile (native camera option if applicable)
```
