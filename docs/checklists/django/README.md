# Django Checklists

> Use these before marking any Django task done.

---

| Checklist | Use When |
|-----------|----------|
| [New View](new-view.md) | Building any Django view — auth, permissions, form validation, business logic placement, response handling |
| [Template](template.md) | Building any Django HTML template — 3-folder structure, inheritance, no inline styles/scripts, data-* pattern |

---

## Quick Reference — Core Django Rules

```text
[ ] Business logic in services.py — NOT in views, templates, or models
[ ] Every protected view uses @login_required or LoginRequiredMixin
[ ] All user input validated through Django Form or Serializer (not raw request.POST)
[ ] No <style> blocks in templates — CSS in static/css/
[ ] No <script> blocks with logic in templates — JS in static/js/
[ ] Every template extends base.html
[ ] Repeated HTML sections use {% include %} — not copy-pasted
[ ] Django data passed to JS via data-* attributes — never string interpolation in <script>
[ ] Role/permissions read from database — never from request payload or session
```

---

## Related Standards

- [Backend-First Logic Standard](../../standards/backend-first-logic.md) — What belongs on backend vs frontend
- [Django Template Standards](../../standards/django-template-standards.md) — Full template reference
- [Code Reusability Standards](../../standards/code-reusability-standards.md) — When to extract services, managers, and mixins
