# Frontend Standards — Settings & Profile

> Settings pages are the most trust-sensitive part of an app. Every change must be confirmed, every destructive action must be gated, and every section must be clear about what it controls.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-page-structure) | Page Structure |
| [2](#2-general-settings-fields) | General Settings Fields |
| [3](#3-save-behavior) | Save Behavior |
| [4](#4-password-change) | Password Change |
| [5](#5-avatar--profile-image) | Avatar & Profile Image |
| [6](#6-notification-preferences) | Notification Preferences |
| [7](#7-connected-accounts) | Connected Accounts |
| [8](#8-danger-zone) | Danger Zone |
| [9](#9-workspace--team-settings) | Workspace & Team Settings |

---

# 1. Page Structure

```text
[ ] Settings page divided into named sections (not one massive form)
[ ] Each section has a heading and save button — not one save button for all sections
[ ] Sections: Profile, Account (email/password), Notifications, Billing, Danger Zone
[ ] Danger zone is always at the bottom, visually separated, with a red border
[ ] Breadcrumb or back link to return to previous context
[ ] Mobile: sections stack vertically with clear separators
[ ] Side navigation (desktop) or segmented control (mobile) for section switching
[ ] Active section highlighted in nav
```

---

# 2. General Settings Fields

```text
NAME
[ ] First name + last name (or display name — pick one, be consistent)
[ ] Max length validated: 100 characters
[ ] Changes take effect on save — not immediately on typing

EMAIL
[ ] Email change requires password confirmation: "Enter your current password to change email"
[ ] New email must be verified before it becomes active
[ ] "Verification email sent to [new email]." shown after submit
[ ] User continues to log in with old email until verified
[ ] "Resend verification" button available

TIMEZONE
[ ] User can set their preferred timezone
[ ] Stored as IANA timezone string: "America/New_York"
[ ] Default: auto-detected from browser
[ ] Affects date/time display throughout the app

LANGUAGE
[ ] Dropdown of supported languages
[ ] Page updates on language change (or on save + reload)
```

---

# 3. Save Behavior

```text
EXPLICIT SAVE (Preferred for most settings)
[ ] "Save Changes" button below each section
[ ] Button disabled until a field has been changed
[ ] Loading state during save
[ ] Success: inline "Saved." confirmation near the button (not a toast for settings)
[ ] Error: inline error message below the button (not a toast)

AUTO-SAVE (Acceptable for notification preferences, toggles)
[ ] Save triggered immediately on toggle/checkbox change
[ ] Subtle "Saving..." indicator while in flight
[ ] "Saved ✓" shown briefly after success — not a full toast
[ ] Error: revert the toggle + show an inline error

NEVER:
× Navigate away without prompting if there are unsaved changes
× Show "Saved" if the save actually failed
× Use a toast for settings saves (too transient for trust-sensitive changes)
```

---

# 4. Password Change

```text
[ ] Separate section from general settings — has its own form and save button
[ ] Three fields: "Current Password", "New Password", "Confirm New Password"
[ ] Current password validated on backend before change is accepted
[ ] New password strength indicator (same as signup — see Forms & Validation)
[ ] Confirm password validated against new password on blur
[ ] On success: clear all three fields, show "Password changed." inline confirmation
[ ] On error: show specific inline error, keep fields (except current password — clear it)
[ ] autocomplete="current-password" on current password field
[ ] autocomplete="new-password" on new password and confirm fields
[ ] Password never shown in URL params or logged
```

---

# 5. Avatar & Profile Image

```text
UPLOAD
[ ] Accepted types: JPG, PNG, GIF (or subset) — shown to user
[ ] Max size: 5 MB — validated before upload
[ ] File type validated by MIME type — not just extension
[ ] Preview shown after file is selected (before upload)
[ ] Upload progress shown
[ ] Cropping tool optional — aspect ratio maintained if no cropping
[ ] EXIF metadata stripped from uploaded images

DISPLAY
[ ] Stored in cloud storage — not in the database as a blob
[ ] Public URL stored in the user record
[ ] Avatar displayed wherever the user's name appears (nav, comments, table rows)
[ ] Fallback: initials in a colored circle — same color every time (deterministic by user ID)

REMOVE
[ ] "Remove photo" option available after upload
[ ] Removes the file from storage and clears the URL
[ ] Fallback initials displayed after removal
```

---

# 6. Notification Preferences

```text
[ ] Each notification type is separately toggleable
[ ] Categories: Email Notifications, In-App Notifications, Push (if applicable)

DEFAULTS
[ ] New member joined workspace → ON (default)
[ ] Task assigned to me → ON (default)
[ ] Comment mentioning me → ON (default)
[ ] Weekly digest → ON (default, can be disabled)
[ ] Marketing/product updates → OFF (default, can be enabled)

RULES
[ ] Security emails (login alerts, password changes) CANNOT be disabled
[ ] Explicitly state which emails cannot be disabled: "Required for account security — cannot be disabled"
[ ] Save behavior: auto-save on toggle with "Saving..." indicator
[ ] Email shown that notifications are sent to — link to change email
```

---

# 7. Connected Accounts

```text
[ ] List all OAuth providers the user can connect (Google, GitHub, etc.)
[ ] Connected: show connected account email + "Disconnect" button
[ ] Not connected: "Connect [Provider]" button
[ ] Disconnect: confirm if it is the only login method — "Disconnecting this will require you to set a password to log in."
[ ] Cannot disconnect all login methods — at least one must remain
[ ] OAuth connection: redirect to provider, handle callback, show success/error
```

---

# 8. Danger Zone

```text
[ ] Danger zone section at bottom of page — always last
[ ] Red border around the entire section
[ ] Heading: "Danger Zone"
[ ] Subheading for each action explaining the consequence
[ ] Each action has its own confirmation dialog — never just a button click

LEAVE WORKSPACE
  Title:  "Leave [Workspace Name]?"
  Body:   "You will lose access to all projects and data in this workspace."
  Button: "Leave Workspace" — red

DELETE ACCOUNT
  Title:  "Delete Your Account?"
  Body:   "All your data will be permanently deleted. This cannot be undone."
  Type-to-confirm: "Type DELETE to confirm"
  Button: Disabled until input matches; "Delete My Account" — red

[ ] Both actions require type-to-confirm (high risk — see Error & Feedback standards)
[ ] After account deletion: session cleared, redirect to homepage
[ ] After leaving workspace: redirect to workspace selector or homepage
```

---

# 9. Workspace & Team Settings

```text
GENERAL
[ ] Workspace name: editable, max 100 characters
[ ] Workspace slug/URL: editable, URL-safe characters only, uniqueness validated
[ ] Logo upload: same rules as avatar
[ ] Timezone: affects displayed dates across the workspace

MEMBERS
[ ] List of members with role shown as a badge
[ ] Role change: dropdown, saved immediately with "Saving..." indicator
[ ] Remove member: confirmation dialog — "Remove [Name] from [Workspace]?"
[ ] Invite: email input + role selector + "Send Invite" button

BILLING (if applicable)
[ ] Current plan shown with limits
[ ] Upgrade/downgrade link
[ ] Payment method management (redirect to billing portal)
[ ] Invoice history downloadable

ADMIN-ONLY ACCESS
[ ] Workspace settings page accessible only to admin/owner roles
[ ] Non-admins see a 403 state if they navigate to this URL directly
```

---

## Practice Task

**→ [Settings & Profile Checklist](../../checklists/frontend/settings-profile.md)**
Apply the checklist to a profile settings page covering display name, email change, password change, avatar upload, and notification preferences.
