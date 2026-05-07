# Code Reusability Standards

Core rule:

> Before you build anything, ask: "Does this already exist, or will it exist again?" If the answer is yes to either — build it once, build it properly, and reuse it everywhere.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-reusability-check--before-you-start-building) | Reusability Check — Before You Build |
| [2](#2-the-reusability-decision-rule) | The Decision Rule |
| [3](#3-platform-specific-reusability-guide) | Platform-Specific Guide (React / WeWeb / Xano / Django / Supabase) |
| [4](#4-reusability-naming-conventions) | Naming Conventions |
| [5](#5-reusability-anti-patterns) | Anti-Patterns |
| [6](#6-when-not-to-over-abstract) | When NOT to Over-Abstract |
| [7](#7-reusability-checklist--before-marking-done) | Checklist Before Marking Done |

---

# 1. The Reusability Check — Before You Start Building

Run this check on every new task before writing a single line:

```text
[ ] Does this component/function/logic already exist in the codebase?
[ ] Is this used in more than one place right now?
[ ] Will this logically be needed in other places in the future?
[ ] Am I copy-pasting code from somewhere else?
[ ] Am I building the same form/table/modal that already exists on another page?
[ ] Am I writing the same validation logic that exists in another file?
[ ] Am I calling the same API in the same way somewhere else?
```

**If any answer is yes — stop. Extract before you build.**

---

# 2. The Reusability Decision Rule

| Situation | Decision |
|-----------|----------|
| Used in 2 or more places right now | Must be reusable |
| Will clearly be used again soon | Build reusable from the start |
| Complex logic with a clear purpose | Extract to a named function |
| You're copy-pasting from another file | Stop. Extract first. |
| Identical UI component used on two pages | Shared component |
| Same API call pattern in multiple files | Shared service/hook |
| Same validation rules in Add and Edit | Shared validation schema |
| Same error handling logic repeated | Shared error handler |
| Three or more similar lines doing the same pattern | Extract to function |

**Three or more similar lines = automatic extraction.**

---

# 3. Platform-Specific Reusability Guide

## React / Next.js

### What to Make Reusable

| Item | Where It Lives | Example |
|------|---------------|---------|
| Shared UI component | `components/ui/` | `Button.tsx`, `Input.tsx`, `Modal.tsx` |
| Feature-specific component | `components/[feature]/` | `ProjectCard.tsx`, `MemberList.tsx` |
| Shared form | `components/forms/` | `ProjectForm.tsx` (used in Add + Edit) |
| Custom hook | `hooks/` | `useProjects.ts`, `useAuth.ts` |
| API call logic | `services/` or `lib/` | `projects.service.ts` |
| Validation schema | `validations/` | `project.schema.ts` |
| Error handler | `utils/api-error-handler.ts` | Used across all API calls |
| Constants and enums | `constants/` | `roles.ts`, `status.ts` |
| Type definitions | `types/` | `Project.ts`, `User.ts` |

### Folder Structure

```text
src/
  components/
    ui/
      Button.tsx
      Input.tsx
      Modal.tsx
      Table.tsx
      Badge.tsx
    forms/
      ProjectForm.tsx       ← Used in Add Project AND Edit Project
      MemberInviteForm.tsx
    modals/
      DeleteConfirmModal.tsx
      ConfirmActionModal.tsx

  hooks/
    useProjects.ts
    useWorkspace.ts
    useAuth.ts

  services/
    projects.service.ts
    members.service.ts
    auth.service.ts

  validations/
    project.schema.ts
    member.schema.ts

  utils/
    api-error-handler.ts
    date-formatter.ts
    string-utils.ts

  constants/
    roles.ts
    status.ts
    routes.ts

  types/
    project.ts
    user.ts
    workspace.ts
```

### Anti-Patterns

```text
× Same Button component defined in 5 different files.
× ProjectForm duplicated in AddProject.tsx and EditProject.tsx with different validation.
× fetch('/api/projects') called directly in 8 different components.
× Same error handling logic copy-pasted across every form submit handler.
× Validation rules written inline in each form component.
```

---

## WeWeb

### What to Make Reusable

| Item | Where It Lives |
|------|---------------|
| Header / Navbar | Reusable Element |
| Footer | Reusable Element |
| Sidebar | Reusable Element |
| Card component | Reusable Element |
| Form section | Reusable Element |
| Modal / Popup | Reusable Element |
| API call | Named Collection or Action |
| Global variables | WeWeb Global Variables |
| Color/typography | WeWeb Global Design Styles |

### WeWeb Reusability Rules

```text
[ ] Header and footer are Reusable Elements — not rebuilt on every page.
[ ] Global colors and typography defined in WeWeb's design system — not hardcoded per element.
[ ] API collections named clearly and reused across pages that need the same data.
[ ] Shared modals (delete confirmation, invite, etc.) are Reusable Elements.
[ ] Form submission workflows are named clearly and reused where the same form appears.
[ ] Page-specific spacing uses global spacing values — not random custom values.
```

### Naming Reusable Elements

| Good | Bad |
|------|-----|
| `Card — Project` | `Group 4` |
| `Modal — Delete Confirmation` | `Popup copy` |
| `Header — Authenticated` | `Header thing` |
| `Form — Invite Member` | `Section 2` |

---

## Xano

### What to Make Reusable

| Item | Where It Lives |
|------|---------------|
| Auth/permission check | Xano Function: `check_workspace_permission` |
| Slug generator | Xano Function: `generate_slug` |
| Email sender | Xano Function: `send_email` |
| Date formatter | Xano Function: `format_date` |
| Price calculator | Xano Function: `calculate_invoice_total` |
| Pagination logic | Xano Function: `paginate_query` |
| Webhook signature check | Xano Function: `verify_webhook_signature` |

### Rule

If you write the same 3+ steps in two different Xano endpoints — extract to a function.

---

## Django

### What to Make Reusable

| Item | Where It Lives | Example |
|------|---------------|---------|
| Business logic | `services.py` or `managers.py` | `ProjectService.create_project()` |
| Reusable queries | `managers.py` | `Project.objects.active()` |
| Permission mixin | `mixins.py` | `WorkspaceMemberMixin` |
| Reusable form | `forms.py` | `ProjectForm` (used in create + update views) |
| Template component | `templates/components/` | `card.html`, `form_field.html` |
| Template filter | `templatetags/` | `{{ date|format_date }}` |
| Shared validation | `validators.py` | `validate_phone_number()` |
| Utility functions | `utils.py` | `generate_slug()`, `send_notification()` |

### Fat Models, Thin Views Rule

```text
View: Handle the HTTP request and response. Not more.
Model/Service: Handle the business logic.
Form/Serializer: Handle validation.
Template: Handle display only.
```

Bad — business logic in view:

```python
def create_project(request):
    name = request.POST.get('name')
    slug = name.lower().replace(' ', '-')
    existing = Project.objects.filter(slug=slug, workspace=workspace).first()
    if existing:
        ...
    Project.objects.create(name=name, slug=slug, workspace=workspace, created_by=request.user)
    send_email(...)
    log_audit(...)
```

Good — logic extracted to service:

```python
# services/project_service.py
class ProjectService:
    @staticmethod
    def create_project(workspace, name, created_by):
        slug = generate_slug(name)
        if Project.objects.filter(slug=slug, workspace=workspace).exists():
            raise ConflictError('Project name already exists.')
        project = Project.objects.create(name=name, slug=slug, workspace=workspace, created_by=created_by)
        send_notification(created_by, 'project_created', project)
        AuditLog.log('project.created', actor=created_by, entity=project)
        return project

# views.py
def create_project(request):
    form = ProjectForm(request.POST)
    if not form.is_valid():
        return error_response(form.errors)
    project = ProjectService.create_project(
        workspace=request.workspace,
        name=form.cleaned_data['name'],
        created_by=request.user
    )
    return success_response(project)
```

---

## Supabase

### What to Make Reusable

| Item | Where It Lives |
|------|---------------|
| Complex multi-step operations | RPC / Postgres Function |
| Repeated RLS patterns | Consistent policy templates |
| Common query patterns | Postgres Views or Functions |
| Validation logic | DB check constraints + RPC |
| Audit logging | DB trigger |
| Workspace membership check | Consistent subquery pattern |

### Reusable RPC Pattern

If the same business operation happens in multiple places (e.g., "create project + create default task + create activity log"), put it in one RPC function — not in three separate frontend calls.

Bad:

```ts
// Frontend makes 3 sequential calls
await supabase.from('projects').insert(...)
await supabase.from('tasks').insert(...)
await supabase.from('activity_logs').insert(...)
```

Good:

```ts
// Frontend makes 1 call, backend handles the rest
await supabase.rpc('create_project_with_defaults', { p_workspace_id, p_name })
```

---

# 4. Reusability Naming Conventions

Names must be clear enough that another team member knows what it does without reading the code.

## Functions

```text
verb_noun or getVerb (for getters)
```

| Good | Bad |
|------|-----|
| `formatDate(date)` | `dateHelper(d)` |
| `generateSlug(name)` | `makeSlug(n)` |
| `validatePhoneNumber(phone)` | `checkPhone(p)` |
| `calculateInvoiceTotal(items)` | `doCalc(stuff)` |
| `sendEmailNotification(user, template)` | `sendEmail(u, t)` |

## Components (React)

```text
PascalCase, named by what it IS
```

| Good | Bad |
|------|-----|
| `ProjectCard` | `Card2` |
| `DeleteConfirmModal` | `Popup` |
| `MemberInviteForm` | `FormComponent` |
| `WorkspaceSelector` | `Dropdown` |

## Hooks (React)

```text
useNoun or useVerbNoun
```

| Good | Bad |
|------|-----|
| `useProjects` | `useData` |
| `useWorkspaceMembers` | `useMembers2` |
| `useProjectForm` | `useForm` |

## Services (React/Django)

```text
noun.service.ts / NounService
```

| Good | Bad |
|------|-----|
| `projects.service.ts` | `api.ts` |
| `ProjectService` | `Helper` |

---

# 5. Reusability Anti-Patterns

These are the most common violations. Watch for all of them.

| Anti-Pattern | Impact | Fix |
|-------------|--------|-----|
| Same component markup in 5 files | One change breaks inconsistently | Extract to shared component |
| Add form and Edit form have different validation | Users can bypass validation on edit | Shared validation schema |
| Same fetch call to `/projects` in 8 components | Changes require editing 8 files | Shared service/hook |
| Business logic inline in a view function | Cannot reuse, cannot test, grows forever | Extract to service |
| Same 10-line error handler in every form | One fix needed everywhere | Shared `handleApiError()` util |
| Template section copy-pasted to 4 pages | Update one, forget the rest | `{% include %}` component |
| Same Xano permission check steps in 12 endpoints | Change once, must update 12 | Xano reusable function |
| WeWeb header rebuilt on every page | Style changes require page-by-page updates | Reusable Element |
| Django utility function defined in `views.py` | Cannot be used by other views/apps | Move to `utils.py` |
| Hardcoded string `"admin"` in 20 places | Rename role → 20 places to update | Constants file |

---

# 6. When NOT to Over-Abstract

Reusability is good — over-abstraction creates complexity.

**Do not make something reusable if:**

```text
× It is only used in one place and has no clear future use.
× Making it "generic" requires more complex parameters than just duplicating it.
× The abstraction would make the code harder to read than the original.
× It is a one-time migration script or throwaway utility.
```

**The rule:** Three or more identical uses = extract. One use = keep it where it is.

---

# 7. Reusability Checklist — Before Marking Done

```text
[ ] Checked if this component/function already exists before building.
[ ] Checked all pages that use the same entity/flow for consistency.
[ ] Add and Edit forms share the same component and validation schema.
[ ] No logic copy-pasted from another file (extracted if needed).
[ ] Any function used in 2+ places has been extracted and named properly.
[ ] Shared utilities placed in the correct shared folder (utils/, services/, etc.).
[ ] No hardcoded strings for roles, statuses, or routes (moved to constants).
[ ] New reusable component/function is named clearly so others can discover it.
[ ] No inline styles or logic that should be in shared CSS/component.
```
