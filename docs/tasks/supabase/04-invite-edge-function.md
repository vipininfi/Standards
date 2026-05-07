# Task: Build a Project Invite Edge Function

**Platform:** Supabase Edge Functions
**Covers:** [Edge Function Checklist](../../checklists/supabase/edge-function.md) · [Edge Function Standards](../../standards/supabase/04-edge-functions.md)

---

## Scenario

Workspace admins and owners can invite users to a project by email. When an invite is sent, the system must:

1. Validate the caller is an admin or owner of the workspace
2. Check if the invitee is already a project member
3. Store the invite record in the database
4. Send an invitation email via [Resend](https://resend.com) (a third-party email service)

Because this requires a secret API key (Resend), it must be an **Edge Function** — not an RPC or direct table query.

---

## What to Build

A Supabase Edge Function named `send-project-invite`.

---

## Function Specification

### Endpoint

```
POST /functions/v1/send-project-invite
Authorization: Bearer <user_jwt>
Content-Type: application/json
```

### Request Body

```json
{
  "project_id": "uuid",
  "invitee_email": "user@example.com",
  "role": "member"
}
```

### Required Logic (in this order)

1. **Handle CORS preflight** (`OPTIONS` request → return 200 immediately)
2. **Validate auth token** — extract and verify the user's JWT; return 401 if missing or invalid
3. **Parse request body** — return 400 if JSON is invalid
4. **Validate inputs:**
   - `project_id` — required, valid UUID format
   - `invitee_email` — required, valid email format
   - `role` — required, must be `"member"` or `"viewer"` (cannot invite as admin)
5. **Load project** — verify the project exists and get its `workspace_id`; return 404 if not found
6. **Check caller permission** — caller must be admin or owner of the workspace; return 403 if not
7. **Check duplicate** — if the invitee is already a project member, return 409 DUPLICATE
8. **Create invite record** — insert into `project_invites` table (see schema below)
9. **Send email via Resend** — send the invite email with the invite link
10. **Return success response**

### project_invites Table (assume it exists)

```sql
project_invites (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id   uuid NOT NULL REFERENCES projects(id),
  workspace_id uuid NOT NULL,
  invited_by   uuid NOT NULL REFERENCES auth.users(id),
  email        text NOT NULL,
  role         text NOT NULL CHECK (role IN ('member', 'viewer')),
  token        text NOT NULL UNIQUE,  -- secure random token for the invite link
  expires_at   timestamptz NOT NULL DEFAULT (now() + interval '7 days'),
  accepted_at  timestamptz,
  created_at   timestamptz NOT NULL DEFAULT now()
)
```

### Email Content

The invitation email should include:
- Who invited them and to which project
- An invite link: `https://app.workflow.com/invites/accept?token=<token>`
- Token expiry notice: "This invite expires in 7 days."

### Success Response

```json
{
  "data": {
    "invite_id": "uuid",
    "invitee_email": "user@example.com",
    "expires_at": "2024-01-22T10:00:00Z"
  },
  "message": "Invitation sent successfully."
}
```

### Error Responses

| Scenario | Code | HTTP Status |
|----------|------|-------------|
| Missing or invalid token | `AUTH_REQUIRED` | 401 |
| Invalid JSON body | `VALIDATION_ERROR` | 400 |
| Invalid project_id format | `VALIDATION_ERROR` | 400 |
| Invalid email format | `VALIDATION_ERROR` | 400 |
| Invalid role value | `VALIDATION_ERROR` | 400 |
| Project not found | `NOT_FOUND` | 404 |
| Caller not admin/owner | `FORBIDDEN` | 403 |
| Invitee already a member | `DUPLICATE` | 409 |
| Resend API failure | `SERVER_ERROR` | 500 |

---

## Environment Variables Required

| Variable | What It Is |
|----------|-----------|
| `SUPABASE_URL` | Your Supabase project URL |
| `SUPABASE_ANON_KEY` | Supabase anon key (for user-scoped client) |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role key (for inserting the invite record — bypasses RLS) |
| `RESEND_API_KEY` | Resend API key for sending emails |
| `APP_BASE_URL` | Base URL of the app (for building the invite link) |

> Service role key is used only inside the Edge Function (server-side) to insert the invite record. It is never sent to the frontend.

---

## Logging Requirements

Log these events (include the function name prefix `[send-project-invite]`):

- Invite created: `console.log('[send-project-invite] Invite created: invite_id=xxx, project_id=xxx')`
- Email sent: `console.log('[send-project-invite] Email sent to: masked@email or user ID')`
- Validation failure: `console.warn('[send-project-invite] Validation failed: ...')`
- Resend failure: `console.error('[send-project-invite] Resend error: status=xxx, message=xxx')`

Do NOT log:
- The full email address in production (use partial masking or just log the invite ID)
- The invite token
- The Resend API key

---

## What You Should NOT Do

- Do not hardcode the Resend API key in the function code — use `Deno.env.get('RESEND_API_KEY')`
- Do not use the service role key for the auth check — use the user's JWT so RLS applies where it should
- Do not skip the permission check — any authenticated user could call this endpoint without it
- Do not send the email before inserting the invite record — if the DB insert fails, no email should go out
- Do not expose internal error details (stack traces, DB errors) in the response
- Do not return 200 for every response — use correct HTTP status codes

---

## Checklist to Run When Done

Use the [Edge Function Checklist](../../checklists/supabase/edge-function.md#9-edge-function-checklist--before-marking-done).

---

## Done When

```text
STRUCTURE
[ ] CORS OPTIONS handler at top
[ ] Try/catch wraps all logic
[ ] All responses include CORS headers and Content-Type: application/json

AUTH
[ ] Authorization header extracted and validated
[ ] Missing/invalid token returns 401 immediately
[ ] Caller's user ID from validated JWT — not from request body

INPUT VALIDATION
[ ] req.json() in try/catch — invalid JSON returns 400
[ ] project_id: required, valid UUID format
[ ] invitee_email: required, valid email format
[ ] role: required, must be 'member' or 'viewer'
[ ] Field-level errors returned for validation failures

BUSINESS LOGIC
[ ] Project existence checked → 404 if not found
[ ] Caller permission checked (admin or owner) → 403 if not
[ ] Duplicate invite/membership checked → 409 if already member
[ ] invite_token generated securely (crypto.randomUUID() or similar)
[ ] project_invites record inserted using service role client
[ ] Email sent via Resend AFTER successful DB insert

SECRETS
[ ] RESEND_API_KEY via Deno.env.get()
[ ] SUPABASE_SERVICE_ROLE_KEY via Deno.env.get()
[ ] No secrets hardcoded
[ ] Secrets added to Supabase Dashboard → Edge Functions → Secrets

RESPONSE FORMAT
[ ] Success: { data: { invite_id, invitee_email, expires_at }, message } — 200
[ ] Each error case returns correct HTTP status + { error, message }
[ ] No stack traces in response

LOGGING
[ ] Function name prefix in all logs
[ ] Invite created and email sent logged
[ ] Errors logged with context
[ ] No sensitive data (token, API key, full email) in logs

TESTING
[ ] Success case: invite created + email sent
[ ] Missing auth token → 401
[ ] Invalid email → 400 with field error
[ ] Invalid role → 400 with field error
[ ] Non-existent project → 404
[ ] Member calling (not admin) → 403
[ ] Already a member → 409
[ ] Resend failure handled gracefully → 500
```
