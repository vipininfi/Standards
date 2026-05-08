# Task — DRF API Views: Projects Endpoint

**Platform:** Django REST Framework
**Checklist to Run:** [DRF API View Checklist](../../checklists/django/api-view.md)
**Standard:** [Backend-First Logic Standard](../../standards/backend-first-logic.md)

---

## Scenario

You need to build the REST API endpoints for the Project resource. These endpoints will be consumed by a React frontend. The endpoints must be authenticated, permission-checked, delegate to `ProjectService`, and return consistent JSON responses.

---

## What to Build

1. `ProjectListCreateView` — `GET /api/v1/workspaces/{workspace_id}/projects/`
2. `ProjectDetailView` — `GET /api/v1/workspaces/{workspace_id}/projects/{project_id}/`
3. `ProjectArchiveView` — `POST /api/v1/workspaces/{workspace_id}/projects/{project_id}/archive/`
4. A custom `IsWorkspaceMember` permission class
5. URL configuration

---

## Requirements

### Permission Class: `IsWorkspaceMember`

```python
# File: permissions.py
class IsWorkspaceMember(BasePermission):
    """
    Allows access only to authenticated users who are active members
    of the workspace specified in the URL kwarg 'workspace_id'.
    """
    def has_permission(self, request, view):
        workspace_id = view.kwargs.get('workspace_id')
        return WorkspaceMember.objects.filter(
            workspace_id=workspace_id,
            user=request.user,
            status='active'
        ).exists()
```

### `ProjectListCreateView`

**GET** — List projects in a workspace:
- Filter: `?status=active` (optional)
- Sort: `?ordering=name` or `?ordering=-created_at` (default: `-created_at`)
- Pagination: 25 per page
- Response: `{ "count": 143, "next": "...", "previous": "...", "results": [...] }`

**POST** — Create a new project:
- Request body: `{ "name": "...", "description": "...", "start_date": "...", "end_date": "..." }`
- Calls `ProjectService.create_project()`
- Success: 201 with the created project
- Duplicate name: 409 Conflict: `{ "error": "DUPLICATE", "message": "A project with this name already exists." }`

### `ProjectDetailView`

**GET** — Get a single project:
- Returns full project detail
- 404 if project does not exist in this workspace

### `ProjectArchiveView`

**POST** — Archive a project:
- Only workspace admins/owners can call this endpoint (add `IsWorkspaceAdmin` or check role in view)
- Calls `ProjectService.archive_project()`
- Success: 200 with the updated project
- Already archived: 409 Conflict
- Non-admin calling: 403 Forbidden

### Serializers

**ProjectSerializer** (read):
```python
fields = ['id', 'name', 'description', 'status', 'start_date', 'end_date',
          'created_by', 'created_at', 'updated_at']
read_only_fields = ['id', 'created_by', 'created_at', 'updated_at', 'status']
```

**ProjectWriteSerializer** (write — for POST):
```python
fields = ['name', 'description', 'start_date', 'end_date']
# Validates: name required, max 100 chars, end_date > start_date
```

### URL Configuration

```python
# urls.py
urlpatterns = [
    path('workspaces/<uuid:workspace_id>/projects/',
         ProjectListCreateView.as_view(), name='project-list'),
    path('workspaces/<uuid:workspace_id>/projects/<uuid:project_id>/',
         ProjectDetailView.as_view(), name='project-detail'),
    path('workspaces/<uuid:workspace_id>/projects/<uuid:project_id>/archive/',
         ProjectArchiveView.as_view(), name='project-archive'),
]
```

---

## Error Response Format

All error responses must follow:

```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable description."
}
```

| Situation | Status | Error Code |
|-----------|--------|-----------|
| Not authenticated | 401 | `AUTH_REQUIRED` |
| Not a workspace member | 403 | `FORBIDDEN` |
| Not an admin (for archive) | 403 | `FORBIDDEN` |
| Project not found | 404 | `NOT_FOUND` |
| Duplicate project name | 409 | `DUPLICATE` |
| Project already archived | 409 | `BUSINESS_RULE_VIOLATION` |
| Validation error | 400 | `VALIDATION_ERROR` |

---

## What You Should NOT Do

```text
× Put business logic in the view (no Project.objects.create() in the view body)
× Skip the IsWorkspaceMember permission class — do not rely on object-level filtering alone
× Return 200 for a created resource — use 201
× Return 200 for a conflict — use 409
× Use fields = '__all__' in any serializer
× Return all projects without pagination
× Not test the 403 case for archive (non-admin calling it)
```

---

## Checklist to Run

Before marking done, run: [DRF API View Checklist](../../checklists/django/api-view.md)

---

## Done When

```text
[ ] IsWorkspaceMember permission class queries the database for membership
[ ] ProjectListCreateView: GET returns paginated list, filters by status, sorts correctly
[ ] ProjectListCreateView: POST creates via ProjectService, returns 201
[ ] ProjectListCreateView: POST duplicate → 409 with correct error format
[ ] ProjectDetailView: GET returns single project
[ ] ProjectDetailView: GET non-existent project → 404
[ ] ProjectArchiveView: POST archives via ProjectService, returns 200
[ ] ProjectArchiveView: POST non-admin → 403
[ ] ProjectArchiveView: POST already archived → 409
[ ] URL patterns named and follow kebab-case convention
[ ] All views have permission_classes = [IsAuthenticated, IsWorkspaceMember]
[ ] Serializers use explicit fields — no __all__
[ ] All test cases pass: auth, permissions, CRUD, errors, pagination
```
