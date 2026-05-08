# Frontend Standards — Permissions & Auth UI

> The frontend reflects permissions. The backend enforces them. Never enforce a rule only on the frontend.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-fundamental-rule) | The Fundamental Rule |
| [2](#2-hide-vs-disable) | Hide vs Disable |
| [3](#3-reading-role--permissions) | Reading Role & Permissions |
| [4](#4-ui-element-permission-states) | UI Element Permission States |
| [5](#5-page-level-access) | Page-Level Access |
| [6](#6-partial-access) | Partial Access |
| [7](#7-real-time-role-changes) | Real-Time Role Changes |
| [8](#8-auth-flow-standards) | Auth Flow Standards |

---

# 1. The Fundamental Rule

```text
Frontend → reflects permissions (hides/disables UI elements)
Backend  → enforces permissions (rejects unauthorized requests)

If the backend does not enforce a rule, a technical user can bypass the frontend.
Frontend-only permission checks are a UX feature — not a security feature.
```

---

# 2. Hide vs Disable

| When to Hide | When to Disable |
|-------------|----------------|
| The user can never do this action regardless of context | The user could do this action after meeting a condition |
| The element is irrelevant to the user's role | The element is relevant but currently blocked |
| Showing it would be confusing or noisy | Showing it helps the user understand what's available |

```text
HIDE WHEN:
[ ] The user's role has no access at all (e.g., Viewer cannot see Delete button anywhere)
[ ] The feature is not part of their plan/tier
[ ] Showing would expose information they shouldn't see

DISABLE WITH TOOLTIP WHEN:
[ ] The user is on a free plan and could upgrade
[ ] The user's role is limited but they might need to request access
[ ] The action requires additional conditions to be met (e.g., form is incomplete)
[ ] The button is disabled during loading

TOOLTIP TEXT FOR DISABLED:
[ ] Explains why: "Admin access required to delete projects."
[ ] Does NOT just say "Disabled"
[ ] Does NOT show a tooltip if the reason is obvious
```

---

# 3. Reading Role & Permissions

```text
[ ] Role is always read from the backend auth session — never from URL params or localStorage alone
[ ] Role is never passed in the request body from the frontend
[ ] The role is part of the authenticated user's session (JWT claim or session lookup)
[ ] If the role changes server-side, the frontend reflects it on next session refresh
[ ] Never trust a role stored only in a frontend variable — re-verify with the backend on sensitive actions

SUPABASE PATTERN:
  const { data: { user } } = await supabase.auth.getUser();
  const role = user?.app_metadata?.role; // from JWT, set by backend

REACT PATTERN:
  // Read from auth context — populated on login from backend response
  const { user } = useAuth();
  const isAdmin = user?.role === 'admin';
```

---

# 4. UI Element Permission States

```text
BUTTONS
[ ] Admin-only action buttons: hidden for non-admins (or disabled with tooltip)
[ ] Danger actions (Delete, Archive): hidden for non-owners/admins
[ ] Edit button: visible to editors and above; hidden for viewers

FORM FIELDS
[ ] Read-only for users without edit permission (not disabled — use a read view)
[ ] If showing a form to a viewer: show a read-only view, not a disabled form
[ ] Never show a submit button to a user who cannot submit

NAVIGATION
[ ] Admin sections (Settings, Team, Billing): hidden in nav for non-admins
[ ] Not just a dead link — completely absent from navigation
[ ] 403 page shown if user navigates to admin URL directly

TABLE ROW ACTIONS
[ ] Delete column: hidden entirely for non-admins
[ ] Edit column: hidden for viewers
[ ] Role-appropriate actions only per row (e.g., cannot archive own account)

BULK ACTIONS
[ ] Bulk delete: available only to admin/owner roles
[ ] Bulk export: permission-gated per plan or role
```

---

# 5. Page-Level Access

```text
[ ] Each page checks permissions on mount — not only at nav link level
[ ] If user lacks access: show a 403 state, not a blank page
[ ] 403 state contains: "You don't have permission to view this page." + link back to safe page
[ ] Never redirect silently — always show the user why they can't access something
[ ] Protected routes wrapped in a permission guard component
[ ] Guard checks: authenticated? correct role? correct workspace membership?
[ ] If not authenticated: redirect to login (with return URL)
[ ] If authenticated but wrong role: show 403 state (no redirect)
```

---

# 6. Partial Access

Some pages are visible to multiple roles but with different capability levels.

```text
[ ] Editor sees the page but not admin-only sections
[ ] Admin sections collapsed or hidden — not visible but greyed out
[ ] "View-only" mode for pages where the role can read but not write
[ ] Read-only badge or banner: "You have view-only access to this workspace."
[ ] Actions in read-only mode are hidden or disabled with tooltips
[ ] Table row actions reduced to role-appropriate subset
[ ] Do not show empty action menus (e.g., a "..." menu with nothing inside)
```

---

# 7. Real-Time Role Changes

```text
[ ] If a user's role is changed server-side while they are logged in:
    — Their session reflects the new role on the next page navigation or explicit refresh
    — They are not shown actions they can no longer perform
[ ] If a user is removed from a workspace mid-session:
    — Their next API call returns 403/FORBIDDEN
    — Frontend catches this and redirects to a "You've been removed" state
[ ] Do not show a blank screen when this happens — show a clear message
[ ] Supabase: use realtime subscriptions on workspace_members if instant reflection is needed
```

---

# 8. Auth Flow Standards

```text
LOGIN
[ ] Login form: email + password (or OAuth)
[ ] "Remember me" checkbox (default: session-length only)
[ ] Failed login: "Incorrect email or password." — same message for both (no enumeration)
[ ] Account lockout after N failed attempts: "Too many attempts. Try again in 15 minutes."
[ ] "Forgot password" link visible on login form

SIGNUP
[ ] Email verification sent immediately after signup
[ ] "Check your email" state shown — not a redirect to the app
[ ] User cannot access the app until email is verified (if required)
[ ] Duplicate email: "An account with this email already exists." + "Sign in" link

SESSION
[ ] Session expiry: redirect to login with "Your session has expired." message
[ ] Token refresh handled silently — no user-visible flash
[ ] After login: redirect to the original destination (not always the dashboard)

LOGOUT
[ ] Clears all local session data (tokens, cached user data)
[ ] Redirects to login page
[ ] Never shows the user their previous data after logout
```

---

## Practice Task

**→ [Permissions & Role-Based UI Checklist](../../checklists/frontend/permissions-role-ui.md)**
Review the checklist and apply it to any page that renders different content based on the user's role.
