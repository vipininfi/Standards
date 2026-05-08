# Checklist — Django REST Framework API View

> Run this checklist every time you build a DRF API view (APIView, ViewSet, or generic view).

**Standard:** [Backend-First Logic Standard](../../standards/backend-first-logic.md)

---

## 1. View Type Decision

| Use This | When |
|---------|------|
| `APIView` | Custom logic that doesn't map cleanly to CRUD |
| `generics.ListCreateAPIView` | Standard list + create endpoint |
| `generics.RetrieveUpdateDestroyAPIView` | Standard get + update + delete endpoint |
| `ModelViewSet` | Full CRUD with routing via `router.register()` |
| `ReadOnlyModelViewSet` | List + retrieve only |

```text
[ ] Correct view type chosen — not defaulting to APIView for everything
[ ] Generic views / ViewSets used when the logic is standard CRUD
[ ] APIView only for non-standard flows (multi-step, non-REST actions)
```

---

## 2. Authentication & Permissions

```text
[ ] Authentication class set (IsAuthenticated is the default for protected endpoints)
[ ] permission_classes set on the view — not relying on defaults alone
    — IsAuthenticated: logged-in users only
    — IsAdminUser: admin-only
    — Custom: e.g., IsWorkspaceMember, IsWorkspaceAdmin
[ ] Custom permission class written in permissions.py — not inline
[ ] Permission class checks the database for role (not just the user's JWT claim)
[ ] Every protected view explicitly declares permission_classes
```

---

## 3. Serializer

```text
[ ] serializer_class set on the view
[ ] Different serializers for read and write if the shapes differ:
    serializer_class = ProjectSerializer         (read — includes nested data)
    write_serializer_class = ProjectWriteSerializer  (write — flat fields only)
[ ] serializer.is_valid(raise_exception=True) used — not is_valid() with manual error check
[ ] validated_data passed to the service layer — not the raw request.data
```

---

## 4. Business Logic Placement

```text
[ ] View calls a service function — it does NOT contain business logic itself

CORRECT:
  def post(self, request):
      serializer = ProjectWriteSerializer(data=request.data)
      serializer.is_valid(raise_exception=True)
      project = ProjectService.create_project(
          workspace_id=..., actor=request.user, **serializer.validated_data
      )
      return Response(ProjectSerializer(project).data, status=201)

WRONG — logic in the view:
  def post(self, request):
      project = Project.objects.create(...)
      TaskList.objects.create(project=project, ...)

[ ] No ORM calls in the view body (except .get() or .filter() for object retrieval)
[ ] No email sending, file handling, or external API calls in the view
```

---

## 5. Response Format

```text
[ ] Success responses use correct HTTP status codes:
    — 200 OK: GET (retrieve or list)
    — 201 Created: POST (create)
    — 200 OK: PATCH/PUT (update — or 204 No Content if no body)
    — 204 No Content: DELETE
[ ] Error responses:
    — 400 Bad Request: validation error
    — 401 Unauthorized: authentication required
    — 403 Forbidden: authenticated but no permission
    — 404 Not Found: object does not exist
    — 409 Conflict: duplicate / uniqueness conflict
    — 500 Internal Server Error: unexpected (logged, generic message returned)
[ ] List responses use pagination — not returning all records in one response
[ ] Error response body follows consistent format:
    { "error": "DUPLICATE", "message": "A project with this name already exists." }
```

---

## 6. Filtering, Sorting, Pagination

```text
[ ] Filtering via query params — not hardcoded in the view
[ ] django-filter or manual filter logic in get_queryset()
[ ] Sorting via ?ordering= param (DRF OrderingFilter)
[ ] Default ordering defined in queryset (not random)
[ ] Pagination class set: PageNumberPagination or CursorPagination
[ ] Page size defined and reasonable (default 25 or 50)
[ ] Total count returned in the paginated response
```

---

## 7. Error Handling

```text
[ ] Serializer errors: raised automatically with raise_exception=True → 400 response
[ ] Object not found: get_object_or_404() or explicit try/except → 404 response
[ ] Permission errors: raised by DRF permission classes → 403 response
[ ] Unexpected errors: caught in try/except, logged, return 500 with generic message
[ ] Never expose stack traces or raw exception messages in the response
[ ] Business rule violations: raise a custom exception that maps to 409 or 422
```

---

## 8. URL Configuration

```text
[ ] URL pattern named: name='project-list', name='project-detail'
[ ] URL names follow kebab-case convention: project-list, project-detail, workspace-members
[ ] ViewSet registered with a router (not manually listing every URL)
[ ] API versioning considered: /api/v1/ prefix
[ ] URL parameters use meaningful names: <uuid:project_id> not <pk>
```

---

## 9. Testing

```text
[ ] Success case: correct data → correct response status and body
[ ] Unauthenticated request: 401 returned
[ ] Authenticated but wrong role: 403 returned
[ ] Validation failure: 400 returned with correct field errors
[ ] Object not found: 404 returned
[ ] Duplicate (if applicable): 409 returned
[ ] Pagination tested: page 1, page 2, last page, beyond last page
[ ] Filter tested: correct filtering behavior
[ ] Sort tested: correct ordering
```

---

## Done When

```text
[ ] Correct view type chosen
[ ] Authentication and permission classes explicitly set
[ ] Business logic delegated to a service function
[ ] Serializer validates input before service is called
[ ] Correct HTTP status codes for all responses
[ ] Pagination, filtering, sorting configured
[ ] All error cases return correct status and message
[ ] URL pattern named and follows convention
[ ] All test cases pass
```

---

## Practice Task

**→ [DRF API View Task](../../tasks/django/04-drf-api-view.md)**
Build a ProjectListCreateView and ProjectDetailView with authentication, permission checks, service delegation, and pagination.
