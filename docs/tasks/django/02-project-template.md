# Task: Build the Project Detail Template

**Platform:** Django Templates
**Covers:** [Template Checklist](../../checklists/django/template.md) · [Django Template Standards](../../standards/django-template-standards.md)

---

## Scenario

Continuing from [Task 01](01-project-list-create-view.md), you will build the HTML templates for the Project List page and the Project Detail page. Every template must follow the 3-folder rule (CSS in `static/css/`, JS in `static/js/`, HTML in `templates/`), extend `base.html`, use zero inline styles or scripts, and pass any Django data to JavaScript only via `data-*` attributes.

---

## What to Build

Three template files and their supporting static files:

| Template | Path |
|----------|------|
| Project list page | `templates/projects/list.html` |
| Project create form page | `templates/projects/form.html` |
| Project detail page | `templates/projects/detail.html` |

Plus supporting static files:
- `static/css/pages/projects.css`
- `static/js/pages/project-detail.js`

---

## Part 1 — `templates/projects/list.html`

### Requirements

- Extends `base.html`
- Sets `{% block title %}Projects — WorkFlow{% endblock %}`
- Lists all projects passed in context as a grid of cards
- Each card shows: project name, status badge (color-coded), start date, number of tasks (if available)
- Cards link to the project detail page using `{% url 'projects:detail' workspace_id=workspace.id pk=project.id %}`
- An "Add Project" button links to the create page
- If no projects: show an empty state message ("No projects yet. Create your first project.")
- Status badges use CSS classes only — no inline `style="color: ..."` or `style="background: ..."`

### Status Badge Classes (defined in CSS, not inline)

```css
/* static/css/components/badge.css */
.badge-active   { background-color: var(--color-success); color: white; }
.badge-on-hold  { background-color: var(--color-warning); color: black; }
.badge-archived { background-color: var(--color-neutral); color: white; }
```

Use `{{ project.status|lower|slugify }}` to generate the class name dynamically.

---

## Part 2 — `templates/projects/form.html`

### Requirements

- Extends `base.html`
- Title block: "Add Project — WorkFlow" (or "Edit Project" in edit mode)
- Renders the `ProjectForm` passed in context
- Shows field-level errors below each field using `{{ form.field_name.errors }}`
- Shows non-field errors (from `form.non_field_errors()`) as a banner above the form
- Required fields show an asterisk (`*`) next to their label
- Submit button label: "Create Project"
- "Cancel" link goes back to the project list
- CSRF token included: `{% csrf_token %}`
- No `<style>` blocks — all styles in `static/css/pages/projects.css`
- No `<script>` blocks — JS in `static/js/pages/project-form.js` if needed

### Form Field Layout

Each field should follow this structure:

```html
<div class="form-field {% if form.name.errors %}has-error{% endif %}">
  <label for="{{ form.name.id_for_label }}">
    Project Name <span class="required-star">*</span>
  </label>
  {{ form.name }}
  {% if form.name.errors %}
    <p class="field-error">{{ form.name.errors.0 }}</p>
  {% endif %}
</div>
```

---

## Part 3 — `templates/projects/detail.html`

### Requirements

- Extends `base.html`
- Title block: `{{ project.name }} — WorkFlow`
- Shows all project fields: name, description, status, dates, created by, created at
- An "Edit Project" button — shown only to the project creator and workspace admins
- A "Archive Project" button — only for admins, triggers a JavaScript confirmation modal before submitting
- Project task list (if tasks are passed in context)

### JavaScript Interaction — Archive Confirmation

The Archive button must trigger a confirmation dialog before the form is submitted. The project name must be passed to the JavaScript — via a `data-*` attribute, NOT via string interpolation in a `<script>` block.

**Correct pattern:**

```html
<!-- In the template -->
<div
  id="project-actions"
  data-project-name="{{ project.name|escapejs }}"
  data-archive-url="{% url 'projects:archive' workspace_id=workspace.id pk=project.id %}"
>
  <button id="archive-btn" class="btn btn-danger">Archive Project</button>
</div>
```

```js
// static/js/pages/project-detail.js
const actions = document.getElementById('project-actions');
const projectName = actions.dataset.projectName;
const archiveUrl = actions.dataset.archiveUrl;

document.getElementById('archive-btn').addEventListener('click', () => {
  const confirmed = confirm(`Archive "${projectName}"? This cannot be undone.`);
  if (confirmed) {
    // Submit archive form or redirect
    window.location.href = archiveUrl;
  }
});
```

---

## Static Files to Create

### `static/css/pages/projects.css`

Must define styles for:
- Project card grid layout (responsive — 1 column mobile, 2 tablet, 3 desktop)
- Status badges (use CSS variables from `base.css`, not hardcoded colors)
- Empty state styling
- Form field layout (`form-field`, `has-error`, `field-error`, `required-star`)
- Detail page layout

**Must use CSS variables from `base.css` — no hardcoded colors or spacing values.**

### `static/js/pages/project-detail.js`

Must contain:
- Archive confirmation logic (reads `data-project-name` from the DOM)
- No global variables
- No inline event handlers in the HTML
- All DOM lookups after `DOMContentLoaded`

---

## What You Should NOT Do

- Do not use `<style>` blocks anywhere in templates
- Do not use `<script>` blocks with logic in templates
- Do not use `onclick="..."` or any inline event handlers
- Do not interpolate Django data into `<script>` tags: NOT `<script>var name = "{{ project.name }}";</script>`
- Do not hardcode colors in CSS — use CSS variables
- Do not duplicate the navbar or footer — use `{% include "components/navbar.html" %}`
- Do not use `/static/` hardcoded paths — always use `{% static 'path' %}`

---

## Checklist to Run When Done

Use the [Django Template Checklist](../../checklists/django/template.md#11-django-template-checklist--before-marking-done).

---

## Done When

```text
FOLDER STRUCTURE
[ ] Templates in templates/projects/
[ ] CSS in static/css/pages/projects.css
[ ] JS in static/js/pages/project-detail.js

TEMPLATE INHERITANCE
[ ] All templates start with {% extends "base.html" %}
[ ] {% load static %} at top of each template
[ ] Each template sets {% block title %}
[ ] No duplicate <html>, <head>, or <body> tags

CSS
[ ] Zero <style> blocks in any template
[ ] All styles in static/css/ files
[ ] CSS uses variables from base.css (no hardcoded colors)
[ ] Status badges styled with CSS classes (not inline style attributes)
[ ] Grid layout is responsive (1 col mobile, 2 tablet, 3 desktop)

JAVASCRIPT
[ ] Zero <script> blocks with logic in any template
[ ] No inline event handlers (onclick, etc.)
[ ] Archive button reads project name from data-project-name attribute
[ ] Confirmation fires before any destructive action

STATIC FILES
[ ] {% load static %} before any {% static %} usage
[ ] All static file paths use {% static 'path' %}
[ ] No hardcoded /static/ paths

FORMS
[ ] {% csrf_token %} inside every <form>
[ ] Field errors displayed below each field
[ ] Non-field errors shown as banner
[ ] Required fields have asterisk indicator

MOBILE
[ ] List page tested: cards stack on mobile
[ ] Form page tested: inputs usable on mobile, no overflow
[ ] Detail page tested: layout readable on mobile
```
