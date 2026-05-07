# Documentation Index

This folder contains everything the team needs to build correctly — standards to read before building, checklists to run before marking done, templates to copy into tickets and pull requests, and practice tasks to apply what you've learned.

---

## Folders

| Folder | What It Contains |
|--------|-----------------|
| [standards/](#standards) | How to build — rules, patterns, and decisions for every platform and role |
| [checklists/](#checklists) | What to check — run these before marking any task done |
| [templates/](#templates) | What to copy — ready-made formats for tickets, PRs, and documentation |
| [tasks/](#tasks) | What to build — realistic practice tasks to apply each standard hands-on |

---

## Standards

> Read the relevant standard **before** you start building. These define how the team builds — not suggestions, but rules.

### Cross-Platform Standards

| Document | What It Covers |
|----------|---------------|
| [Backend-First Logic](standards/backend-first-logic.md) | What belongs on the backend vs frontend. Auth, roles, validation, prices, business rules — always enforced on the backend. Frontend is display only. |
| [Code Reusability Standards](standards/code-reusability-standards.md) | Before you build anything — check if it exists. When to extract components, hooks, services, and functions. Platform-specific guides for React, WeWeb, Xano, Django, Supabase. |
| [Team Ownership Standards](standards/team-ownership.md) | How to work — update format, issue raising, ticket discipline, PR standards, communication rules, accountability. |
| [QA & Delivery Standards](standards/qa-delivery-standards.md) | Self-testing expectations, definition of done per role, QA handoff checklist, test case requirements. |

### Platform Standards

| Document | What It Covers |
|----------|---------------|
| [Supabase Standards →](standards/supabase/README.md) | Full Supabase reference — split into 6 focused files: |
| &nbsp;&nbsp;[01 — Database & RLS](standards/supabase/01-database-and-rls.md) | Table schema, naming, constraints, indexes, RLS policies, auth standards, service role rules |
| &nbsp;&nbsp;[02 — API Design](standards/supabase/02-api-design.md) | API type selection (direct / RPC / Edge), response format, standard error codes |
| &nbsp;&nbsp;[03 — API Documentation](standards/supabase/03-api-documentation.md) | Documentation template, frontend call examples, RPC pattern, full worked example |
| &nbsp;&nbsp;[04 — Edge Functions](standards/supabase/04-edge-functions.md) | When to use, required structure, auth pattern, logging, full example |
| &nbsp;&nbsp;[05 — Validation, Multi-Tenant & Storage](standards/supabase/05-validation-multitenant-storage.md) | Validation layers, multi-tenant scoping, role permission matrix, storage security |
| &nbsp;&nbsp;[06 — Operations & Testing](standards/supabase/06-operations-and-testing.md) | Audit logs, webhooks, migrations, testing standards, RLS test matrix, backend handoff checklist |
| [Xano Standards](standards/xano-standards.md) | Auth-first pattern, permission checks, input validation, error format, reusable functions, webhooks, naming |
| [Django Template Standards](standards/django-template-standards.md) | 3-folder structure (CSS/JS/HTML separated), template inheritance, no inline styles or scripts, data-* pattern |
| [Frontend & UI Standards](standards/frontend-ui-standards.md) | Forms, validation, loading states, error handling, Add/Edit consistency, design system, mobile |
| [Website Replication Standards →](standards/website/README.md) | Full website build and migration reference — split into 5 focused files: |
| &nbsp;&nbsp;[01 — Scoping & Design](standards/website/01-scoping-and-design.md) | Ownership check, replication type, client intake, page inventory, design token extraction |
| &nbsp;&nbsp;[02 — Building by Platform](standards/website/02-building-by-platform.md) | React vs WeWeb vs Bubble vs WordPress decision, Figma-to-platform build flows |
| &nbsp;&nbsp;[03 — Migration](standards/website/03-migration.md) | Existing site migration, WordPress audit, content migration, SEO preservation, forms replication |
| &nbsp;&nbsp;[04 — Responsive, QA & Performance](standards/website/04-responsive-qa-performance.md) | Breakpoints, pixel-perfect QA, performance, accessibility, security, animation |
| &nbsp;&nbsp;[05 — Launch & Handoff](standards/website/05-launch-and-handoff.md) | Tracking/analytics, redirects, documentation, QA checklists, client review, definition of done |

---

## Checklists

> Run the relevant checklist **before** marking any task done. Each checklist is a gate — not optional.

### General Checklists

| Checklist | Who Uses It |
|-----------|------------|
| [Backend Checklist](checklists/backend-checklist.md) | Backend developer — before marking any API or feature complete |
| [Frontend Checklist](checklists/frontend-checklist.md) | Frontend developer — before submitting any task |
| [Website Launch Checklist](checklists/website-launch-checklist.md) | Website builder — before going live |
| [Task Completion Checklist](checklists/task-completion-checklist.md) | All roles — universal task completion gate |

### Frontend Checklists → [checklists/frontend/](checklists/frontend/)

Detailed checklists for specific frontend tasks and UI patterns.

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](checklists/frontend/form-field-validation.md) | Building any form — covers every input type: text, email, phone (E.164), location/address (Google Places), date, time, datetime, file upload (MIME type), image upload, OTP, autocomplete, and more |
| [Login & Signup Standards](checklists/frontend/login-signup.md) | Building login, signup, forgot password, reset password, email verification, or OAuth/social login flows |
| [Add / Edit Consistency](checklists/frontend/add-edit-consistency.md) | Building Add or Edit forms for the same entity — shared component requirement, validation consistency, pre-fill behavior, UI matching |

### Supabase Checklists → [checklists/supabase/](checklists/supabase/)

Step-by-step checklists for every Supabase backend task.

| Checklist | Use When |
|-----------|----------|
| [New Table](checklists/supabase/new-table.md) | Creating any new database table — schema, required columns, naming, constraints, indexes, RLS, migration, documentation |
| [RLS Policies](checklists/supabase/rls-policies.md) | Writing or reviewing Row Level Security — policy patterns by access type, role permission matrix, anti-patterns, full testing guide |
| [New API Endpoint](checklists/supabase/new-api-endpoint.md) | Building any new endpoint (direct query, RPC, or Edge Function) — auth, input validation, response format, error codes, testing |
| [Edge Function](checklists/supabase/edge-function.md) | Building a Supabase Edge Function — required structure, CORS, auth pattern, secrets, error handling, webhook signature verification |

### Xano Checklists → [checklists/xano/](checklists/xano/)

| Checklist | Use When |
|-----------|----------|
| [New API Endpoint](checklists/xano/new-endpoint.md) | Building any Xano endpoint — auth-first step, DB permission check, input validation, response format, reusable functions, documentation |

### Django Checklists → [checklists/django/](checklists/django/)

| Checklist | Use When |
|-----------|----------|
| [New View](checklists/django/new-view.md) | Building any Django view — auth decorators, form validation, services.py pattern, response handling, error mapping, testing |
| [Template](checklists/django/template.md) | Building any Django HTML template — 3-folder structure, base.html inheritance, no inline styles/scripts, data-* attribute pattern, CSRF helper |

### Website Checklists → [checklists/website/](checklists/website/)

| Checklist | Use When |
|-----------|----------|
| [Website Launch Checklist](checklists/website-launch-checklist.md) | Pre-launch gate — design, responsive, forms, SEO, redirects, analytics, performance, accessibility, go-live, handoff |

---

## Templates

> Copy these into tickets, PRs, and documentation. Fill in the placeholders — do not start from scratch.

| Template | Use When |
|----------|----------|
| [Work Log Template](templates/work-log-template.md) | Logging time and posting status updates in a ticket — includes In Progress, Completed, and Blocked formats |
| [Bug Report Template](templates/bug-report-template.md) | Reporting any bug or blocker found during work — steps to reproduce, expected vs actual, severity, environment |
| [Completion Comment Template](templates/completion-comment-template.md) | Marking a task done with proper evidence — what was done, how to test, screenshots/links, time spent |
| [API Documentation Template](templates/api-doc-template.md) | Documenting any backend API for the frontend team — endpoint details, request payload, all error cases, frontend code example |
| [Form Documentation Template](templates/form-doc-template.md) | Documenting form fields, validation rules, submission behavior, and API integration |

---

## Tasks

> Realistic practice tasks to build with what you've learned. Each task has a scenario, specific requirements, a "What NOT to do" section, and links directly to the checklist to run when done. Tasks are linked from the bottom of each relevant checklist — click "Practice Task" at the end of any checklist to go straight to the exercise.

### Frontend Tasks → [tasks/frontend/](tasks/frontend/)

| Task | What You Practice |
|------|------------------|
| [01 — Registration Form](tasks/frontend/01-registration-form.md) | Email, password strength, confirm password, phone E.164, checkbox, loading/error/success states |
| [02 — Login Form with Forgot & Reset Password](tasks/frontend/02-login-form.md) | Auth flow, email enumeration prevention, token validation on page load, forgot password states |
| [03 — Shared Add / Edit Project Form](tasks/frontend/03-add-edit-project-form.md) | One shared component, one schema, identical behavior in Add and Edit, pre-fill skeleton loader |

### Supabase Tasks → [tasks/supabase/](tasks/supabase/)

| Task | What You Practice |
|------|------------------|
| [01 — Create the Projects Table](tasks/supabase/01-create-projects-table.md) | Schema, constraints, CHECK, soft delete, updated_at trigger, composite index, migration file |
| [02 — Write RLS Policies](tasks/supabase/02-rls-policies.md) | All four policy types, workspace membership subquery, role-based rules, WITH CHECK, full role testing |
| [03 — Create Project API (RPC)](tasks/supabase/03-create-project-api.md) | Atomic RPC (project + task list + audit log), RAISE EXCEPTION error codes, API documentation |
| [04 — Project Invite Edge Function](tasks/supabase/04-invite-edge-function.md) | Edge Function structure, JWT auth, Deno secrets, Resend email, logging rules |

### Xano Tasks → [tasks/xano/](tasks/xano/)

| Task | What You Practice |
|------|------------------|
| [01 — Create Project Endpoint](tasks/xano/01-create-project-endpoint.md) | Auth-first, reusable permission function, DB role check, field-level validation, standard error format |

### Django Tasks → [tasks/django/](tasks/django/)

| Task | What You Practice |
|------|------------------|
| [01 — Project List & Create View](tasks/django/01-project-list-create-view.md) | LoginRequiredMixin, WorkspaceMemberMixin, form validation, services.py separation, POST→redirect→GET |
| [02 — Project Detail Template](tasks/django/02-project-template.md) | 3-folder structure, base.html, zero inline styles/scripts, data-* attributes for JS, responsive CSS |

### Website Tasks → [tasks/website/](tasks/website/)

| Task | What You Practice |
|------|------------------|
| [01 — Replicate a Landing Page](tasks/website/01-landing-page-section.md) | Pixel-perfect replication, design tokens, reusable components, 3 breakpoints, semantic HTML, performance |
