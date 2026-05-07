# Backend-First Logic Standard

Core rule:

> The backend is the only source of truth. Frontend is for display and interaction only. Any rule, check, calculation, or permission that matters — must live in the backend. Frontend validation is a UX courtesy, not a security gate.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-fundamental-principle) | The Fundamental Principle |
| [2](#2-what-belongs-on-the-backend-always) | What Belongs on the Backend |
| [3](#3-what-frontend-is-allowed-to-do-display--ux-only) | What Frontend Is Allowed to Do |
| [4](#4-anti-patterns--never-do-these) | Anti-Patterns — Never Do These |
| [5](#5-platform-specific-rules) | Platform-Specific Rules |
| [6](#6-the-backend-logic-decision-test) | The Decision Test |
| [7](#7-where-logic-lives-by-platform) | Where Logic Lives By Platform |
| [8](#8-summary-rules) | Summary Rules |

---

# 1. The Fundamental Principle

```text
If it matters → it must be enforced on the backend.
Frontend validation → improves UX.
Backend validation → enforces correctness and security.
Both must exist. Backend is never optional.
```

A frontend developer can bypass browser validation in seconds. A user can open DevTools, modify a request, or call your API directly. If the backend does not enforce the rule — the rule does not exist.

---

# 2. What Belongs on the Backend (Always)

These must never be frontend-only:

| Logic | Platform | Why |
|-------|----------|-----|
| Authentication verification | Supabase / Xano / Django | Anyone can fake a logged-in state on frontend |
| Role and permission checks | Supabase RLS / Xano / Django | Roles can be faked or manipulated on frontend |
| Input validation (required, format, length) | Supabase / Xano / Django | Frontend validation can be bypassed |
| Business rules (status transitions, eligibility, limits) | Supabase / Xano / Django | These are invariants — frontend cannot guarantee them |
| Price and discount calculations | Xano / Django | Sending price from frontend means users can manipulate it |
| Ownership verification ("can this user access this record?") | Supabase RLS / Xano / Django | Frontend only knows what it's told |
| Quota and rate limit enforcement | Xano / Django | Never trust frontend counts |
| Soft delete / hard delete rules | Supabase / Xano / Django | Frontend can hide a button — it cannot stop a direct API call |
| File type and size validation | Supabase / Xano / Django | Frontend file checks are easily bypassed |
| Sensitive data masking (API keys, tokens, PII) | Supabase / Xano / Django | Never compute on frontend and send masked output |
| Audit log creation | Supabase / Xano / Django | Frontend can skip calls — backend triggers cannot |

---

# 3. What Frontend Is Allowed to Do (Display + UX Only)

| Frontend Responsibility | Allowed? | Notes |
|------------------------|----------|-------|
| Show/hide buttons based on known user role | Yes | Still enforced on backend |
| Disable form fields based on user state | Yes | Still validated on backend |
| Client-side form validation before API call | Yes | Reduces unnecessary API calls |
| Display loading/error/success states | Yes | Core frontend responsibility |
| Format dates, numbers, currencies for display | Yes | Display-only, no backend impact |
| Cache data locally for performance | Yes | Always re-validate on backend when mutation happens |
| Compute totals for display only (shopping cart preview) | Yes | Backend must recompute on submit |
| Show permission-based UI (hide invite button for viewers) | Yes | Backend still enforces on the API call |

---

# 4. Anti-Patterns — Never Do These

## Sending Role From Frontend and Trusting It

Bad:

```json
POST /projects
{
  "name": "New Project",
  "user_role": "admin"
}
```

The backend then uses `user_role` from the request to decide what the user can do.

**Any user can send `"user_role": "admin"` and bypass all restrictions.**

Good:

```text
Backend looks up the user's role from the database using auth.uid() / auth token.
Role is never accepted from the frontend payload.
```

---

## Doing Permission Checks Only in Frontend

Bad:

```ts
// Frontend hides the button — that's all
if (user.role !== 'admin') {
  return null; // hide button
}
```

The API has no role check. Anyone who calls the API directly becomes an admin.

Good:

```text
Frontend: Hide the button for non-admins (UX).
Backend: Check role on every API call, return 403 if not allowed (enforcement).
Both must exist.
```

---

## Sending Computed Price From Frontend

Bad:

```json
POST /checkout
{
  "items": [...],
  "total": 150.00,
  "discount": 20.00
}
```

The backend trusts the total and processes payment for 150.

**Any user can send `"total": 1.00` and get the product for 1 rupee.**

Good:

```text
Backend receives the item list and quantities.
Backend computes the price from its own database/pricing rules.
Backend charges the computed amount — never the frontend-sent amount.
```

---

## Filtering Sensitive Data on Frontend

Bad:

```ts
// Backend returns all user data, frontend filters out private fields
const { password_hash, secret_token, ...safeUser } = user;
```

The browser received the sensitive fields. They appear in network requests.

Good:

```text
Backend only returns fields the user is allowed to see.
Sensitive fields are never included in the API response.
Frontend receives only what it needs.
```

---

## Skipping Backend Validation Because Frontend Already Validates

Bad:

```python
# Django view
def create_project(request):
    # "Frontend validates this, so we trust it"
    name = request.data.get('name')
    Project.objects.create(name=name, workspace=workspace)
```

Good:

```python
def create_project(request):
    name = request.data.get('name', '').strip()
    if not name or len(name) < 2:
        return JsonResponse({'code': 'VALIDATION_ERROR', 'message': 'Name is required.'}, status=422)
    if len(name) > 100:
        return JsonResponse({'code': 'VALIDATION_ERROR', 'message': 'Name too long.'}, status=422)
    Project.objects.create(name=name, workspace=workspace)
```

---

## Letting Frontend Control Admin Actions via URL Parameters

Bad:

```text
GET /admin/users?role=admin
```

Backend promotes users to admin based on the URL parameter.

Good:

```text
Backend checks if the calling user is an admin.
Backend performs the action using its own logic.
URL parameters carry only resource identifiers, not permissions.
```

---

# 5. Platform-Specific Rules

## Supabase

```text
[ ] RLS is enabled on every table in the public schema.
[ ] RLS policies use auth.uid() — not user-submitted IDs.
[ ] No policy uses "using (true)" without documented reason.
[ ] Authorization data stored in raw_app_meta_data, not raw_user_meta_data.
[ ] Service role key never appears in any frontend code, environment variable, or public repo.
[ ] Frontend calls Supabase with the user's access token — not the service role key.
[ ] Frontend does not control what workspace_id is trusted — RLS verifies membership.
```

## Xano

```text
[ ] Auth token validation is the first step in every protected endpoint.
[ ] User role is always fetched from the Xano database — never from the request payload.
[ ] All inputs are validated in Xano using Preconditions — not just in the frontend.
[ ] Prices, totals, discounts are computed in Xano — not sent from frontend.
[ ] Webhook endpoints verify signatures before processing.
[ ] Environment variables hold all API keys — never hardcoded.
```

## Django

```text
[ ] Every view that requires login uses @login_required or LoginRequiredMixin.
[ ] Role checks happen in the view or a permission class — never rely only on frontend hiding.
[ ] Forms and serializers validate all inputs on the backend.
[ ] Sensitive fields (passwords, tokens, keys) are never included in API responses.
[ ] Users cannot access other users' data by changing an ID in the URL.
[ ] All database queries are scoped to the authenticated user's workspace/organization.
```

## WeWeb / React (Frontend)

```text
[ ] Frontend never stores the Supabase service role key or any backend secret.
[ ] Frontend never sends a role, permission, or price that the backend should compute.
[ ] Frontend shows/hides UI based on role — but the API enforces it independently.
[ ] All user inputs are validated on API call — not just on UI.
[ ] Frontend never trusts data from localStorage/cookies as authoritative for permissions.
```

---

# 6. The Backend Logic Decision Test

Before writing any logic, ask:

> **"If a technically capable user bypassed the frontend and called this API directly — would they be able to do something they should not be able to do?"**

If the answer is yes — the backend is missing a check.

Run this test for every:
- Permission rule
- Validation rule
- Calculation
- Status transition
- Data access

---

# 7. Where Logic Lives By Platform

| Logic Type | Supabase | Xano | Django | Frontend |
|------------|----------|------|--------|----------|
| Who can read this record | RLS SELECT policy | Auth + Permission check | View + QuerySet filter | Display only |
| Who can create this record | RLS INSERT policy | Auth + Permission check + Validation | Form + Permission class | UI gate only |
| Who can edit this record | RLS UPDATE policy | Auth + Permission check | View + Permission class | UI gate only |
| Who can delete this record | RLS DELETE policy | Auth + Permission check | View + Permission class | Confirm dialog only |
| Input validation | DB constraints + RPC | Preconditions | Form / Serializer | UX hint only |
| Price calculation | RPC function | Custom function | Service function | Display preview only |
| Business rule enforcement | RLS + RPC | Xano function | Service / Model method | Not allowed |
| Audit logging | DB trigger / RPC | Post-operation step | Django signal / middleware | Not allowed |
| File validation | Storage policy | Preconditions | View / Form | UX hint only |

---

# 8. Summary Rules

```text
1. Backend is the source of truth — always.
2. Frontend validation is UX — not security.
3. Role is read from the database — never from the request.
4. Price is computed on the backend — never sent from frontend.
5. Ownership is verified by the backend using the authenticated user's ID.
6. Service keys and secrets never touch frontend code.
7. Hiding a button is not the same as enforcing a permission.
8. Frontend can bypass any UI check — backend cannot be bypassed.
9. If it matters, enforce it in two places: backend (required) and frontend (UX).
10. When in doubt: the backend decides, the frontend displays.
```
