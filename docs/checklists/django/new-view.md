# Django — New View Checklist

> **Core Rule:** Views handle the HTTP request and response — nothing more. Business logic lives in `services.py`. Validation lives in forms or serializers. Templates handle display only. Every view that requires login is protected with `@login_required` or `LoginRequiredMixin`.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-before-you-build) | Before You Build |
| [2](#2-view-type-decision) | View Type Decision |
| [3](#3-authentication--authorization) | Authentication & Authorization |
| [4](#4-form--input-validation) | Form & Input Validation |
| [5](#5-business-logic-placement) | Business Logic Placement |
| [6](#6-response-handling) | Response Handling |
| [7](#7-error-handling) | Error Handling |
| [8](#8-reusability--code-organization) | Reusability & Code Organization |
| [9](#9-testing-requirements) | Testing Requirements |
| [10](#10-django-view-checklist--before-marking-done) | Django View Checklist — Before Marking Done |

---

# 1. Before You Build

```text
[ ] What does this view do? (one clear sentence)
[ ] What URL will it be accessible at?
[ ] Does a similar view already exist that could be extended?
[ ] Does the required business logic already exist in a service?
[ ] What permissions does this view require?
[ ] What form/serializer will validate the input?
[ ] What does the success response look like? (redirect, template, JSON)
[ ] What does the failure response look like?
```

---

# 2. View Type Decision

| Use This | When |
|----------|------|
| Function-based view (`def`) | Simple, one-off views with minimal logic |
| Class-based view (`View`, `TemplateView`, `ListView`) | Standard patterns (list, detail, create, update, delete) |
| `CreateView` / `UpdateView` | Form-based create and update with Django forms |
| `ListView` / `DetailView` | Standard read views |
| `DeleteView` | Standard delete with confirmation |
| DRF `APIView` or `ViewSet` | JSON APIs consumed by React or other frontends |

```text
[ ] View type chosen based on the pattern — not personal preference
[ ] Not writing a class-based view that just reimplements a built-in generic view
[ ] Not writing a function view for something with many methods (use CBV)
```

---

# 3. Authentication & Authorization

```text
[ ] Every protected view uses @login_required or LoginRequiredMixin — no exceptions
[ ] Role/permission check performed for sensitive operations:
  — User is member of the correct workspace/org
  — User has the required role (admin, owner, etc.)
[ ] Permission read from database — not from session payload or request parameter
[ ] Anonymous users handled: redirect to login or return 401 (for APIs)
[ ] DRF views use permission_classes = [IsAuthenticated] or custom permission class
[ ] Object-level permission checked: user has access to THIS specific object (not just the type)
  — e.g., cannot view another user's project just by guessing the ID
[ ] No role or user ID accepted from request.POST or request.GET for permission decisions
```

---

# 4. Form & Input Validation

```text
[ ] All user input flows through a Django Form or Serializer — not accessed directly from request.POST
[ ] form.is_valid() called before accessing any cleaned data
[ ] cleaned_data used for all values — not raw request.POST.get()
[ ] Validation errors returned to user with clear field-level messages
[ ] Required fields defined in the form — not validated inline in the view
[ ] Custom validators live in validators.py — not inline in forms or views
[ ] For REST APIs: serializer.is_valid(raise_exception=True) used
[ ] No manual type casting from strings to integers/dates in the view — form handles this
```

---

# 5. Business Logic Placement

This is the most important rule for Django views.

```text
[ ] View does NOT contain business logic — only HTTP handling
[ ] Business logic lives in services.py or a service class
[ ] View calls service: result = ProjectService.create_project(...)
[ ] No database queries written directly in the view (use service or manager)
[ ] No slug generation, price calculation, or email sending in the view
[ ] No multi-step operations inline in the view (extract to service)
```

**Good pattern:**

```python
# views.py
def create_project(request):
    form = ProjectForm(request.POST)
    if not form.is_valid():
        return render(request, 'projects/create.html', {'form': form})

    project = ProjectService.create_project(
        workspace=request.workspace,
        name=form.cleaned_data['name'],
        created_by=request.user
    )
    messages.success(request, 'Project created successfully.')
    return redirect('projects:detail', pk=project.pk)
```

**Bad pattern (logic in view):**

```python
# views.py — DO NOT DO THIS
def create_project(request):
    name = request.POST.get('name')
    slug = name.lower().replace(' ', '-')
    if Project.objects.filter(slug=slug).exists():
        # ...
    project = Project.objects.create(name=name, slug=slug, ...)
    send_email(request.user, 'project_created', project)
    AuditLog.objects.create(...)
    return redirect(...)
```

---

# 6. Response Handling

## For HTML Views (Template)

```text
[ ] Success: redirect using redirect() after POST (POST → redirect → GET pattern)
[ ] Never return a rendered template directly after a successful POST (causes re-submit on refresh)
[ ] Failure: render form with errors (not redirect — user loses error context)
[ ] Success messages use Django messages framework (messages.success)
[ ] Error messages use messages.error or form errors — not hardcoded in template
```

## For REST API Views (JSON)

```text
[ ] Success: return Response(serializer.data, status=201) for create
[ ] Success: return Response(serializer.data, status=200) for read/update
[ ] Delete: return Response(status=204)
[ ] Validation error: return Response(serializer.errors, status=400)
[ ] Auth error: return Response({'error': 'AUTH_REQUIRED'}, status=401)
[ ] Permission error: return Response({'error': 'FORBIDDEN'}, status=403)
[ ] Not found: return Response({'error': 'NOT_FOUND'}, status=404)
```

---

# 7. Error Handling

```text
[ ] No unhandled exceptions that return a 500 with a Django debug page in production
[ ] Custom exception handlers defined for DRF APIs (global handler, not per-view)
[ ] Business logic exceptions caught and mapped to HTTP responses:
  — NotFoundException → 404
  — PermissionDenied → 403
  — ConflictError → 409
  — ValidationError → 400
[ ] Django's Http404 or get_object_or_404() used for not-found cases
[ ] User-facing error messages do not expose internal details (no stack traces, no SQL errors)
[ ] Errors logged server-side with enough context for debugging
```

---

# 8. Reusability & Code Organization

```text
[ ] Permission logic extracted to a mixin or decorator if used in multiple views
[ ] Common query patterns in managers.py (e.g., Project.objects.active())
[ ] Reusable form used across create and update views (same form class)
[ ] No copy-pasted validation logic between views — extracted to form or validator
[ ] URL names defined clearly: projects:list, projects:detail, projects:create
[ ] No business logic in URL conf (urls.py stays clean)
```

---

# 9. Testing Requirements

```text
[ ] Test: successful request returns expected response
[ ] Test: unauthenticated request redirects to login (or 401 for API)
[ ] Test: unauthorized user (wrong workspace) gets 403
[ ] Test: invalid input returns form errors (or 400 for API)
[ ] Test: not found ID returns 404
[ ] Test: POST → redirect → correct destination (for HTML views)
[ ] Test: business logic was called with correct arguments (mock service layer)
[ ] Test: duplicate/conflict case handled correctly
[ ] Tests use Django TestCase or pytest-django
[ ] Test database does not share state between tests (use setUp/tearDown)
```

---

# 10. Django View Checklist — Before Marking Done

```text
VIEW TYPE
[ ] Correct view type chosen (function / class-based / generic / DRF)

AUTH & PERMISSIONS
[ ] @login_required or LoginRequiredMixin applied
[ ] Role/permission read from database — not from request
[ ] Object-level permission checked (user has access to THIS object)
[ ] Anonymous users handled with redirect or 401

FORM/VALIDATION
[ ] All input through Django Form or Serializer
[ ] form.is_valid() / serializer.is_valid() called before accessing data
[ ] cleaned_data used — not raw request.POST
[ ] Required fields defined in form — not validated in view

BUSINESS LOGIC
[ ] No business logic in view — extracted to services.py
[ ] No database queries directly in view — use service or manager
[ ] No email sending, slug generation, or calculations in view

RESPONSE
[ ] HTML views: POST → redirect (never render template after successful POST)
[ ] API views: correct HTTP status codes (200/201/204/400/401/403/404)
[ ] Errors shown to user — not only logged
[ ] Success messages via Django messages framework or API response

ERROR HANDLING
[ ] Business exceptions mapped to HTTP responses
[ ] get_object_or_404 used for not-found cases
[ ] No stack traces exposed to users
[ ] Errors logged server-side

REUSABILITY
[ ] No copy-pasted logic from other views
[ ] Common queries in managers.py
[ ] Permission logic extracted if reused

TESTING
[ ] Success case tested
[ ] Auth failure tested
[ ] Permission failure tested
[ ] Validation failure tested
[ ] Not found tested
```

---

## Practice Task

Apply what you learned by building a real view with a reusable permission mixin, form validation, and a service layer.

**→ [Task 01: Build a Project List & Create View](../../tasks/django/01-project-list-create-view.md)**

Covers: LoginRequiredMixin, WorkspaceMemberMixin (write once, use everywhere), ProjectForm with cross-field date validation, ProjectService.create_project() with slug + uniqueness + audit log, POST→redirect→GET pattern, PermissionDenied handling.
