# Standards

> Read the relevant standard before you start building. These define how the team works — not suggestions, but rules everyone is expected to know and apply.

---

## Cross-Platform Standards

These apply to every team member regardless of role or platform.

| Document | What It Covers |
|----------|---------------|
| [Backend-First Logic](backend-first-logic.md) | The fundamental rule: auth, roles, validation, prices, business rules, and file validation are always enforced on the backend. Frontend validates for UX only — it never enforces. Covers anti-patterns to avoid and platform-specific rules for Supabase, Xano, Django, WeWeb, and React. |
| [Code Reusability Standards](code-reusability-standards.md) | Before you build anything — check if it already exists. When to extract to a shared component, hook, service, or function. Decision rules (3+ identical uses = extract), platform-specific guides for React, WeWeb, Xano, Django, and Supabase, naming conventions, and when NOT to over-abstract. |
| [Team Ownership Standards](team-ownership.md) | How the team works — work update format (In Progress / Completed / Blocked), issue raising, ticket discipline, PR standards, communication rules, escalation process, and accountability for bugs. |
| [QA & Delivery Standards](qa-delivery-standards.md) | What "done" actually means — self-testing checklist, definition of done per role, required test cases, RLS testing matrix, QA handoff checklist, and a summary table across all roles. |
| [Frontend & UI Standards](frontend-ui-standards.md) | UI rules for React and WeWeb — Add/Edit consistency (same component, same schema), form standards, required field indicators, password validation, phone input (E.164), loading states, error display, notifications, delete confirmation, design system, and backend+frontend error contract. |
| [Django Template Standards](django-template-standards.md) | Three-folder rule (CSS in `static/css/`, JS in `static/js/`, HTML in `templates/`), base.html inheritance, no inline styles or scripts, Django data to JavaScript via `data-*` attributes (never string interpolation), reusable `{% include %}` components, static file loading with `{% static %}`, naming conventions. |

---

## Supabase Standards → [supabase/](supabase/)

Split into 6 focused files covering the full Supabase backend lifecycle.

| Document | What It Covers |
|----------|---------------|
| [Overview & Definition of Done](supabase/README.md) | Developer responsibilities, full definition of done checklist, the key questions every API must answer |
| [01 — Database & RLS](supabase/01-database-and-rls.md) | Table schema standards, naming conventions (plural snake_case), required constraints, index rules, RLS enabled on every table, policy types (SELECT/INSERT/UPDATE/DELETE), RLS anti-patterns, auth standards, service role key rules |
| [02 — API Design](supabase/02-api-design.md) | Choosing between direct table query / RPC / Edge Function, 15-step core build flow, response format standard (success and error JSON), standard error codes, frontend contract rules |
| [03 — API Documentation](supabase/03-api-documentation.md) | Full documentation template (17 fields), frontend Supabase client call examples, RPC call pattern, complete worked example (Add Project API with all error cases) |
| [04 — Edge Functions](supabase/04-edge-functions.md) | When to use Edge Functions (vs RPC), required structure, 10 mandatory steps, response standard helper, JWT auth pattern, full example (Send Project Invite) |
| [05 — Validation, Multi-Tenant & Storage](supabase/05-validation-multitenant-storage.md) | Three validation layers (frontend / API / database), frontend error handling contract, field-level error format, multi-tenant workspace scoping, workspace membership SQL, role permission matrix, storage bucket security, storage path standard |
| [06 — Operations & Testing](supabase/06-operations-and-testing.md) | Audit log table and what to log, Edge Function logging rules, webhook signature and idempotency, API key management, versioning, migration standards, 13 required test cases, RLS testing matrix (6 roles × 4 operations), backend handoff checklist |

---

## Xano Standards

| Document | What It Covers |
|----------|---------------|
| [Xano Standards](xano-standards.md) | Auth token as the first step in every endpoint, permission check from the database (never from request), three-layer input validation, standard error response format with error codes, reusable Xano Functions (naming: verb_noun, group organization), database naming conventions, webhook signature and idempotency, environment variables, API documentation standard, definition of done |

---

## Website Standards → [website/](website/)

Split into 5 focused files covering the full website build and migration lifecycle.

| Document | What It Covers |
|----------|---------------|
| [Overview & Replication Types](website/README.md) | Six replication types (Pixel-Perfect / Functional / Migration / Modernized / Backend / CMS), full definition of done checklist, 10 questions to confirm scope |
| [01 — Scoping & Design](website/01-scoping-and-design.md) | Ownership and permission confirmation, client intake checklist, discovery audit, page inventory template, design extraction from Figma and deployed sites, design token standard (colors, typography, spacing, breakpoints) |
| [02 — Building by Platform](website/02-building-by-platform.md) | Technology decision table (React vs WeWeb vs Bubble vs WordPress), Figma-to-React development flow and component list, Figma-to-WeWeb build standards and anti-patterns, Figma-to-Bubble build standards and anti-patterns |
| [03 — Migration](website/03-migration.md) | Existing site reverse-engineering checklist, migration flow, WordPress audit and migration decisions, content migration standards, SEO preservation checklist and anti-patterns, forms and integrations replication |
| [04 — Responsive, QA & Performance](website/04-responsive-qa-performance.md) | Breakpoint standards, responsive checklist per screen size, pixel-perfect QA process, performance rules (image optimization, lazy loading, Core Web Vitals), accessibility must-haves, security checklist, CMS decision table, no-code naming standards, animation replication |
| [05 — Launch & Handoff](website/05-launch-and-handoff.md) | Header/footer documentation, tracking and analytics installation, redirect mapping, launch checklist, required documentation, form/API handoff template, QA checklists (visual / functional / SEO / performance), client review standard, common risks, platform-specific risks, definition of done, client message template |
