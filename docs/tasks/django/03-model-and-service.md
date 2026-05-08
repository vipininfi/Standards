# Task — Django Model + Service: `Project`

**Platform:** Django
**Checklist to Run:** [Django Model Checklist](../../checklists/django/model.md)
**Standard:** [Backend-First Logic Standard](../../standards/backend-first-logic.md)

---

## Scenario

You need to create the `Project` model for a workspace-based project management app. Projects belong to a workspace. Each project has a status, optional dates, and supports soft delete. Business logic for creating and archiving projects must live in a `ProjectService` class — not in the view.

---

## What to Build

1. The `Project` model in `models.py`
2. A custom manager that excludes soft-deleted records by default
3. A `ProjectService` class in `services.py`
4. A migration file
5. Unit tests for the model constraints and service methods

---

## Requirements

### Project Model Fields

| Field | Type | Rules |
|-------|------|-------|
| id | UUIDField | PK, default=uuid4, editable=False |
| workspace | ForeignKey → Workspace | ON DELETE CASCADE, related_name='projects' |
| name | CharField | max_length=100, cannot be blank |
| description | TextField | blank=True, default='' |
| status | CharField | max_length=20, choices: active/on-hold/archived, default='active' |
| start_date | DateField | null=True, blank=True |
| end_date | DateField | null=True, blank=True |
| created_by | ForeignKey → User | ON DELETE SET_NULL, null=True, related_name='created_projects' |
| created_at | DateTimeField | auto_now_add=True |
| updated_at | DateTimeField | auto_now=True |
| deleted_at | DateTimeField | null=True, blank=True, default=None |

### Model Constraints

```python
class Meta:
    ordering = ['-created_at']
    verbose_name = 'Project'
    verbose_name_plural = 'Projects'
    constraints = [
        # A workspace cannot have two active projects with the same name
        models.UniqueConstraint(
            fields=['workspace', 'name'],
            condition=models.Q(deleted_at__isnull=True),
            name='unique_active_project_name_per_workspace'
        ),
        # end_date must be after start_date
        models.CheckConstraint(
            check=(
                models.Q(end_date__isnull=True) |
                models.Q(start_date__isnull=True) |
                models.Q(end_date__gt=models.F('start_date'))
            ),
            name='project_end_date_after_start_date'
        ),
    ]
```

### Custom Manager

```python
class ProjectManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)

class Project(models.Model):
    objects = ProjectManager()        # default: excludes soft-deleted
    all_objects = models.Manager()    # includes soft-deleted
```

### ProjectService Methods

#### `create_project(workspace, name, created_by, description='', start_date=None, end_date=None)`

Rules:
- `name` cannot be blank or exceed 100 characters
- If both `end_date` and `start_date` are provided: `end_date` must be after `start_date`
- A project with the same name cannot already exist in the workspace (active projects only)
- On success: create the project AND create a default `TaskList` named "General" linked to it
- On duplicate name: raise `ValidationError("A project with this name already exists in this workspace.")`

#### `archive_project(project, actor)`

Rules:
- `project.status` must not already be `'archived'`
- On success: set `project.status = 'archived'`, save the project
- Write an entry to `AuditLog`: `action='project.archived'`, `entity_id=project.id`, `actor=actor`
- Caller's role must be checked by the view/permission layer before calling this method

---

## What You Should NOT Do

```text
× Put create_project or archive_project logic in the model's save() method
× Put business logic in the view (no Project.objects.create() in a view)
× Call Project.objects.create() directly from inside a view
× Use fields = '__all__' in any ModelForm or Serializer for this model
× Raise a generic Exception — always raise ValidationError or a named custom exception
× Skip the CheckConstraint and UniqueConstraint — they protect data at the DB level
× Forget to create the default TaskList inside create_project
```

---

## Checklist to Run

Before marking done, run: [Django Model Checklist](../../checklists/django/model.md)

---

## Done When

```text
[ ] Project model has all 11 fields with correct types and options
[ ] ForeignKey relationships have correct on_delete and related_name
[ ] UniqueConstraint for active projects with same name per workspace
[ ] CheckConstraint for end_date > start_date
[ ] class Meta has ordering, verbose_name, verbose_name_plural, constraints
[ ] Custom manager excludes soft-deleted records by default; all_objects includes them
[ ] __str__ returns a useful human-readable string (e.g., "{workspace.name} / {name}")
[ ] Migration generated, reviewed, and committed
[ ] ProjectService.create_project: validates name, checks uniqueness, creates project + TaskList
[ ] ProjectService.archive_project: validates not already archived, updates status, writes audit log
[ ] Unit tests cover: valid create, duplicate name error, invalid date range, archive, double-archive error
[ ] All tests pass
```
