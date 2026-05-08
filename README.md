# Working Standards

> **Core Rule:** Own the work, think beyond the ticket, communicate clearly, test before handoff, and make every change traceable.

This is the team's single source of truth for how we build. Every team member is expected to know these standards and apply them before marking anything done.

---

## Non-Negotiables (Everyone, Every Task)

These apply regardless of role, platform, or task type.

| # | Rule |
|---|------|
| 1 | **No silent failures** — every error must be visible in the UI. |
| 2 | **No unclear "done"** — every completion needs evidence in the ticket. |
| 3 | **No untested handoff** — test your own work before marking complete. |
| 4 | **No inconsistent forms** — Add and Edit for the same entity must behave identically. |
| 5 | **No API error only in console** — show it to the user. |
| 6 | **No missing loading state** — every async action shows loading. |
| 7 | **No destructive action without confirmation.** |
| 8 | **No important issue only in chat** — it must be in the ticket. |
| 9 | **No shared logic changed without checking impact** across all places that use it. |
| 10 | **No mobile skipped** — every UI must be checked on mobile before done. |
| 11 | **Backend is the source of truth** — never enforce rules only on frontend. |
| 12 | **Check before you build** — if it exists or will be reused, extract it first. |

---

## What Are You Working On?

| Task | Standard | Quick Checklist |
|------|----------|-----------------|
| Building a Supabase API, table, or RLS | [Supabase Standards →](docs/standards/supabase/README.md) | [Backend Checklist](docs/checklists/backend-checklist.md) |
| Building a Xano API or workflow | [Xano Standards](docs/standards/xano-standards.md) | [Backend Checklist](docs/checklists/backend-checklist.md) |
| Building a Django view, model, or service | [Backend-First Logic Standard](docs/standards/backend-first-logic.md) | [Backend Checklist](docs/checklists/backend-checklist.md) |
| Building Django HTML templates | [Django Template Standards](docs/standards/django-template-standards.md) | [Frontend Checklist](docs/checklists/frontend-checklist.md) |
| Building a React page, component, or form | [Frontend & UI Standards](docs/standards/frontend-ui-standards.md) | [Frontend Checklist](docs/checklists/frontend-checklist.md) |
| Building in WeWeb | [Frontend & UI Standards](docs/standards/frontend-ui-standards.md) + [Xano Standards](docs/standards/xano-standards.md) | [Frontend Checklist](docs/checklists/frontend-checklist.md) |
| Replicating or building a website (Figma/existing/migration) | [Website Replication Standards →](docs/standards/website/README.md) | [Website Launch Checklist](docs/checklists/website-launch-checklist.md) |
| Deciding whether to make something reusable | [Code Reusability Standards](docs/standards/code-reusability-standards.md) | — |
| Deciding what logic goes where (frontend vs backend) | [Backend-First Logic Standard](docs/standards/backend-first-logic.md) | — |
| Writing a work update or logging time | [Team Ownership Standards](docs/standards/team-ownership.md) | [Work Log Template](docs/templates/work-log-template.md) |
| Marking a task as complete | [Completion Template](docs/templates/completion-comment-template.md) | [Task Completion Checklist](docs/checklists/task-completion-checklist.md) |
| Reporting a bug or blocker | [Bug Report Template](docs/templates/bug-report-template.md) | — |
| Documenting an API for frontend | [API Documentation Template](docs/templates/api-doc-template.md) | — |
| Documenting a form | [Form Documentation Template](docs/templates/form-doc-template.md) | — |
| Practicing a standard with a real task | [Practice Tasks →](docs/tasks/README.md) | — |

---

## By Role

### Backend Developer (Supabase)
**Start here:** [Supabase Standards](docs/standards/supabase/README.md)

Key rules:
- Every exposed table must have RLS enabled with separate SELECT, INSERT, UPDATE, DELETE policies.
- No API is complete without documentation and test cases.
- Service role key must **never** appear in frontend code.
- Read [Backend-First Logic](docs/standards/backend-first-logic.md) — the backend enforces everything. Frontend only displays.

---

### Backend Developer (Xano)
**Start here:** [Xano Standards](docs/standards/xano-standards.md)

Key rules:
- Auth token validation is the **first step** in every protected endpoint.
- Role is always read from the database — never from the request payload.
- All inputs validated in Xano — frontend validation is UX only.
- Shared logic extracted to Xano Functions — never duplicated across endpoints.
- Read [Backend-First Logic](docs/standards/backend-first-logic.md) — frontend cannot be the enforcement layer.

---

