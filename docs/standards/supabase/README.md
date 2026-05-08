# Supabase Backend Standards

> **Core Rule:** No API is complete until database schema, RLS, validation, error handling, logs, tests, and frontend documentation are all done.

---

## Contents

| File | What It Covers |
|------|---------------|
| [01 — Database Schema & RLS](01-database-and-rls.md) | Developer responsibilities, table schema standards, naming, constraints, indexes, RLS policies, auth standards, service role rules |
| [02 — API Design & Error Codes](02-api-design.md) | API types decision table, core build flow, API design standards, response format, standard error codes, frontend contract |
| [03 — API Documentation](03-api-documentation.md) | API documentation template, frontend code examples, RPC pattern, full worked API example (Add Project) |
| [04 — Edge Functions](04-edge-functions.md) | When to use Edge Functions, required structure, response standard, auth pattern, full Edge Function example |
| [05 — Validation, Multi-Tenant & Storage](05-validation-multitenant-storage.md) | Validation layers, frontend error contract, multi-tenant scoping, workspace membership, role permission matrix, storage RLS |
| [06 — Audit, Webhooks, Migrations & Testing](06-operations-and-testing.md) | Audit logs, edge function logging, webhooks, API keys, versioning, migrations, testing standards, RLS testing matrix, backend handoff checklist |
| [07 — RPC & Postgres Functions](07-rpc-functions.md) | When to use RPC vs direct query vs Edge Function, SECURITY INVOKER vs SECURITY DEFINER, parameter naming (p_ prefix), return type standards, validation order (7 steps), error raising format, transaction handling, auth pattern, naming conventions, performance rules, testing requirements |

---

## Developer Responsibilities

Every Supabase backend developer is accountable for:

| Area | Responsibility |
|------|---------------|
| Database Schema | Clean tables, constraints, indexes, relationships, timestamps. |
| RLS Policies | Row-level access rules for every exposed table. |
| API Layer | Correct API type: direct query, RPC, or Edge Function. |
| Auth Rules | Define who can call what: anonymous, authenticated, owner, admin, org member. |
| Validation | Validate payload on frontend AND backend AND database. |
| Error Handling | Return predictable errors — never silent failure. |
| Documentation | Write frontend-facing API contract for every API. |
| Testing | Test success, validation error, unauthorized, forbidden, duplicate, missing data. |
| Security | No service-role key exposed, no open table without RLS. |
| Observability | Log important actions, failed requests, webhook retries, background job failures. |

---

## Definition of Done

An API is done only when **all** of these are checked:

```text
[ ] Migration created
[ ] Table schema reviewed
[ ] Constraints added
[ ] Indexes added
[ ] RLS enabled
[ ] SELECT policy added
[ ] INSERT policy added
[ ] UPDATE policy added
[ ] DELETE policy added or intentionally blocked
[ ] RPC/Edge Function created if needed
[ ] Auth requirement implemented
[ ] Role checks implemented
[ ] Input validation implemented
[ ] Error responses standardized
[ ] Logs/audit logs added
[ ] Storage policy added if files exist
[ ] Webhook signature/idempotency added if webhook exists
[ ] API documented
[ ] Frontend sample code provided
[ ] Test cases documented
[ ] Tested as owner/admin/member/viewer/non-member/anonymous
[ ] Staging verified
```

---

## The Most Important Backend Rule

For every API, be able to answer clearly:

```text
Who is calling this?
What are they allowed to access?
How does the database enforce that?
What exact payload is allowed?
What exact error is returned when something fails?
What should frontend do in every case?
```

If any answer is unclear — the API is not production-ready.
