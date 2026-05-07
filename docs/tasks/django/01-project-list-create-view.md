# Task: Build a Project List & Create View

**Platform:** Django
**Covers:** [New View Checklist](../../checklists/django/new-view.md) · [Backend-First Logic](../../standards/backend-first-logic.md) · [Code Reusability Standards](../../standards/code-reusability-standards.md)

---

## Scenario

You are building the Projects section for **WorkFlow** in Django. Workspace members need to see a list of their workspace's active projects and be able to create new ones. Business logic — uniqueness checking, slug generation, audit logging — must live in `services/project_service.py`, not in the view.

---

## What to Build

Two views and their URLs:

| View | URL | Method |
|------|-----|--------|
| `ProjectListView` | `/workspaces/<workspace_id>/projects/` | GET |
| `ProjectCreateView` | `/workspaces/<workspace_id>/projects/create/` | GET + POST |

---

## Models (assume already exist)

```python
# projects/models.py
class Project(models.Model):
    workspace    = models.ForeignKey('workspaces.Workspace', on_delete=models.CASCADE)
    created_by   = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.RESTRICT)
    name         = models.CharField(max_length=100)
    slug         = models.SlugField(max_length=120)
    description  = models.TextField(blank=True)
    status       = models.CharField(max_length=20, default='active')
    start_date   = models.DateField(null=True, blank=True)
    end_date     = models.DateField(null=True, blank=True)
    created_at   = models.DateTimeField(auto_now_add=True)
    updated_at   = models.DateTimeField(auto_now=True)
    deleted_at   = models.DateTimeField(null=True, blank=True)

    class Meta:
        unique_together = [('workspace', 'slug')]
```

---

## Part 1 — ProjectListView

### Requirements

- Login required
- User must be a member of the workspace (not just any logged-in user)
- List only active projects (`deleted_at__isnull=True`) for the workspace
- Ordered by `created_at` descending
- Pass projects and workspace to the template context

### Implementation Notes

- Use `LoginRequiredMixin`
- Write a `WorkspaceMemberMixin` in `workspaces/mixins.py` that:
  - Gets the workspace from `workspace_id` URL kwarg
  - Checks the current user is a member
  - Raises `PermissionDenied` if not
  - Adds `self.workspace` for use in the view
- Use this mixin on both views
- Query in `get_queryset()` — not in `get()` or `get_context_data()`

---

## Part 2 — ProjectCreateView

### Form: `ProjectForm`

Create in `projects/forms.py`:

```python
class ProjectForm(forms.ModelForm):
    class Meta:
        model = Project
        fields = ['name', 'description', 'start_date', 'end_date']
```

Validation rules to add:
- `name`: required, min 3 chars, max 100 chars, stripped of whitespace
- `description`: optional, max 500 chars
- `end_date`: if both start and end provided, end must be after start (cross-field validation in `clean()`)

### Service: `ProjectService.create_project()`

Create in `projects/services/project_service.py`:

```python
class ProjectService:
    @staticmethod
    def create_project(workspace, name, created_by, description='', start_date=None, end_date=None):
        # 1. Generate slug from name
        # 2. Check uniqueness in this workspace (raise ConflictError if duplicate)
        # 3. Create the Project record
        # 4. Create a default TaskList named "General" for this project
        # 5. Create an AuditLog entry: action='project.created'
        # 6. Return the project
        ...
```

The view calls this service — it does NOT do these steps inline.

### View Requirements

- GET: render empty `ProjectForm`
- POST: validate form → call service → redirect on success → re-render with errors on failure
- On success: redirect to the project detail page with a success message (`messages.success`)
- On failure: render the form with validation errors — form stays populated
- Do NOT redirect on POST failure — user should see their errors without losing their input
- Do NOT put slug generation, uniqueness checking, or audit logging in the view

### URL Pattern

```python
# projects/urls.py
app_name = 'projects'
urlpatterns = [
    path('', ProjectListView.as_view(), name='list'),
    path('create/', ProjectCreateView.as_view(), name='create'),
]

# workspaces/urls.py
path('workspaces/<int:workspace_id>/projects/', include('projects.urls')),
```

---

## What You Should NOT Do

- Do not write slug generation in the view — put it in `ProjectService`
- Do not query the database directly in the view — use the service and model manager
- Do not access `request.POST.get('name')` directly — always use `form.cleaned_data`
- Do not redirect after a failed POST — render the form with errors
- Do not render a template response after a successful POST — always redirect (POST → redirect → GET)
- Do not put the workspace membership check inline in each view — extract to `WorkspaceMemberMixin`
- Do not duplicate `WorkspaceMemberMixin` — it must be defined once and imported everywhere it's needed

---

## Checklist to Run When Done

Use the [New View Checklist](../../checklists/django/new-view.md#10-django-view-checklist--before-marking-done).

---

## Done When

```text
STRUCTURE
[ ] WorkspaceMemberMixin in workspaces/mixins.py (not inline in views)
[ ] ProjectForm in projects/forms.py
[ ] ProjectService.create_project() in projects/services/project_service.py
[ ] Both views use LoginRequiredMixin + WorkspaceMemberMixin
[ ] URL patterns defined with app_name = 'projects'

LIST VIEW
[ ] Only active projects shown (deleted_at__isnull=True)
[ ] Scoped to correct workspace
[ ] Ordered by created_at descending
[ ] Non-member gets PermissionDenied (403)

CREATE VIEW
[ ] GET: empty form rendered
[ ] POST valid: calls ProjectService.create_project() → redirect to detail
[ ] POST invalid: re-renders form with errors, form stays populated
[ ] Success: messages.success with "Project created." message
[ ] Form validation: name min 3, max 100; end > start; description max 500
[ ] created_by = request.user (not from POST data)
[ ] Slug generated in service (not in view)
[ ] Duplicate name → ConflictError caught → form error shown to user

PERMISSIONS
[ ] @login_required / LoginRequiredMixin on both views
[ ] WorkspaceMemberMixin checks membership for both views
[ ] Non-member trying to access → 403

TESTING
[ ] GET list: member sees projects, non-member gets 403
[ ] GET create: member sees form, non-member gets 403
[ ] POST create valid: project created, redirect to detail, success message
[ ] POST create invalid (empty name): form re-rendered with error
[ ] POST create duplicate name: form re-rendered with "name already exists" error
[ ] POST create (unauthenticated): redirect to login
```
