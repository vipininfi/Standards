# Supabase Practice Tasks

Complete these in order — each task builds on the previous one.

| Task | What You Practice |
|------|------------------|
| [01 — Create the Projects Table](01-create-projects-table.md) | Schema design, required columns (id/created_at/updated_at/workspace_id/created_by), CHECK constraints, indexes, soft delete, updated_at trigger, migration file |
| [02 — Write RLS Policies](02-rls-policies.md) | All four policy types, workspace membership subquery pattern, role-based rules, INSERT WITH CHECK for created_by, UPDATE WITH CHECK to prevent workspace change, testing across all roles |
| [03 — Create Project API (RPC)](03-create-project-api.md) | RPC function with atomic multi-step logic (insert project + task list + audit log), RAISE EXCEPTION with standard error codes, API documentation |
| [04 — Project Invite Edge Function](04-invite-edge-function.md) | Edge Function structure (CORS/auth/validation/logic/response), Deno secrets, JWT validation, third-party API call (Resend email), logging rules |

**Standards to read first:**
- [Supabase Standards Overview](../../standards/supabase/README.md)
- [01 — Database & RLS](../../standards/supabase/01-database-and-rls.md)
- [02 — API Design](../../standards/supabase/02-api-design.md)
- [04 — Edge Functions](../../standards/supabase/04-edge-functions.md)
