# Practice Tasks

> Each task gives you a realistic scenario and specific requirements to build. Complete the task, then run the linked checklist to verify your work meets the team's standards.

---

## How to Use These Tasks

1. Pick the task that matches what you are learning or building
2. Read the **Scenario** — understand the context before touching any code
3. Read the **What to Build** and **Requirements** sections fully before starting
4. Follow the **What You Should NOT Do** — these are the most common mistakes
5. When done, run the **Checklist** linked at the bottom of the task
6. A task is not complete until every item in the "Done When" section is checked

---

## Frontend Tasks → [frontend/](frontend/)

| Task | What You Practice |
|------|------------------|
| [01 — Registration Form](frontend/01-registration-form.md) | Form field validation: email, password strength, confirm password, phone (E.164), terms checkbox, loading states, error messages |
| [02 — Login Form with Forgot & Reset Password](frontend/02-login-form.md) | Full auth flow: login, forgot password (email enumeration prevention), reset password (token validation on load), session states |
| [03 — Shared Add / Edit Project Form](frontend/03-add-edit-project-form.md) | Add/Edit consistency: one shared component, one validation schema, pre-fill behavior, identical states and error messages in both modes |

---

## Supabase Tasks → [supabase/](supabase/)

These tasks build on each other in sequence — complete them in order.

| Task | What You Practice |
|------|------------------|
| [01 — Create the Projects Table](supabase/01-create-projects-table.md) | Table schema, required columns, constraints, CHECK constraints, indexes, soft delete, `updated_at` trigger, migration file |
| [02 — Write RLS Policies](supabase/02-rls-policies.md) | All four policy types (SELECT/INSERT/UPDATE/DELETE), workspace membership subquery, role-based access, WITH CHECK, testing across all roles |
| [03 — Create Project API (RPC)](supabase/03-create-project-api.md) | RPC function design, atomic multi-step operations, validation in SQL, standard error codes via RAISE EXCEPTION, API documentation |
| [04 — Project Invite Edge Function](supabase/04-invite-edge-function.md) | Edge Function structure, CORS, JWT auth, secrets management, third-party email (Resend), webhook-safe logging, all error cases |
| [05 — Archive Project RPC Function](supabase/05-rpc-function-task.md) | Advanced RPC: SECURITY INVOKER, 7-step validation order, multi-table atomic update, BUSINESS_RULE_VIOLATION check, EXCEPTION WHEN OTHERS |
| [06 — Storage Bucket: Project Attachments](supabase/06-storage-setup-task.md) | Storage bucket setup, RLS on storage.objects, file path structure, signed URLs (not public URLs), upload/download/delete flow |

---

## Xano Tasks → [xano/](xano/)

| Task | What You Practice |
|------|------------------|
| [01 — Create Project Endpoint](xano/01-create-project-endpoint.md) | Auth-first pattern, reusable `check_workspace_permission` function, three-layer validation, standard error format, role check from DB |
| [02 — Reusable Function: check_workspace_permission](xano/02-reusable-function.md) | Extracting shared logic, role hierarchy (not just equality), single responsibility, error raising, updating all calling endpoints |

---

## Django Tasks → [django/](django/)

These tasks build on each other — complete in order.

| Task | What You Practice |
|------|------------------|
| [01 — Project List & Create View](django/01-project-list-create-view.md) | LoginRequiredMixin, WorkspaceMemberMixin, form validation, services.py separation, POST→redirect→GET pattern, PermissionDenied |
| [02 — Project Detail Template](django/02-project-template.md) | 3-folder structure, base.html inheritance, zero inline styles/scripts, data-* attributes for JS, responsive CSS with variables |
| [03 — Project Model + Service](django/03-model-and-service.md) | Model field types, constraints (UniqueConstraint, CheckConstraint), soft delete with custom manager, services.py with business logic |
| [04 — DRF API Views: Projects](django/04-drf-api-view.md) | DRF view types, custom permission classes (database-verified), serializer wiring, service delegation, correct HTTP status codes, pagination |

---

## Website Tasks → [website/](website/)

| Task | What You Practice |
|------|------------------|
| [01 — Replicate a Landing Page](website/01-landing-page-section.md) | Pixel-perfect replication, design token extraction, reusable components, responsive layout (3 breakpoints), semantic HTML, performance |
