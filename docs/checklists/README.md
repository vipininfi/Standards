# Checklists

> Run the relevant checklist before marking any task done. These are gates — not optional.

---

## General Checklists

These apply across all platforms and roles.

| Checklist | Who Uses It |
|-----------|------------|
| [Backend Checklist](backend-checklist.md) | Backend developer — before marking any API or feature complete |
| [Frontend Checklist](frontend-checklist.md) | Frontend developer — before submitting any task |
| [Website Launch Checklist](website-launch-checklist.md) | Website builder — before going live |
| [Task Completion Checklist](task-completion-checklist.md) | All roles — universal task completion gate |

---

## Frontend → [frontend/](frontend/)

Detailed checklists for specific frontend tasks and UI patterns.

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](frontend/form-field-validation.md) | Building any form — covers every input type: text, email, phone (E.164), location/address (Google Places + lat/lng), date, time, datetime, file/image upload (MIME type), OTP, autocomplete, percentage, color picker, and more |
| [Login & Signup Standards](frontend/login-signup.md) | Building login, signup, forgot password, reset password, email verification, or OAuth/social login flows |
| [Add / Edit Consistency](frontend/add-edit-consistency.md) | Building Add or Edit forms for the same entity — shared component requirement, identical validation, UI matching, pre-fill behavior |

---

## Supabase → [supabase/](supabase/)

Step-by-step checklists for every Supabase backend task.

| Checklist | Use When |
|-----------|----------|
| [New Table](supabase/new-table.md) | Creating any new database table — schema, required columns, naming conventions, constraints, indexes, RLS setup, migration file, documentation |
| [RLS Policies](supabase/rls-policies.md) | Writing or reviewing Row Level Security — policy patterns by access type, role permission matrix, anti-patterns, full testing guide per role |
| [New API Endpoint](supabase/new-api-endpoint.md) | Building any new endpoint (direct query / RPC / Edge Function) — auth, input validation, response format, error codes, testing requirements |
| [Edge Function](supabase/edge-function.md) | Building a Supabase Edge Function — required structure, CORS, auth pattern, secrets management, error handling, webhook signature verification |

---

## Xano → [xano/](xano/)

| Checklist | Use When |
|-----------|----------|
| [New API Endpoint](xano/new-endpoint.md) | Building any Xano endpoint — auth token as first step, DB-based permission check, input validation order, standard error format, reusable function extraction |

---

## Django → [django/](django/)

| Checklist | Use When |
|-----------|----------|
| [New View](django/new-view.md) | Building any Django view — auth decorators, form/serializer validation, services.py pattern, response handling, error mapping, testing |
| [Template](django/template.md) | Building any Django HTML template — 3-folder structure, base.html inheritance, zero inline styles/scripts, data-* attribute pattern, CSRF helper |

---

## Website → [website/](website/)

| Checklist | Use When |
|-----------|----------|
| [Website Launch Checklist](website-launch-checklist.md) | Pre-launch gate — design, responsive testing, forms, SEO, migration redirects, analytics, performance, accessibility, go-live, client handoff |
