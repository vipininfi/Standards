# Supabase Checklists

> Use these before marking any Supabase backend task done.

---

| Checklist | Use When |
|-----------|----------|
| [New Table](new-table.md) | Creating any new database table — schema, constraints, indexes, RLS, migration |
| [RLS Policies](rls-policies.md) | Writing or reviewing Row Level Security policies — patterns, role matrix, testing |
| [New API Endpoint](new-api-endpoint.md) | Building any new endpoint — choosing API type, auth, validation, error format, documentation |
| [Edge Function](edge-function.md) | Building an Edge Function — structure, auth, validation, secrets, webhooks, logging |
| [RPC Function](rpc-function.md) | Writing any Postgres RPC function — SECURITY mode, parameter types, validation order, error format, transaction safety, documentation |
| [Storage Bucket](storage-bucket.md) | Creating a storage bucket — visibility, RLS policies, file path structure, signed URLs, upload/delete flow |
| [Database Trigger](database-trigger.md) | Writing a database trigger — BEFORE vs AFTER, trigger function standards, timestamps pattern, audit log pattern |

---

## Quick Reference — Core Supabase Rules

```text
[ ] Every exposed table has RLS enabled
[ ] No table accessible without a matching RLS policy
[ ] Service role key NEVER in frontend code
[ ] Role always read from the database — never from request payload
[ ] workspace_id verified via membership subquery — not just matched against payload
[ ] created_by set in RLS INSERT policy WITH CHECK — not from request body
[ ] Every API endpoint has documentation before frontend handoff
[ ] Every endpoint tested for: success, unauthorized, forbidden, not found, duplicate, validation error
```

---

## Related Standards

- [Supabase Standards](../../standards/supabase/README.md) — Full reference
  - [01 — Database & RLS](../../standards/supabase/01-database-and-rls.md)
  - [02 — API Design](../../standards/supabase/02-api-design.md)
  - [03 — API Documentation](../../standards/supabase/03-api-documentation.md)
  - [04 — Edge Functions](../../standards/supabase/04-edge-functions.md)
  - [05 — Validation, Multi-Tenant & Storage](../../standards/supabase/05-validation-multitenant-storage.md)
  - [06 — Operations & Testing](../../standards/supabase/06-operations-and-testing.md)
  - [07 — RPC & Postgres Functions](../../standards/supabase/07-rpc-functions.md)
