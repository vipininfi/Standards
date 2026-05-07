# Edge Function Standards

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Sections 20–23 (when to use Edge Functions, required structure, response standard, auth pattern, full example)

---

# 20. Edge Function Standards

Use Edge Functions for:

| Use Case                | Example                                          |
| ----------------------- | ------------------------------------------------ |
| External APIs           | SendGrid, Stripe, OpenAI, Deepgram.              |
| Webhooks                | Stripe webhook, Smartlead webhook, Zoho webhook. |
| Secret Logic            | API keys, service role operations.               |
| Heavy Validation        | Multi-step business rules.                       |
| Background-like Process | File parse, AI response generation.              |
| Admin Action            | Suspend user, generate report.                   |

## Edge Function Must Include

```text id="e3zu5x"
1. CORS handling
2. Method validation
3. Auth validation
4. Payload parsing
5. Payload validation
6. Permission check
7. Business logic
8. Error mapping
9. Structured logs
10. Safe response
```

---

# 21. Edge Function Response Standard

```ts id="xww9ko"
function jsonResponse(body: unknown, status = 200) {
  return new Response(JSON.stringify(body), {
    status,
    headers: {
      "Content-Type": "application/json",
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
      "Access-Control-Allow-Methods": "POST, OPTIONS"
    }
  });
}
```

---

# 22. Edge Function Auth Pattern

For user-authenticated Edge Functions:

```ts id="ppitug"
const authHeader = req.headers.get("Authorization");

if (!authHeader || !authHeader.startsWith("Bearer ")) {
  return jsonResponse({
    success: false,
    error: {
      code: "UNAUTHENTICATED",
      message: "Missing or invalid authorization token."
    }
  }, 401);
}
```

Then verify user using Supabase Auth:

```ts id="a8ti8e"
const supabase = createClient(
  Deno.env.get("SUPABASE_URL")!,
  Deno.env.get("SUPABASE_ANON_KEY")!,
  {
    global: {
      headers: {
        Authorization: authHeader
      }
    }
  }
);

const { data: userData, error: userError } = await supabase.auth.getUser();

if (userError || !userData.user) {
  return jsonResponse({
    success: false,
    error: {
      code: "UNAUTHENTICATED",
      message: "Invalid or expired session."
    }
  }, 401);
}
```

Supabase’s legacy auth integration guide shows the important pattern: create the Supabase client inside the function using the caller’s `Authorization` header so RLS policies are applied under that user context. ([Supabase][5])

---

# 23. Edge Function Example Documentation

```md id="6hzcaz"
# API: Send Project Invite

## 1. Purpose
Sends an invitation email to a new team member and creates an invitation record.

## 2. API Type
Supabase Edge Function

## 3. Endpoint
POST /functions/v1/send-project-invite

## 4. Auth Required
Yes.

Header:
Authorization: Bearer <access_token>

## 5. Permission Required
User must be workspace owner or admin.

## 6. Request Payload

{
  "workspace_id": "uuid",
  "email": "user@example.com",
  "role": "admin | manager | member | viewer"
}

## 7. Validation Rules
- workspace_id must be valid UUID.
- email must be valid email.
- role must be one of allowed roles.
- user cannot invite owner unless current user is owner.
- duplicate pending invite is not allowed.

## 8. Success Response

{
  "success": true,
  "data": {
    "invitation_id": "uuid",
    "email": "user@example.com",
    "role": "member",
    "expires_at": "2026-05-14T10:00:00Z"
  },
  "message": "Invitation sent successfully."
}

## 9. Error Responses
- 401 UNAUTHENTICATED
- 403 FORBIDDEN
- 409 INVITE_ALREADY_EXISTS
- 422 VALIDATION_ERROR
- 500 INTERNAL_ERROR
- 502 EMAIL_SERVICE_ERROR

## 10. Frontend Notes
- Disable invite button while request is running.
- If 401, redirect to login.
- If 403, hide invite button for unauthorized role.
- If 409, show "This user already has a pending invite."
- If 502, show "Invite created but email could not be sent" only if backend confirms partial success.
```

---

