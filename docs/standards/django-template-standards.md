# Django Template & Static File Standards

Core rule:

> Templates display data. Views handle requests. Models handle data. Business logic never lives in templates. CSS, JavaScript, and HTML are always separated — never mixed into one file.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-the-three-folder-rule) | The Three-Folder Rule |
| [2](#2-static-folder-structure) | Static Folder Structure |
| [3](#3-template-folder-structure) | Template Folder Structure |
| [4](#4-file-naming-standard) | File Naming Standard |
| [5](#5-basehtml-standard) | base.html Standard |
| [6](#6-template-block-standards) | Template Block Standards |
| [7](#7-css-organization-rules) | CSS Organization Rules |
| [8](#8-javascript-rules) | JavaScript Rules |
| [9](#9-what-goes-where--decision-table) | What Goes Where — Decision Table |
| [10](#10-django-app-template-isolation) | Django App Template Isolation |
| [11](#11-static-file-loading-standard) | Static File Loading |
| [12](#12-reusable-template-components) | Reusable Template Components |
| [13](#13-anti-patterns-to-avoid) | Anti-Patterns to Avoid |
| [14](#14-definition-of-done--django-template-work) | Definition of Done |

---

# 1. The Three-Folder Rule

Every Django project must have a clean separation of static files across exactly three areas:

```text
static/
  css/       ← All styles. Nothing else.
  js/        ← All JavaScript. Nothing else.
  images/    ← All images, icons, logos.

templates/   ← All HTML. No inline styles. No inline scripts.
```

**Never:**
- Write `<style>` blocks inside an HTML template.
- Write `<script>` blocks with logic inside an HTML template.
- Put images directly in your `templates/` folder.
- Mix styles from multiple components into one giant CSS file.

---

# 2. Static Folder Structure

## Full Structure

```text
static/
  css/
    base.css                  ← Reset, typography, global variables
    layout.css                ← Grid, containers, section spacing
    components/
      button.css
      card.css
      form.css
      modal.css
      navbar.css
      table.css
      badge.css
      alert.css
    pages/
      home.css
      dashboard.css
      login.css
      profile.css

  js/
    base.js                   ← Global utilities, event init
    components/
      modal.js
      dropdown.js
      toast.js
      form-validator.js
      confirm-delete.js
    pages/
      dashboard.js
      home.js
      profile.js
    utils/
      api.js                  ← Fetch wrapper / CSRF helper
      format.js               ← Date/number formatters
      storage.js              ← localStorage helpers

  images/
    logo/
      logo.svg
      logo-white.svg
    icons/
      (SVG icons or icon sprites)
    ui/
      placeholder.png
      avatar-default.png
```

---

# 3. Template Folder Structure

```text
templates/
  base.html                   ← Master layout (head, nav, footer, blocks)
  base_auth.html              ← Layout for login/signup (no nav)

  includes/
    navbar.html
    footer.html
    sidebar.html
    pagination.html
    messages.html             ← Django messages / toast alerts
    breadcrumb.html

  components/
    card.html
    modal_confirm.html
    empty_state.html
    loading_spinner.html
    form_field.html           ← Reusable form field with label + error

  pages/
    home.html
    dashboard.html
    profile.html
    settings.html

  auth/
    login.html
    signup.html
    forgot_password.html
    reset_password.html

  errors/
    404.html
    500.html
    403.html

  emails/
    base_email.html
    welcome.html
    reset_password.html
    notification.html

  app_name/                   ← One subfolder per Django app
    project_list.html
    project_detail.html
    project_create.html
    project_edit.html
```

---

# 4. File Naming Standard

## CSS Files

| Good | Bad |
|------|-----|
| `button.css` | `btn_styles.css` |
| `project_card.css` | `ProjectCard.css` |
| `dashboard.css` | `page1.css` |
| `base.css` | `global_stuff.css` |

Use `snake_case`. Name by component or page — not by author, date, or feature number.

## JavaScript Files

| Good | Bad |
|------|-----|
| `modal.js` | `Modal.js` |
| `form-validator.js` | `validation_stuff.js` |
| `api.js` | `helper.js` |
| `dashboard.js` | `page_script_2.js` |

## Template Files

| Good | Bad |
|------|-----|
| `project_list.html` | `projects.html` |
| `project_detail.html` | `projectDetails.html` |
| `modal_confirm.html` | `ConfirmPopup.html` |
| `form_field.html` | `field.html` |

Use `snake_case`. Name by entity + action/type.

---

# 5. base.html Standard

Every project must have one master layout that all pages extend.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}App Name{% endblock %}</title>

  {% load static %}
  <link rel="stylesheet" href="{% static 'css/base.css' %}">
  <link rel="stylesheet" href="{% static 'css/layout.css' %}">
  {% block extra_css %}{% endblock %}
</head>
<body class="{% block body_class %}{% endblock %}">

  {% include "includes/navbar.html" %}

  <main>
    {% include "includes/messages.html" %}
    {% block content %}{% endblock %}
  </main>

  {% include "includes/footer.html" %}

  <script src="{% static 'js/base.js' %}"></script>
  {% block extra_js %}{% endblock %}
</body>
</html>
```

## Page Template Extending Base

```html
{% extends "base.html" %}
{% load static %}

{% block title %}Projects — App Name{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{% static 'css/pages/dashboard.css' %}">
{% endblock %}

{% block content %}
<section class="page-header">
  <h1>Projects</h1>
</section>

<section class="project-list">
  {% for project in projects %}
    {% include "components/card.html" with project=project %}
  {% empty %}
    {% include "components/empty_state.html" with message="No projects yet." %}
  {% endfor %}
</section>
{% endblock %}

{% block extra_js %}
<script src="{% static 'js/pages/dashboard.js' %}"></script>
{% endblock %}
```

---

# 6. Template Block Standards

| Block | Purpose |
|-------|---------|
| `{% block title %}` | Page title in `<head>` |
| `{% block extra_css %}` | Page-specific CSS files |
| `{% block body_class %}` | Body class for page-level styling |
| `{% block content %}` | Main page content |
| `{% block extra_js %}` | Page-specific JS files |
| `{% block sidebar %}` | Optional sidebar |

Never add page-specific CSS or JS in `base.html`. Always use `extra_css` and `extra_js` blocks.

---

# 7. CSS Organization Rules

## base.css — Contains

```css
/* 1. CSS Custom Properties (Variables) */
:root {
  --color-primary: #2563eb;
  --color-text: #111827;
  --color-background: #f9fafb;
  --color-border: #e5e7eb;
  --color-error: #dc2626;
  --color-success: #16a34a;

  --font-family: 'Inter', sans-serif;
  --font-size-base: 16px;

  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;

  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 32px;
  --spacing-xl: 64px;
}

/* 2. Reset */
/* 3. Typography */
/* 4. Utility classes */
```

## Component CSS Rule

Each component CSS file only styles that component.

```css
/* button.css */
.btn { ... }
.btn-primary { ... }
.btn-secondary { ... }
.btn-danger { ... }
.btn:disabled { ... }
.btn.loading { ... }
```

Never put button styles in `card.css`. Never put modal styles in `dashboard.css`.

## Page CSS Rule

Page CSS only styles page-specific layout. Reuse component classes.

```css
/* dashboard.css — only page-level layout */
.dashboard-grid { ... }
.dashboard-header { ... }
.stats-row { ... }
```

---

# 8. JavaScript Rules

## No Inline JavaScript in Templates

Bad:

```html
<button onclick="deleteProject({{ project.id }})">Delete</button>

<script>
  function deleteProject(id) {
    fetch('/delete/' + id);
  }
</script>
```

Good:

```html
<button class="btn-delete" data-id="{{ project.id }}" data-url="{% url 'project:delete' project.id %}">
  Delete
</button>
```

```js
// js/pages/dashboard.js
document.querySelectorAll('.btn-delete').forEach(btn => {
  btn.addEventListener('click', () => {
    const id = btn.dataset.id;
    const url = btn.dataset.url;
    // handle delete
  });
});
```

## Use `data-*` Attributes to Pass Django Data to JS

```html
<div id="project-map"
     data-lat="{{ project.latitude }}"
     data-lng="{{ project.longitude }}"
     data-name="{{ project.name }}">
</div>
```

```js
const map = document.getElementById('project-map');
const lat = map.dataset.lat;
const lng = map.dataset.lng;
```

Never interpolate Django variables directly inside JS strings in a template.

## CSRF Token for AJAX Calls

```js
// utils/api.js
function getCsrfToken() {
  return document.querySelector('[name=csrfmiddlewaretoken]')?.value
    || document.cookie.split('; ')
        .find(row => row.startsWith('csrftoken='))
        ?.split('=')[1];
}

async function post(url, data) {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRFToken': getCsrfToken()
    },
    body: JSON.stringify(data)
  });
  return response.json();
}
```

---

# 9. What Goes Where — Decision Table

| Logic Type | Where It Lives |
|------------|---------------|
| Business rules (pricing, permissions, status transitions) | Django view or service function |
| Database queries | Django view, manager, or querysets — never in template |
| Validation | Django form / serializer — not template |
| Display formatting (date, number) | Django template filter or template tag |
| Page layout and structure | HTML template |
| Styling and appearance | CSS file |
| User interaction (click, toggle, submit) | JS file |
| Inline styles for specific element | Use a CSS class, not `style=""` |
| Hardcoded content that may change | Move to database or settings |

## Never Put in Templates

```text
× Queries ({% for obj in Model.objects.all() %} is not valid Django, but avoid heavy logic)
× Business decisions ("if price > 100 and user.plan == 'free'...")
× Calculations
× Sensitive data output without escaping
× Inline styles
× Inline script blocks with logic
```

---

# 10. Django App Template Isolation

Every Django app should own its templates in a subfolder:

```text
templates/
  projects/          ← Templates for the "projects" app
    list.html
    detail.html
    create.html
    edit.html
  users/             ← Templates for the "users" app
    profile.html
    settings.html
  billing/           ← Templates for the "billing" app
    plans.html
    invoices.html
```

Reference in views:

```python
return render(request, 'projects/list.html', context)
```

---

# 11. Static File Loading Standard

Always use `{% load static %}` at the top of any template that references static files.

```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/components/card.css' %}">
<script src="{% static 'js/pages/dashboard.js' %}"></script>
<img src="{% static 'images/logo/logo.svg' %}" alt="App Logo">
```

Never hardcode static paths:

```html
<!-- Bad -->
<link rel="stylesheet" href="/static/css/card.css">

<!-- Good -->
<link rel="stylesheet" href="{% static 'css/components/card.css' %}">
```

---

# 12. Reusable Template Components

For UI patterns that repeat across pages, use `{% include %}`.

## Reusable Card Component

```html
<!-- templates/components/card.html -->
<div class="card">
  <div class="card-header">
    <h3>{{ title }}</h3>
    {% if subtitle %}<p class="card-subtitle">{{ subtitle }}</p>{% endif %}
  </div>
  <div class="card-body">
    {{ content }}
  </div>
  {% if actions %}
  <div class="card-footer">
    {{ actions }}
  </div>
  {% endif %}
</div>
```

## Usage

```html
{% include "components/card.html" with title="Project Name" subtitle="Created today" %}
```

## Reusable Form Field

```html
<!-- templates/components/form_field.html -->
<div class="form-group {% if field.errors %}has-error{% endif %}">
  <label for="{{ field.id_for_label }}">
    {{ field.label }}
    {% if field.field.required %}<span class="required-star">*</span>{% endif %}
  </label>
  {{ field }}
  {% if field.errors %}
    <span class="field-error">{{ field.errors|first }}</span>
  {% endif %}
  {% if field.help_text %}
    <span class="field-hint">{{ field.help_text }}</span>
  {% endif %}
</div>
```

## Usage

```html
{% for field in form %}
  {% include "components/form_field.html" with field=field %}
{% endfor %}
```

---

# 13. Anti-Patterns to Avoid

| Anti-Pattern | Fix |
|-------------|-----|
| `<style>` blocks inside templates | Move to component CSS file |
| `<script>` with logic inside templates | Move to page JS file |
| Giant single `styles.css` for entire project | Split into base + component + page CSS files |
| Giant single `main.js` for entire project | Split into component and page JS files |
| Duplicating the same HTML section across pages | Extract to `{% include %}` component |
| Hardcoded color/spacing values in CSS | Use CSS custom properties in `base.css` |
| Business logic in template conditionals | Move to view or model property |
| Django variables directly interpolated in JS strings | Use `data-*` attributes |
| No base template — every page is standalone | Implement template inheritance with `base.html` |

---

# 14. Definition of Done — Django Template Work

```text
[ ] Template extends base.html using {% extends %}
[ ] No inline <style> blocks
[ ] No inline <script> blocks with logic
[ ] Page-specific CSS in static/css/pages/[page].css
[ ] Page-specific JS in static/js/pages/[page].js
[ ] CSS variables used from base.css (not hardcoded hex/px values)
[ ] Repeated UI sections extracted to {% include %} components
[ ] Django data passed to JS via data-* attributes (not string interpolation)
[ ] Template file named in snake_case matching entity + action
[ ] Static files referenced using {% static %} tag
[ ] Mobile layout checked
[ ] No business logic in template
```
