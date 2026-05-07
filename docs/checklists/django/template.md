# Django — Template Checklist

> **Core Rule:** Templates display data — they do not contain logic, styles, or scripts. Three separate folders: `static/css/`, `static/js/`, `templates/`. Every template extends `base.html`. No inline styles. No inline scripts.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-folder-structure) | Folder Structure |
| [2](#2-template-inheritance) | Template Inheritance |
| [3](#3-css-standards) | CSS Standards |
| [4](#4-javascript-standards) | JavaScript Standards |
| [5](#5-passing-data-from-django-to-javascript) | Passing Data from Django to JavaScript |
| [6](#6-reusable-template-components) | Reusable Template Components |
| [7](#7-template-logic-rules) | Template Logic Rules |
| [8](#8-static-file-loading) | Static File Loading |
| [9](#9-naming-standards) | Naming Standards |
| [10](#10-anti-patterns) | Anti-Patterns |
| [11](#11-django-template-checklist--before-marking-done) | Django Template Checklist — Before Marking Done |

---

# 1. Folder Structure

```text
[ ] CSS in static/css/ — never inline in templates
[ ] JavaScript in static/js/ — never inline in templates
[ ] Templates in templates/ — never mixed with static files
[ ] App-specific templates in templates/[app_name]/
[ ] App-specific CSS in static/[app_name]/css/
[ ] App-specific JS in static/[app_name]/js/
```

**Correct folder structure:**

```
myproject/
  static/
    css/
      base.css           ← global variables, reset, shared styles
      components/
        navbar.css
        card.css
        button.css
      pages/
        dashboard.css
        project-detail.css
    js/
      components/
        navbar.js
        modal.js
      pages/
        dashboard.js
        project-form.js
  templates/
    base.html
    components/
      navbar.html
      footer.html
      card.html
    projects/
      list.html
      detail.html
      form.html
    users/
      profile.html
```

---

# 2. Template Inheritance

```text
[ ] Every page template extends base.html: {% extends "base.html" %}
[ ] base.html defines all layout blocks
[ ] Child templates only override the blocks they need
[ ] No duplicate HTML structure (no <html>, <head>, <body> in child templates)
```

**base.html required blocks:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}App Name{% endblock %}</title>
  {% load static %}
  <link rel="stylesheet" href="{% static 'css/base.css' %}">
  {% block extra_css %}{% endblock %}
</head>
<body>
  {% include "components/navbar.html" %}

  <main>
    {% block content %}{% endblock %}
  </main>

  {% include "components/footer.html" %}

  <script src="{% static 'js/base.js' %}"></script>
  {% block extra_js %}{% endblock %}
</body>
</html>
```

**Child template pattern:**

```html
{% extends "base.html" %}
{% load static %}

{% block title %}Projects — App Name{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{% static 'css/pages/projects.css' %}">
{% endblock %}

{% block content %}
<!-- page content here -->
{% endblock %}

{% block extra_js %}
<script src="{% static 'js/pages/projects.js' %}"></script>
{% endblock %}
```

---

# 3. CSS Standards

```text
[ ] No <style> blocks anywhere in templates — zero exceptions
[ ] All CSS in static/css/ files
[ ] CSS variables defined in base.css (not hardcoded per file):
  :root {
    --color-primary: #0070f3;
    --color-text: #111827;
    --spacing-md: 1rem;
    --font-base: 'Inter', sans-serif;
  }
[ ] Component CSS in static/css/components/
[ ] Page-specific CSS in static/css/pages/
[ ] Responsive breakpoints defined consistently across all CSS files
[ ] No !important unless absolutely unavoidable (comment explaining why)
[ ] Class names use kebab-case: .project-card, .nav-item, .btn-primary
```

---

# 4. JavaScript Standards

```text
[ ] No <script> blocks with logic in templates — zero exceptions
[ ] All JavaScript in static/js/ files
[ ] No inline event handlers in HTML: NOT onclick="doSomething()"
[ ] Event listeners added in JS files using addEventListener
[ ] CSRF token helper defined in base.js and reused across all JS files
[ ] All JS files are non-blocking: placed before </body> or loaded with defer
[ ] No JavaScript frameworks (React, Vue) mixed into Django templates — use Django templates OR a React frontend, not both
```

---

# 5. Passing Data from Django to JavaScript

**Use `data-*` attributes — never string interpolation in `<script>` tags.**

```text
[ ] Django context data passed to JS via data-* attributes on HTML elements
[ ] Never: <script>var projectId = "{{ project.id }}";</script>
[ ] Always: <div id="project-root" data-project-id="{{ project.id }}"></div>
[ ] JS reads data-* attributes: document.getElementById('project-root').dataset.projectId
```

**Correct pattern:**

```html
<!-- template: projects/detail.html -->
{% block content %}
<div
  id="project-root"
  data-project-id="{{ project.id }}"
  data-workspace-id="{{ workspace.id }}"
  data-can-edit="{{ can_edit|yesno:'true,false' }}"
>
  <!-- content -->
</div>
{% endblock %}
```

```js
// static/js/pages/project-detail.js
const root = document.getElementById('project-root');
const projectId = root.dataset.projectId;
const canEdit = root.dataset.canEdit === 'true';
```

**For CSRF token in fetch requests:**

```js
// static/js/base.js — included in base.html
function getCsrfToken() {
  return document.querySelector('[name=csrfmiddlewaretoken]')?.value
    ?? document.cookie.split('; ')
        .find(c => c.startsWith('csrftoken='))
        ?.split('=')[1] ?? '';
}

// Usage in any fetch call
fetch('/api/projects/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRFToken': getCsrfToken(),
  },
  body: JSON.stringify(data),
});
```

---

# 6. Reusable Template Components

```text
[ ] Repeated sections use {% include %} — not copy-pasted HTML
[ ] Components stored in templates/components/
[ ] Component files named clearly: navbar.html, footer.html, card.html, form-field.html
[ ] Components accept context variables where needed
[ ] Header/navbar is one file — not rebuilt on every page
[ ] Footer is one file — not rebuilt on every page
[ ] Form field HTML (label + input + error display) is one component — not repeated inline
```

**Include with context:**

```html
<!-- Using a card component -->
{% for project in projects %}
  {% include "components/project-card.html" with project=project %}
{% endfor %}
```

---

# 7. Template Logic Rules

```text
[ ] No complex Python logic in templates — only display logic
[ ] Allowed in templates:
  — {{ variable }} output
  — {% if %} / {% else %} / {% endif %}
  — {% for %} / {% endfor %}
  — {% include %}, {% extends %}, {% block %}
  — {{ variable|filter }} (built-in or custom filters)
[ ] NOT allowed in templates:
  — Complex calculations
  — Database queries (use view/service to prepare data)
  — String manipulation (use template filter or prepare in view)
  — Conditional class logic beyond simple is_active type checks
[ ] Custom template filters defined in templatetags/ — not inline in templates
[ ] Template receives pre-processed context from the view — not raw querysets needing complex logic
```

---

# 8. Static File Loading

```text
[ ] {% load static %} at the top of every template that uses {% static %}
[ ] Static files referenced using {% static 'path/to/file' %} — not hardcoded /static/ paths
[ ] Never hardcode: <link href="/static/css/base.css"> — always use {% static %}
[ ] STATICFILES_DIRS and STATIC_URL configured in settings.py
[ ] python manage.py collectstatic run before production deployment
[ ] Versioning/cache busting handled via ManifestStaticFilesStorage or equivalent
```

---

# 9. Naming Standards

```text
[ ] Template files: snake_case.html (project_detail.html, user_profile.html)
[ ] CSS files: kebab-case.css (project-card.css, user-profile.css)
[ ] JS files: kebab-case.js (project-form.js, dashboard.js)
[ ] CSS class names: kebab-case (.project-card, .nav-item)
[ ] JS function names: camelCase (loadProjectData, handleFormSubmit)
[ ] data-* attribute names: kebab-case (data-project-id, data-workspace-id)
[ ] Template blocks: lowercase underscore (content, extra_css, extra_js, page_title)
```

---

# 10. Anti-Patterns

| Anti-Pattern | Impact | Fix |
|-------------|--------|-----|
| `<style>` block in template | Styles can't be cached, inconsistent across pages | Move to static/css/ |
| `<script>` block with logic in template | Can't be linted, can't be reused, hard to debug | Move to static/js/ |
| `var x = "{{ variable }}"` in script tag | XSS risk, tightly coupled | Use data-* attributes |
| `onclick="doSomething()"` inline | Hard to maintain, can't be tested | Use addEventListener in JS file |
| Page template not extending base.html | Duplicate head/body markup, inconsistent layout | Always extend base.html |
| Navbar HTML on every page | One change needs updating every page | {% include "components/navbar.html" %} |
| Complex Python-like logic in templates | Moves business logic to templates | Prepare data in view/service |
| Hardcoded /static/ paths | Breaks when STATIC_URL changes | Use {% static %} tag |
| app-name missing in template path | Templates conflict across apps | Use templates/app_name/file.html |

---

# 11. Django Template Checklist — Before Marking Done

```text
FOLDER STRUCTURE
[ ] CSS only in static/css/
[ ] JS only in static/js/
[ ] Templates only in templates/
[ ] App-scoped templates in templates/[app_name]/

TEMPLATE INHERITANCE
[ ] Template starts with {% extends "base.html" %}
[ ] Only overriding blocks defined in base.html
[ ] No duplicate <html>, <head>, or <body> tags

CSS
[ ] Zero <style> blocks in the template
[ ] All styles in static/css/ files
[ ] CSS variables used (no hardcoded colors or spacing)

JAVASCRIPT
[ ] Zero <script> blocks with logic in template
[ ] All JS in static/js/ files
[ ] No inline event handlers (onclick, onsubmit, etc.)
[ ] CSRF token handled via getCsrfToken() helper

DATA → JS
[ ] Django data passed via data-* attributes
[ ] No string interpolation inside <script> tags
[ ] JS reads data-* values from DOM elements

REUSABLE COMPONENTS
[ ] Repeated sections use {% include %}
[ ] Navbar in templates/components/navbar.html
[ ] Footer in templates/components/footer.html

STATIC FILES
[ ] {% load static %} at top
[ ] All static paths use {% static 'path' %}
[ ] No hardcoded /static/ paths

NAMING
[ ] Template files: snake_case.html
[ ] CSS files: kebab-case.css
[ ] JS files: kebab-case.js

MOBILE
[ ] Tested on mobile browser (responsive layout verified)
```

---

## Practice Task

Apply what you learned by building real Django templates with zero inline styles, zero inline scripts, and correct data-* attribute patterns.

**→ [Task 02: Build the Project Detail Template](../../tasks/django/02-project-template.md)**

Covers: 3-folder structure, base.html inheritance, project list/form/detail templates, status badge with CSS classes (no inline style), archive confirmation via data-project-name attribute (no string interpolation in script tags), responsive grid in CSS with variables.
