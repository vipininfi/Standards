# Xano Checklists

> Use these before marking any Xano backend task done.

---

| Checklist | Use When |
|-----------|----------|
| [New API Endpoint](new-endpoint.md) | Building any new Xano endpoint — auth, permissions, validation, response format, reusable functions, documentation |
| [Reusable Function](reusable-function.md) | Extracting shared logic into a Xano Custom Function — naming, single responsibility, inputs/outputs, error raising, testing |
| [Webhook Endpoint](webhook-endpoint.md) | Building a Xano webhook receiver — signature verification, idempotency, payload handling, response standards, logging |

---

## Quick Reference — Core Xano Rules

```text
[ ] Auth token validation is the FIRST step in every protected endpoint
[ ] Role is ALWAYS read from the database — never from the request payload
[ ] Workspace membership verified before any data is read or written
[ ] All inputs validated BEFORE any business logic runs
[ ] Errors returned in standard format: { status, code, message }
[ ] Shared logic (3+ uses) extracted to a Xano Function
[ ] Every endpoint documented before frontend handoff
[ ] Every endpoint tested for: success, unauthorized, forbidden, not found, duplicate, validation error
```

---

## Related Standards

- [Xano Standards](../../standards/xano-standards.md) — Full Xano reference
- [Backend-First Logic](../../standards/backend-first-logic.md) — What belongs on backend vs frontend
- [Code Reusability Standards](../../standards/code-reusability-standards.md) — When to extract Xano Functions
