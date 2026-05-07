# Audit Logs, Webhooks, Migrations & Testing

> **Part of:** [Supabase Standards](<README.md>)

**Covers:** Sections 32–33 (audit logs, edge function logging), Section 34 (webhooks), Sections 35–37 (API keys, versioning, migrations), Sections 38–40 (testing standards, RLS testing matrix, backend handoff checklist)

---

# 32. Audit Log Standards

Create audit logs for sensitive actions.

## Log These Actions

| Action                             | Required? |
| ---------------------------------- | --------- |
| User invited                       | Yes       |
| User removed                       | Yes       |
| Role changed                       | Yes       |
| Project deleted                    | Yes       |
| Billing changed                    | Yes       |
| API key created/revoked            | Yes       |
| Integration connected/disconnected | Yes       |
| Admin impersonation                | Yes       |
| Data export                        | Yes       |
| Failed login/security event        | Yes       |

## Audit Table

```sql id="5v9vix"
create table public.audit_logs (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid references public.workspaces(id),
  actor_id uuid references auth.users(id),
  action text not null,
  entity_type text not null,
  entity_id uuid,
  old_value jsonb,
  new_value jsonb,
  ip_address text,
  user_agent text,
  created_at timestamptz not null default now()
);
```

---

# 33. Logging Standards for Edge Functions

Every Edge Function should log:

```text id="svi0z6"
request_id
function_name
user_id
workspace_id
action
status
duration_ms
error_code
```

Do not log:

```text id="v78xw5"
passwords
access tokens
refresh tokens
API keys
payment card data
full private documents
sensitive PII unless required
```

---

# 34. Webhook Standards

Use Edge Functions for webhooks from Stripe, Smartlead, Zoho, SendGrid, etc.

## P0 Webhook Requirements

| Requirement            | Standard                                          |
| ---------------------- | ------------------------------------------------- |
| Signature Verification | Mandatory if provider supports it.                |
| Idempotency            | Store event ID, do not process twice.             |
| Raw Body               | Required for Stripe-style signature verification. |
| Retry Safe             | Duplicate webhook should not duplicate records.   |
| Logs                   | Store event status.                               |
| Error Handling         | Return proper status code.                        |

Database Webhooks in Supabase can fire on `INSERT`, `UPDATE`, and `DELETE` after row changes, and they use `pg_net` asynchronously so external calls do not block the database operation for long-running network requests. ([Supabase][6])

---

# 35. API Key Management Standards

If platform gives API keys to users:

| Feature        | Standard                         |
| -------------- | -------------------------------- |
| Create API Key | Show key only once.              |
| Store Key      | Store hashed key, not plaintext. |
| Revoke Key     | Immediate disable.               |
| Rotate Key     | Create new, expire old.          |
| Scope Key      | Read/write/admin scopes.         |
| Rate Limit     | Per API key.                     |
| Audit          | Log creation/revocation.         |

---

# 36. API Versioning Standards

For Edge Functions:

```text id="jco9c9"
/functions/v1/create-project
/functions/v1/send-invite
/functions/v1/stripe-webhook
```

For internal API docs:

```text id="8yko38"
API Version: v1
Status: Draft / Ready / Deprecated
Last Updated: 2026-05-07
Owner: Backend Developer Name
```

Do not silently change API payloads after frontend starts.

---

# 37. Supabase Migration Standards

## P0

| Rule                                     | Standard                    |
| ---------------------------------------- | --------------------------- |
| No dashboard-only production changes     | Use migrations.             |
| Every table/policy/function in migration | Required.                   |
| Migration reviewed before deploy         | Required.                   |
| Staging before production                | Required.                   |
| Rollback plan                            | Required for risky changes. |

Migration should include:

```text id="mq4n6k"
tables
constraints
indexes
RLS enablement
RLS policies
functions
triggers
seed data if needed
```

---

# 38. Testing Standards

## Every API Must Test

| Test Case                   | Required      |
| --------------------------- | ------------- |
| Authenticated success       | Yes           |
| Anonymous request           | Yes           |
| Expired token               | Yes           |
| Wrong role                  | Yes           |
| Wrong workspace             | Yes           |
| Missing required field      | Yes           |
| Invalid field type          | Yes           |
| Duplicate record            | Yes           |
| Related record missing      | Yes           |
| Database constraint failure | Yes           |
| RLS denial                  | Yes           |
| External service failure    | If applicable |
| Retry/idempotency           | If applicable |

---

# 39. RLS Testing Matrix

For every table:

| User Type  | SELECT |  INSERT |  UPDATE |  DELETE |
| ---------- | -----: | ------: | ------: | ------: |
| Anonymous  |     No |      No |      No |      No |
| Owner      |    Yes |     Yes |     Yes |     Yes |
| Admin      |    Yes |     Yes |     Yes | Depends |
| Manager    |    Yes |     Yes |     Yes |      No |
| Member     |    Yes | Depends | Depends |      No |
| Viewer     |    Yes |      No |      No |      No |
| Non-member |     No |      No |      No |      No |

---

# 40. Backend Handoff Checklist for Frontend

Before frontend starts using an API, backend must provide:

```text id="oqzjmi"
1. API name
2. API status: ready/testing/deprecated
3. Auth requirement
4. Role requirement
5. Endpoint/table/RPC/function
6. Request payload
7. Required fields
8. Optional fields
9. Validation rules
10. Success response
11. Error response examples
12. Loading behavior expected
13. Empty state behavior
14. Permission-denied behavior
15. RLS notes
16. Test credentials/users if needed
17. Sample frontend code
18. Known limitations
```

---