### Backend Developer (Django)
**Start here:** [Backend-First Logic Standard](docs/standards/backend-first-logic.md)

Key rules:
- Business logic lives in `services.py` — not in views, not in templates.
- Every view that requires login uses `@login_required` or `LoginRequiredMixin`.
- Validation happens in forms/serializers — never skip it because "frontend validates."
- Read [Code Reusability Standards](docs/standards/code-reusability-standards.md) for service and utility extraction patterns.

---

### Django Template Developer
**Start here:** [Django Template Standards](docs/standards/django-template-standards.md)

Key rules:
- Three folders: `static/css/`, `static/js/`, `templates/` — kept separate, always.
- No `<style>` blocks in templates. No `<script>` blocks with logic in templates.
- All pages extend `base.html`. Repeated sections use `{% include %}`.
- Django data passed to JavaScript using `data-*` attributes — never string interpolation in `<script>` tags.

---

### Frontend Developer (React / WeWeb)
**Start here:** [Frontend & UI Standards](docs/standards/frontend-ui-standards.md)

Key rules:
- Add and Edit for the same entity must use the **same component** and validation schema.
- Every async action shows a loading state.
- Never `console.log(error)` without showing it in the UI.
- Check [Code Reusability Standards](docs/standards/code-reusability-standards.md) before building any component or hook.
- Read [Backend-First Logic](docs/standards/backend-first-logic.md) — frontend hides UI, backend enforces rules.

---

### Website Builder (React / WeWeb / Bubble / WordPress)
**Start here:** [Website Replication Standards](docs/standards/website/README.md)

Key rules:
- Confirm client ownership/rights before replicating any design or content.
- Audit the full page inventory before writing any code.
- For migrations: preserve all SEO URLs, metadata, and add 301 redirects.
- No no-code build without global styles, reusable components, and responsive testing.

---

### All Team Members
**Start here:** [Team Ownership Standards](docs/standards/team-ownership.md)

Key rules:
- Read the task fully before starting. Ask questions before building unclear requirements.
- All updates go in the ticket — not just in chat.
- Every update must state: what was done, time spent, current status, next step.
- Own your bugs. If your change caused an issue, raise and fix it.
- **Flow:** Understand → Clarify → Plan → Build → Test → Update → Raise Issues → Complete with Evidence

---

## Standards Library

| Document | What It Covers |
|----------|----------------|
| [Supabase Standards](docs/standards/supabase/README.md) → [Database & RLS](docs/standards/supabase/01-database-and-rls.md) · [API Design](docs/standards/supabase/02-api-design.md) · [API Docs](docs/standards/supabase/03-api-documentation.md) · [Edge Functions](docs/standards/supabase/04-edge-functions.md) · [Multi-Tenant & Storage](docs/standards/supabase/05-validation-multitenant-storage.md) · [Operations & Testing](docs/standards/supabase/06-operations-and-testing.md) · [RPC Functions](docs/standards/supabase/07-rpc-functions.md) | APIs, RLS, database schema, Edge Functions, RPC functions, validation, error handling, webhooks, migrations |
| [Xano Standards](docs/standards/xano-standards.md) | Xano API design, auth, validation, reusable functions, error format, webhooks, naming |
| [Backend-First Logic Standard](docs/standards/backend-first-logic.md) | What belongs on the backend vs frontend, anti-patterns, platform-specific rules |
| [Django Template Standards](docs/standards/django-template-standards.md) | 3-folder structure, template inheritance, no inline styles/scripts, data-* pattern, naming |
| [Website Replication Standards](docs/standards/website/README.md) → [Scoping & Design](docs/standards/website/01-scoping-and-design.md) · [By Platform](docs/standards/website/02-building-by-platform.md) · [Migration](docs/standards/website/03-migration.md) · [QA & Performance](docs/standards/website/04-responsive-qa-performance.md) · [Launch & Handoff](docs/standards/website/05-launch-and-handoff.md) | Figma → React/WeWeb/Bubble, existing site migration, SEO, forms, integrations, launch |
| [Frontend Standards](docs/standards/frontend/README.md) → [Forms & Validation](docs/standards/frontend/01-forms-validation.md) · [Data Display](docs/standards/frontend/02-data-display.md) · [UI Components](docs/standards/frontend/03-ui-components.md) · [Error & Feedback](docs/standards/frontend/04-error-feedback.md) · [Permissions & Auth UI](docs/standards/frontend/05-permissions-auth.md) · [Settings & Profile](docs/standards/frontend/06-settings.md) | Forms, validation, tables, modals, toasts, loading states, errors, permissions, settings — all UI patterns |
| [Frontend & UI Standards](docs/standards/frontend-ui-standards.md) | Original single-file reference — forms, loading states, error handling, design system, components |
| [Code Reusability Standards](docs/standards/code-reusability-standards.md) | When to make things reusable, platform-specific guides (React/WeWeb/Xano/Django/Supabase) |
| [Team Ownership Standards](docs/standards/team-ownership.md) | Work updates, issue raising, communication, accountability, ticket discipline, PR standards |
| [QA & Delivery Standards](docs/standards/qa-delivery-standards.md) | Testing flows, QA checklists, definition of done across all roles |
| [Third-Party Integration Standards](docs/standards/third-parties-implementation/README.md) → [Stripe](docs/standards/third-parties-implementation/stripe.md) · [Razorpay](docs/standards/third-parties-implementation/razorpay.md) · [SendGrid](docs/standards/third-parties-implementation/sendgrid.md) · [Gmail](docs/standards/third-parties-implementation/gmail.md) | Payment gateways (Stripe, Razorpay), email providers (SendGrid, Gmail) — full setup, configuration, webhook standards, metadata, error handling |

