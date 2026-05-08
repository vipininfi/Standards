# Checklist — Django Model

> Run this checklist every time you create or significantly modify a Django model.

**Standard:** [Backend-First Logic Standard](../../standards/backend-first-logic.md)

---

## 1. Field Definitions

```text
[ ] Every field has the correct type (CharField, TextField, IntegerField, etc.)
[ ] CharField has a max_length set — never omitted
[ ] TextField used for long text — not CharField with a large max_length
[ ] DateTimeField used with auto_now_add=True for created_at
[ ] DateTimeField used with auto_now=True for updated_at
[ ] UUIDField used for external-facing IDs (not sequential integers in public APIs)
[ ] DecimalField used for money — not FloatField (avoid floating point errors)
[ ] BooleanField has a default value (True or False — never null=True on a boolean)
[ ] Choices: defined as class-level constants or TextChoices — not raw strings inline
```

---

## 2. Relationships

```text
[ ] ForeignKey has on_delete behavior set explicitly — never omitted:
    — CASCADE: delete related records when parent deleted (tasks when project deleted)
    — SET_NULL: set to null when parent deleted (requires null=True, blank=True)
    — PROTECT: prevent deleting parent if children exist (for important linked data)
    — SET_DEFAULT: set to a default value (rare)
[ ] related_name set on all ForeignKey and ManyToManyField relationships
    (avoids Django's auto-generated _set names in queries)
[ ] ManyToManyField: through model used when the relationship has extra data
[ ] GenericForeignKey: used only when truly necessary (adds complexity — prefer explicit FK)
```

---

## 3. Null and Blank

```text
[ ] null=True used only for fields where NULL is meaningful in the database
[ ] blank=True used for fields that can be empty in forms (separate from null)
[ ] CharField: avoid null=True — use blank=True + default="" instead (empty string is cleaner)
[ ] DateTimeField: null=True, blank=True acceptable for optional dates (e.g., deleted_at)
[ ] ForeignKey: null=True if the relationship is optional
[ ] BooleanField: never null=True — always has a default
```

---

## 4. Indexes and Constraints

```text
[ ] db_index=True on columns frequently used in WHERE, ORDER BY, or JOIN
[ ] Unique constraints using UniqueConstraint in Meta.constraints (not just unique=True on a field)
[ ] Composite unique constraints documented: e.g., unique together on (workspace, name) where deleted_at IS NULL
[ ] Check constraints using CheckConstraint: e.g., end_date > start_date
[ ] Indexes on ForeignKey fields (Django adds these automatically, but verify)
[ ] Full-text search: GinIndex or GistIndex if needed
```

---

## 5. Meta Class

```text
[ ] class Meta defined with:
    [ ] ordering: default sort order defined (e.g., ordering = ['-created_at'])
    [ ] verbose_name and verbose_name_plural set
    [ ] db_table set if the table name should differ from Django's default
    [ ] constraints list (UniqueConstraint, CheckConstraint)
    [ ] indexes list (Index)
```

---

## 6. Model Methods

```text
[ ] __str__ method defined — returns a human-readable representation
[ ] No business logic in __str__ (no database queries)
[ ] Business logic lives in services.py — not in model methods
[ ] Model methods acceptable for: property-style data access, simple computed properties
[ ] @property decorator used for computed fields not stored in the DB
[ ] No API calls or file operations in model methods
[ ] No direct database writes inside save() override (use a signal or service instead)
```

---

## 7. Soft Delete

```text
IF SOFT DELETE IS REQUIRED:
[ ] deleted_at = DateTimeField(null=True, blank=True, default=None) on the model
[ ] Custom manager filters out deleted records by default: .filter(deleted_at__isnull=True)
[ ] Second manager (e.g., all_objects) provides access to all records including deleted
[ ] Soft delete: set deleted_at = now() — do not call .delete()
[ ] Hard delete available via all_objects.filter().delete() for admin/cleanup only
[ ] Uniqueness constraints account for soft delete: unique on (workspace, name) where deleted_at IS NULL
```

---

## 8. Migration

```text
[ ] Migration generated and reviewed before running: python manage.py makemigrations
[ ] Migration file reviewed — no unexpected changes
[ ] Migration is reversible: no RunPython without a reverse function (or documented as atomic=False)
[ ] Data migrations: use RunPython with a reverse function that undoes the changes
[ ] No raw SQL in migrations without a comment explaining why
[ ] Migration file committed with the model changes (not applied without committing)
```

---

## 9. Testing

```text
[ ] Model can be created with valid data (factory or fixture)
[ ] Field constraints tested: null, blank, max_length violations raise correct errors
[ ] ForeignKey on_delete behavior tested (CASCADE, PROTECT, SET_NULL)
[ ] Unique constraints tested: duplicate raises IntegrityError
[ ] Check constraints tested: violation raises IntegrityError
[ ] Soft delete tested: deleted records excluded from default queryset
[ ] __str__ tested: returns expected string
```

---

## Done When

```text
[ ] All fields have correct types, null/blank settings
[ ] ForeignKey on_delete set on all relationships
[ ] related_names set on all FK and M2M fields
[ ] Indexes defined for frequently-queried columns
[ ] Unique and check constraints defined in Meta.constraints
[ ] class Meta has ordering, verbose_name, verbose_name_plural
[ ] __str__ returns a useful human-readable string
[ ] Migration generated and reviewed
[ ] Tests pass for field constraints, relationships, and constraints
```

---

## Practice Task

**→ [Model and Service Task](../../tasks/django/03-model-and-service.md)**
Create a Project model with all required fields, indexes, constraints, soft delete, and a ProjectService class with create and archive methods.
