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

**Forms & Auth**

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](frontend/form-field-validation.md) | Building any form — every input type: text, email, phone (E.164), location/Google Places, date, time, file upload (MIME type), OTP, autocomplete, and more |
| [Login & Signup Standards](frontend/login-signup.md) | Building login, signup, forgot password, reset password, email verification, or OAuth/social login |
| [Add / Edit Consistency](frontend/add-edit-consistency.md) | Building Add or Edit forms for the same entity — shared component, identical validation, pre-fill behavior |
| [Multi-Step Forms](frontend/multi-step-forms.md) | Building any wizard or multi-step flow — step validation, progress indicator, back/forward, data preservation |

**Data Display**

| Checklist | Use When |
|-----------|----------|
| [Tables & Data Lists](frontend/tables-data-lists.md) | Building any table or list — skeleton loading, empty state, error state, sorting, filtering, pagination, row/bulk actions |
| [Search, Filters & Pagination](frontend/search-filters-pagination.md) | Adding search, filters, or pagination — debounce, URL state, backend-sent, empty states |

**UI Components**

| Checklist | Use When |
|-----------|----------|
| [Modals & Dialogs](frontend/modals-dialogs.md) | Building any modal — open/close, focus trap, ESC key, form modals, confirmation dialogs, accessibility |
| [Notifications & Toasts](frontend/notifications-toasts.md) | Adding feedback to any async action — types, auto-dismiss, stacking, inline banners, message standards |
| [Delete & Destructive Actions](frontend/delete-destructive-actions.md) | Any delete, archive, or irreversible action — risk levels, confirmation standards, type-to-confirm, soft vs hard delete |

**States & Behavior**

| Checklist | Use When |
|-----------|----------|
| [Loading States & Skeletons](frontend/loading-states-skeletons.md) | Any async action or data fetch — button loading, skeleton screens, page loading, table loading |
| [Error Handling](frontend/error-handling.md) | Any page or feature calling an API — field errors, form banners, page errors, network failures, auth errors |
| [Settings & Profile Pages](frontend/settings-profile.md) | Building any settings section — save behavior, password change, avatar upload, notification prefs, danger zone |
| [Permissions & Role-Based UI](frontend/permissions-role-ui.md) | Any UI dependent on the user's role — hide vs disable, tooltip for disabled, page access, role read from backend |

---

## Supabase → [supabase/](supabase/)

Step-by-step checklists for every Supabase backend task.

| Checklist | Use When |
|-----------|----------|
| [New Table](supabase/new-table.md) | Creating any new database table — schema, required columns, naming conventions, constraints, indexes, RLS setup, migration file, documentation |
| [RLS Policies](supabase/rls-policies.md) | Writing or reviewing Row Level Security — policy patterns by access type, role permission matrix, anti-patterns, full testing guide per role |
| [New API Endpoint](supabase/new-api-endpoint.md) | Building any new endpoint (direct query / RPC / Edge Function) — auth, input validation, response format, error codes, testing requirements |
| [Edge Function](supabase/edge-function.md) | Building a Supabase Edge Function — required structure, CORS, auth pattern, secrets management, error handling, webhook signature verification |
| [RPC Function](supabase/rpc-function.md) | Writing any Postgres RPC function — SECURITY mode, parameter naming, validation order, error format, transaction safety |
| [Storage Bucket](supabase/storage-bucket.md) | Creating a storage bucket — visibility, RLS policies, file path structure, signed URLs, upload/delete flow |
| [Database Trigger](supabase/database-trigger.md) | Writing a database trigger — BEFORE vs AFTER, trigger function standards, timestamps, audit log pattern |

---

## Xano → [xano/](xano/)

| Checklist | Use When |
|-----------|----------|
| [New API Endpoint](xano/new-endpoint.md) | Building any Xano endpoint — auth token as first step, DB-based permission check, input validation order, standard error format, reusable function extraction |
| [Reusable Function](xano/reusable-function.md) | Extracting shared logic into a Xano Custom Function — naming, single responsibility, inputs/outputs, error raising, testing |
| [Webhook Endpoint](xano/webhook-endpoint.md) | Building a Xano webhook receiver — signature verification, idempotency, payload handling, response standards, logging |

---

## Django → [django/](django/)

| Checklist | Use When |
|-----------|----------|
| [New View](django/new-view.md) | Building any Django view — auth decorators, form/serializer validation, services.py pattern, response handling, error mapping, testing |
| [Template](django/template.md) | Building any Django HTML template — 3-folder structure, base.html inheritance, zero inline styles/scripts, data-* attribute pattern, CSRF helper |
| [Model](django/model.md) | Creating or modifying a Django model — field types, relationships, on_delete, null/blank, constraints, indexes, Meta class, migrations |
| [Form / Serializer](django/form-serializer.md) | Building a Django Form or DRF Serializer — field definitions, field-level vs cross-field validation, read_only/write_only fields |
| [DRF API View](django/api-view.md) | Building a DRF API view — view type selection, permission classes, serializer wiring, service delegation, response format, pagination |

---

## Website → [website/](website/)

| Checklist | Use When |
|-----------|----------|
| [Website Launch Checklist](website-launch-checklist.md) | Pre-launch gate — design, responsive testing, forms, SEO, migration redirects, analytics, performance, accessibility, go-live, client handoff |
| [WeWeb Page Build](website/weweb-page.md) | Building a WeWeb page — global layout, data binding (all 4 states), Xano calls, forms, reusable components, mobile |
| [React Page Build](website/react-page.md) | Building a React page — component structure, data fetching hooks, shared Add/Edit forms, TypeScript, accessibility, mobile |

---

## Third-Party Integrations → [third-parties-implementation/](third-parties-implementation/)

| Checklist | Use When |
|-----------|----------|
| [Stripe Integration](third-parties-implementation/stripe-integration.md) | Adding any Stripe payment flow — keys, customers, Payment Intents, Checkout, subscriptions, webhooks, metadata, error handling, refunds |
| [Razorpay Integration](third-parties-implementation/razorpay-integration.md) | Adding any Razorpay payment flow — keys, order creation, payment verification (HMAC SHA256), Checkout.js, subscriptions, webhooks, refunds |
| [SendGrid Integration](third-parties-implementation/sendgrid-integration.md) | Setting up SendGrid — domain auth, API key, template decision, code-rendered vs dynamic templates, error handling, bounce management |
| [Gmail Integration](third-parties-implementation/gmail-integration.md) | Using Gmail SMTP — App Passwords, Django config, environment check, when to switch to SendGrid |