---

## Templates

Copy-paste ready — use these in tickets, PRs, and documentation.

| Template | Use When |
|----------|----------|
| [Work Log Template](docs/templates/work-log-template.md) | Logging time and posting status updates in a ticket |
| [Bug Report Template](docs/templates/bug-report-template.md) | Reporting a bug or blocker found during work |
| [Completion Comment Template](docs/templates/completion-comment-template.md) | Marking a task done with proper evidence |
| [API Documentation Template](docs/templates/api-doc-template.md) | Documenting any backend API for the frontend team |
| [Form Documentation Template](docs/templates/form-doc-template.md) | Documenting form fields, validation, and submission behavior |

---

## Checklists

Run these before marking any task done.

### General

| Checklist | Who Uses It |
|-----------|------------|
| [Backend Checklist](docs/checklists/backend-checklist.md) | Backend developer — before marking an API or feature complete |
| [Frontend Checklist](docs/checklists/frontend-checklist.md) | Frontend developer — before submitting any task |
| [Website Launch Checklist](docs/checklists/website-launch-checklist.md) | Website builder — before going live |
| [Task Completion Checklist](docs/checklists/task-completion-checklist.md) | All roles — general task completion gate |

### Frontend Checklists → [docs/checklists/frontend/](docs/checklists/frontend/)

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](docs/checklists/frontend/form-field-validation.md) | Building any form field — every input type (text, email, phone, location, date, file, OTP, and more) |
| [Login & Signup Standards](docs/checklists/frontend/login-signup.md) | Building login, signup, forgot password, reset password, or email verification |
| [Add / Edit Consistency](docs/checklists/frontend/add-edit-consistency.md) | Building Add or Edit forms — ensures identical behavior, validation, and layout |
| [Multi-Step Forms](docs/checklists/frontend/multi-step-forms.md) | Building any wizard or multi-step flow |
| [Tables & Data Lists](docs/checklists/frontend/tables-data-lists.md) | Building any table or list — all states, sorting, filtering, pagination, bulk actions |
| [Search, Filters & Pagination](docs/checklists/frontend/search-filters-pagination.md) | Adding search, filters, or pagination to any page |
| [Modals & Dialogs](docs/checklists/frontend/modals-dialogs.md) | Building any modal — open/close, focus trap, accessibility, form modals |
| [Notifications & Toasts](docs/checklists/frontend/notifications-toasts.md) | Adding feedback to any async action — types, timing, stacking, content standards |
| [Delete & Destructive Actions](docs/checklists/frontend/delete-destructive-actions.md) | Any delete, archive, or irreversible action — risk levels, confirmation standards |
| [Loading States & Skeletons](docs/checklists/frontend/loading-states-skeletons.md) | Any async action or data fetch — buttons, skeletons, page loading, tables |
| [Error Handling](docs/checklists/frontend/error-handling.md) | Any page calling an API — field errors, banners, page errors, network, auth |
| [Settings & Profile Pages](docs/checklists/frontend/settings-profile.md) | Building any settings or profile section |
| [Permissions & Role-Based UI](docs/checklists/frontend/permissions-role-ui.md) | Any UI element that depends on the user's role |

### Supabase Checklists → [docs/checklists/supabase/](docs/checklists/supabase/)

