# Frontend & UI Standards

> **Core Rule:** The frontend is the user's entire experience of the product. Every state must be handled. Every error must be shown. Every action must give feedback. Every permission must be reflected — but enforced on the backend.

---

## Contents

| File | What It Covers |
|------|---------------|
| [01 — Forms & Validation](01-forms-validation.md) | Form structure, field standards, required markers, validation timing, error display, Add/Edit consistency, password, phone, disabled fields |
| [02 — Data Display](02-data-display.md) | Tables, lists, cards, sorting, filtering, search (debounce, URL params, backend-sent), pagination, infinite scroll, empty states |
| [03 — UI Components](03-ui-components.md) | Modals (open/close, focus trap, accessibility), notifications and toasts (types, timing, stacking), loading states, skeleton screens, buttons |
| [04 — Error & Feedback](04-error-feedback.md) | Error types and where they appear, API error code mapping, empty states, destructive action confirmation (3 risk levels), no silent failures |
| [05 — Permissions & Auth UI](05-permissions-auth.md) | Role-based UI (hide vs disable), reading role from backend, page-level access, partial access, real-time role changes |
| [06 — Settings & Profile](06-settings.md) | Settings page structure, save behavior, password change, avatar upload, notification preferences, danger zone |

---

## The Non-Negotiables

These apply to every frontend task without exception:

```text
[ ] Add and Edit for the same entity use the same component and the same validation schema
[ ] Every async action has a loading state — button spinner, disabled while loading, no double submit
[ ] Every error is shown in the UI — never only in the console
[ ] No destructive action without a confirmation dialog
[ ] Mobile tested before marking done (375px and 768px)
[ ] Backend enforces all rules — frontend reflects permissions, never enforces them alone
[ ] Search and filters go to the backend — never filter a client-side array
[ ] State that should be shareable lives in URL params (search, filter, sort, page)
[ ] No silent failures anywhere in the app
```

---

## Related

- [Frontend & UI Standards (original)](../frontend-ui-standards.md) — Original single-file reference (form standards, notifications, loading, delete confirmation, design system, error contract)
- [Code Reusability Standards](../code-reusability-standards.md) — When to extract components, hooks, and services
- [Backend-First Logic](../backend-first-logic.md) — What belongs on the backend vs frontend
- [Frontend Checklists](../../checklists/frontend/README.md) — Run these before marking any frontend task done
