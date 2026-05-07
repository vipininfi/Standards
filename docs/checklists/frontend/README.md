# Frontend Checklists

> Use these before marking any frontend task done. Each checklist targets a specific area of frontend work.

---

## Forms & Auth

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](form-field-validation.md) | Building any form field — covers every input type (text, email, phone E.164, location/Google Places, date, time, file upload with MIME type, OTP, autocomplete, and more) |
| [Login & Signup Standards](login-signup.md) | Building login, signup, forgot password, reset password, email verification, or OAuth/social login |
| [Add / Edit Consistency](add-edit-consistency.md) | Building Add or Edit forms for the same entity — shared component, identical validation schema, pre-fill behavior |
| [Multi-Step Forms](multi-step-forms.md) | Building any wizard or multi-step flow — step validation, progress indicator, back/forward, data preservation, final-step-only submission |

---

## Data Display

| Checklist | Use When |
|-----------|----------|
| [Tables & Data Lists](tables-data-lists.md) | Building any table or list — loading skeleton, empty state, error state, sorting, filtering, pagination, row actions, bulk actions, mobile layout |
| [Search, Filters & Pagination](search-filters-pagination.md) | Adding search, filter controls, or pagination to any page — debounce, URL params, backend-sent, empty states, combined state |

---

## UI Components

| Checklist | Use When |
|-----------|----------|
| [Modals & Dialogs](modals-dialogs.md) | Building any modal or dialog — open/close behavior, focus trap, ESC key, backdrop, form modals, confirmation dialogs, accessibility |
| [Notifications & Toasts](notifications-toasts.md) | Adding notifications to any async action — types (success/error/warning/info), auto-dismiss, stacking, inline banners, message content standards |
| [Delete & Destructive Actions](delete-destructive-actions.md) | Any delete button, archive action, or irreversible operation — risk levels, confirmation dialog standards, type-to-confirm, soft vs hard delete |

---

## States & Behavior

| Checklist | Use When |
|-----------|----------|
| [Loading States & Skeletons](loading-states-skeletons.md) | Any async action or data fetch — button loading states, skeleton screens, page loading, table loading, inline component loading |
| [Error Handling](error-handling.md) | Any page or feature that calls an API — field errors, form banner errors, page-level errors, network failures, permission errors, auth errors |
| [Settings & Profile Pages](settings-profile.md) | Building any settings or profile section — save behavior, password change, avatar upload, notification preferences, danger zone |
| [Permissions & Role-Based UI](permissions-role-ui.md) | Any UI element that depends on the user's role — hide vs disable, tooltip for disabled, page-level access, reading role from backend |

---

## Quick Reference — Core Frontend Rules

```text
[ ] Add and Edit for the same entity use the same component and validation schema
[ ] Every async action shows a loading state — no exceptions
[ ] Every error is shown in the UI — never only in the console
[ ] No destructive action without a confirmation dialog
[ ] Search and filters go to the backend — never filter a client-side array
[ ] State that should be shareable lives in URL params
[ ] Backend enforces all rules — frontend reflects permissions, never enforces them alone
[ ] Mobile tested before marking done
[ ] No silent failures anywhere
```

---

## Related Standards

- [Frontend & UI Standards](../../standards/frontend-ui-standards.md) — Full frontend standards reference
- [Code Reusability Standards](../../standards/code-reusability-standards.md) — When to extract components and hooks
- [Backend-First Logic](../../standards/backend-first-logic.md) — What belongs on the backend vs frontend