| Checklist | Use When |
|-----------|----------|
| [New Table](docs/checklists/supabase/new-table.md) | Creating any new database table |
| [RLS Policies](docs/checklists/supabase/rls-policies.md) | Writing or reviewing Row Level Security policies |
| [New API Endpoint](docs/checklists/supabase/new-api-endpoint.md) | Building any new Supabase endpoint or RPC |
| [Edge Function](docs/checklists/supabase/edge-function.md) | Building a Supabase Edge Function |
| [RPC Function](docs/checklists/supabase/rpc-function.md) | Writing any Postgres RPC function exposed via supabase.rpc() |
| [Storage Bucket](docs/checklists/supabase/storage-bucket.md) | Creating a storage bucket or adding file upload functionality |
| [Database Trigger](docs/checklists/supabase/database-trigger.md) | Writing a database trigger or trigger function |

### Xano Checklists → [docs/checklists/xano/](docs/checklists/xano/)

| Checklist | Use When |
|-----------|----------|
| [New API Endpoint](docs/checklists/xano/new-endpoint.md) | Building any new Xano endpoint |
| [Reusable Function](docs/checklists/xano/reusable-function.md) | Extracting shared logic into a Xano Custom Function |
| [Webhook Endpoint](docs/checklists/xano/webhook-endpoint.md) | Building a Xano webhook receiver from an external service |

### Django Checklists → [docs/checklists/django/](docs/checklists/django/)

| Checklist | Use When |
|-----------|----------|
| [New View](docs/checklists/django/new-view.md) | Building any Django view |
| [Template](docs/checklists/django/template.md) | Building any Django HTML template |
| [Model](docs/checklists/django/model.md) | Creating or modifying a Django model |
| [Form / Serializer](docs/checklists/django/form-serializer.md) | Building a Django Form or DRF Serializer |
| [DRF API View](docs/checklists/django/api-view.md) | Building a Django REST Framework API view |

### Website Checklists → [docs/checklists/website/](docs/checklists/website/)

| Checklist | Use When |
|-----------|----------|
| [Website Launch Checklist](docs/checklists/website-launch-checklist.md) | Pre-launch gate for all platforms |
| [WeWeb Page Build](docs/checklists/website/weweb-page.md) | Building any WeWeb page or section |
| [React Page Build](docs/checklists/website/react-page.md) | Building any React page or component |

### Third-Party Integration Checklists → [docs/checklists/third-parties-implementation/](docs/checklists/third-parties-implementation/)

| Checklist | Use When |
|-----------|----------|
| [Stripe Integration](docs/checklists/third-parties-implementation/stripe-integration.md) | Adding Stripe payments — keys, customers, Payment Intents, Checkout, subscriptions, webhooks, metadata, refunds |
| [Razorpay Integration](docs/checklists/third-parties-implementation/razorpay-integration.md) | Adding Razorpay payments — keys, orders, payment verification (HMAC SHA256), Checkout.js, subscriptions, webhooks, refunds |
| [SendGrid Integration](docs/checklists/third-parties-implementation/sendgrid-integration.md) | Setting up SendGrid — domain auth, API key, template decision, code-rendered vs dynamic templates, error handling |
| [Gmail Integration](docs/checklists/third-parties-implementation/gmail-integration.md) | Using Gmail SMTP — App Passwords, Django config, limits, when to switch to SendGrid |

---

## Definition of Done

### Backend API (Supabase) is done when:
Schema → Constraints → Indexes → RLS enabled → RLS policies → Auth check → Validation → Error responses → Logs → Tests → **Frontend API documentation** are all complete. See [Supabase Definition of Done](docs/standards/supabase/README.md#definition-of-done).

### Backend API (Xano) is done when:
Auth token check → Permission check → Input validation → Business logic → Standard error format → Reusable functions extracted → API documented → Tested for success + all failure cases.

### Django view/feature is done when:
Permission enforced → Validation in form/serializer → Logic in service → Template has no inline styles/scripts → Tests pass → Business logic not duplicated elsewhere.

### Django template task is done when:
Extends `base.html` → No inline `<style>` → No inline `<script>` logic → CSS in `static/css/` → JS in `static/js/` → Repeated sections use `{% include %}` → Mobile checked.

### Frontend task is done when:
Main flow works → Validation works → Loading state works → Error shown in UI → Mobile checked → Add/Edit consistent → Self-tested → Ticket updated.

### Website replica is done when:
All pages built → Forms tested → Mobile responsive → SEO metadata → Redirects → Analytics → Client approved staging.

### Any task is done when:
Requirement met → Reusability checked → Backend enforces all rules → Self-tested → Ticket updated with evidence → No silent failures.
