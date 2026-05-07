# Frontend Checklists

> Use these before marking any frontend task done. Each checklist targets a specific area of frontend work.

---

| Checklist | Use When |
|-----------|----------|
| [Form Field Validation](form-field-validation.md) | Building any form — covers every input type (text, email, phone, location, date/time, file upload, OTP, and more) |
| [Login & Signup Standards](login-signup.md) | Building login, signup, forgot password, reset password, or email verification flows |
| [Add / Edit Consistency](add-edit-consistency.md) | Building Add or Edit forms for the same entity — ensures they are identical in validation, layout, and behavior |

---

## Quick Reference — Core Frontend Rules

```text
[ ] Add and Edit for the same entity use the same component and validation schema
[ ] Every async action shows a loading state
[ ] Every error is shown in the UI — never only in the console
[ ] No destructive action without a confirmation dialog
[ ] Mobile tested before marking done
[ ] Backend enforces all rules — frontend validation is UX only
[ ] No hardcoded roles, statuses, or routes — use constants
[ ] Check if a component or hook already exists before building a new one
```

---

## Related Standards

- [Frontend & UI Standards](../../standards/frontend-ui-standards.md) — Full frontend standards reference
- [Code Reusability Standards](../../standards/code-reusability-standards.md) — When to extract and share components
- [Backend-First Logic](../../standards/backend-first-logic.md) — What belongs on the backend vs frontend
