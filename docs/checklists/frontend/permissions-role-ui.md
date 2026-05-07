# Permissions & Role-Based UI Checklist

> **Core Rule:** The backend enforces all permissions. The frontend only adjusts the UI to reflect them. Never rely on hiding a button as a security measure — the backend must block unauthorized actions independently of what the frontend shows or hides.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-fundamental-rule) | The Fundamental Rule |
| [2](#2-hide-vs-disable) | Hide vs Disable |
| [3](#3-reading-the-users-role) | Reading the User's Role |
| [4](#4-permission-states-per-ui-element) | Permission States Per UI Element |
| [5](#5-page-level-access) | Page-Level Access |
| [6](#6-partial-access) | Partial Access |
| [7](#7-role-changes--real-time-permission-updates) | Role Changes & Real-Time Permission Updates |
| [8](#8-permissions-checklist--before-marking-done) | Permissions Checklist — Before Marking Done |

---

# 1. The Fundamental Rule

```text
[ ] Backend validates permissions on every API call — regardless of what the frontend shows
[ ] Hiding a button is a UX improvement only — it is NOT a security measure
[ ] A user who knows the API endpoint should be blocked on the backend even if the button is hidden in the UI
[ ] Frontend never enforces permissions alone — it only reflects them
[ ] Permissions logic lives in one place per platform:
  — Supabase: RLS policies
  — Xano: check_workspace_permission function
  — Django: LoginRequiredMixin + permission checks in views
```

---

# 2. Hide vs Disable

| Approach | When to Use | Rule |
|----------|------------|------|
| **Hide** the element | The user's role has no access at all and seeing the option would confuse them | Acceptable, but backend still enforces |
| **Disable** the element | The user might have access in some contexts, or they need to know the feature exists | Show why it is disabled |
| **Show with explanation** | Upgrading would grant access (paid feature, role upgrade) | Show with upgrade prompt |

```text
[ ] Disabled elements have a tooltip explaining why they are disabled:
  — "You need admin permission to delete projects."
  — "Upgrade to Pro to access this feature."
[ ] Disabled elements are visually distinct from active elements (reduced opacity, different cursor)
[ ] Disabled elements are not completely hidden — the user should understand what exists but is unavailable
[ ] Hidden elements: only when the feature is completely irrelevant to the role (not just restricted)
[ ] Do not hide navigation items that the user can see but not use — disable them with a tooltip instead
```

---

# 3. Reading the User's Role

```text
[ ] User's role read from the authenticated session/token — NOT from a URL param, query string, or local storage setting
[ ] Role read fresh from the backend on each page load — not assumed from a cookie that could be stale
[ ] If using Supabase: role is read from workspace_members table via RLS or a dedicated query
[ ] If using Xano: role is read from workspace_members table in the check_workspace_permission function
[ ] If using Django: role is read from the database in the view or mixin — not from session payload
[ ] Frontend stores the role in auth context/state after loading — not in localStorage
[ ] Role is never accepted from a user-controlled source (form field, URL, manual input)
```

---

# 4. Permission States Per UI Element

### Buttons

```text
[ ] "Add Project" button: visible to members, admin, owner — hidden from viewers
[ ] "Edit" button on a record: visible to the record owner and admins — disabled for others
[ ] "Delete" button: visible only to admins/owners — hidden for members and viewers
[ ] "Invite Member" button: visible only to admins/owners
[ ] "Export" button: visible if user has export permission — disabled with tooltip if not
```

### Form Fields

```text
[ ] Fields the user cannot edit: shown as read-only (disabled input or plain text) — not removed from the page
[ ] Read-only fields have a tooltip: "Only admins can change this setting."
[ ] Entire form shown as read-only if user has view-only access to the entity
[ ] Do not show a "Save" button if the entire form is read-only
```

### Navigation

```text
[ ] Navigation items the user cannot access: hidden OR shown as disabled with a tooltip
  — Prefer hiding for items completely outside the role's access
  — Prefer disabling for items that could be accessible with a role upgrade
[ ] Admin panel / settings link: hidden for non-admin roles
[ ] Billing section: hidden for non-owner roles
```

### Tables / Lists

```text
[ ] Action column (Edit, Delete) adjusted per row based on the user's permission for that specific row
  — A member can edit their own rows but not others' rows
  — An admin can edit all rows
[ ] Do not show an Edit button on a row the user cannot edit — disable it with a tooltip
[ ] Bulk action "Delete selected" hidden or disabled if user does not have delete permission
```

---

# 5. Page-Level Access

```text
[ ] If the user navigates to a page they do not have access to:
  — Show a clear "You don't have permission to view this." message in the page body
  — Do NOT redirect to 404 (resource exists — the user just doesn't have access)
  — Do NOT show a blank page
  — Provide a link back to a page they do have access to (e.g., "Back to dashboard")
[ ] If the user's role does not have a feature at all (e.g., analytics for free tier):
  — Show a locked/upgrade state — not a 404
  — Explain what they would get with an upgrade
[ ] Protected routes redirect unauthenticated users to login — not to 404
[ ] After redirect to login: restore the intended URL so the user lands on the right page after auth
```

---

# 6. Partial Access

When a user has access to some but not all parts of a page:

```text
[ ] Show the parts they have access to fully
[ ] Show the parts they don't have access to as locked/blurred/disabled with a clear explanation
[ ] Example: a viewer on a project settings page sees general info but the "Danger Zone" section is hidden or shows "Admin only"
[ ] Example: a member on billing sees the current plan but the "Manage billing" button is disabled with "Owner access required"
[ ] Partial access is not a security measure — backend restricts the data and actions, frontend just displays the state
```

---

# 7. Role Changes & Real-Time Permission Updates

```text
[ ] If a user's role changes while they are logged in:
  — Their current session reflects the old role until they reload or the token refreshes
  — When they next make an API call: the backend enforces the new role
  — If the call is rejected due to role change: show "You no longer have permission to do this." — not a generic error
[ ] If a user is removed from a workspace while viewing a workspace page:
  — Next API call to the workspace will return 403
  — Frontend catches this and redirects to a "You no longer have access to this workspace." page
[ ] Token refresh should re-read the current role from the database
[ ] Do not cache the user's role indefinitely — refresh from backend on login and on token refresh
```

---

# 8. Permissions Checklist — Before Marking Done

```text
FUNDAMENTAL
[ ] Backend enforces permissions independently of frontend
[ ] Frontend reflects permissions — it does not enforce them
[ ] Role read from auth session / database — not from URL or local storage

HIDE VS DISABLE
[ ] Elements the user can never access in their role: hidden
[ ] Elements the user could access with role upgrade or in other contexts: disabled + tooltip
[ ] Disabled elements have tooltip explaining why ("Admin access required")
[ ] Disabled elements are visually distinct

BUTTONS
[ ] Add/Create: matches who can create
[ ] Edit: matches who can edit (owner + admin vs member vs viewer)
[ ] Delete: matches who can delete (admin/owner only)
[ ] Invite/Manage: admin/owner only

FORM FIELDS
[ ] Read-only fields shown as disabled (not removed)
[ ] Read-only fields have tooltip
[ ] No Save button if entire form is read-only

TABLES
[ ] Row actions adjusted per row, per user's permission on that row
[ ] Bulk actions hidden/disabled if user cannot perform them

PAGE ACCESS
[ ] Unauthorized page: "You don't have permission." message (not blank, not 404)
[ ] Unauthenticated: redirect to login with return URL
[ ] Locked features: upgrade prompt (not 404)

ROLE CHANGES
[ ] 403 from role change → clear message (not generic error)
[ ] Removed from workspace → redirect with explanation
```
