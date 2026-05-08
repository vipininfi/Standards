# Project — Internal Knowledge Base

**Stack:** Django + Django Templates (HTML/CSS/JS)
**Estimated Time:** 8 hours
**Difficulty:** Intermediate

---

## What You Are Building

A server-rendered internal knowledge base. Members can browse and read articles organised by category. Editors and admins can create and edit articles. Only admins can delete articles. Articles can be saved as drafts (visible only to editors/admins) or published (visible to all members). Django handles the full request-response cycle. No React. All pages extend `base.html`. All CSS in `static/css/`, all JS in `static/js/`. Data passed to JavaScript via `data-*` attributes only.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Django models: Category, Article | 45m |
| 2 | Django forms: CategoryForm, ArticleForm | 30m |
| 3 | Django service: ArticleService | 45m |
| 4 | Django views: 7 views (list, detail, create, edit, delete, category list, category create) | 1h 30m |
| 5 | Templates: article_list.html, article_detail.html, article_form.html, category_list.html | 2h |
| 6 | Static CSS: article cards, category badges, search bar, status badges | 45m |
| 7 | Static JS: delete confirmation, character counter for excerpt | 30m |

---

## 1. Django Models

### `Category`

```python
from django.db import models
from django.utils.text import slugify

class Category(models.Model):
    workspace   = models.ForeignKey('Workspace', on_delete=models.CASCADE, related_name='categories')
    name        = models.CharField(max_length=100)
    slug        = models.SlugField(max_length=110)
    description = models.TextField(blank=True, default='')
    created_at  = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['name']
        verbose_name_plural = 'Categories'
        constraints = [
            models.UniqueConstraint(
                fields=['workspace', 'slug'],
                name='unique_category_slug_per_workspace'
            ),
        ]

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)

    def __str__(self):
        return self.name
```

### `Article`

```python
class Article(models.Model):

    class Status(models.TextChoices):
        DRAFT     = 'draft',     'Draft'
        PUBLISHED = 'published', 'Published'

    workspace   = models.ForeignKey('Workspace', on_delete=models.CASCADE, related_name='articles')
    category    = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True,
                                    related_name='articles')
    author      = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                    null=True, related_name='authored_articles')
    title       = models.CharField(max_length=200)
    slug        = models.SlugField(max_length=220)
    excerpt     = models.CharField(max_length=300, blank=True, default='')
    body        = models.TextField()
    status      = models.CharField(max_length=20, choices=Status.choices, default=Status.DRAFT)
    created_at  = models.DateTimeField(auto_now_add=True)
    updated_at  = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        ordering = ['-created_at']
        constraints = [
            models.UniqueConstraint(
                fields=['workspace', 'slug'],
                name='unique_article_slug_per_workspace'
            ),
        ]
        indexes = [
            models.Index(fields=['workspace', 'status', '-created_at']),
            models.Index(fields=['workspace', 'category', 'status']),
        ]

    def __str__(self):
        return self.title
```

**Run checklist:** [Django Model](../../../../checklists/django/model.md)

---

## 2. Django Forms

### `CategoryForm`

```python
class CategoryForm(forms.ModelForm):
    class Meta:
        model = Category
        fields = ['name', 'description']

    def clean_name(self):
        name = self.cleaned_data.get('name', '').strip()
        if not name:
            raise forms.ValidationError("Category name is required.")
        return name
```

### `ArticleForm`

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'category', 'excerpt', 'body', 'status']
        widgets = {
            'excerpt': forms.Textarea(attrs={'rows': 3, 'maxlength': 300}),
            'body':    forms.Textarea(attrs={'rows': 20}),
        }

    def __init__(self, *args, workspace=None, **kwargs):
        super().__init__(*args, **kwargs)
        if workspace:
            self.fields['category'].queryset = Category.objects.filter(workspace=workspace)
        self.fields['category'].required = False

    def clean_title(self):
        title = self.cleaned_data.get('title', '').strip()
        if not title:
            raise forms.ValidationError("Title is required.")
        return title

    def clean_body(self):
        body = self.cleaned_data.get('body', '').strip()
        if not body:
            raise forms.ValidationError("Body is required.")
        return body
```

**Run checklist:** [Django Form / Serializer](../../../../checklists/django/form-serializer.md)

---

## 3. ArticleService

```python
# services.py
from django.utils import timezone
from django.utils.text import slugify
from django.core.exceptions import ValidationError, PermissionDenied

class ArticleService:

    @staticmethod
    def create_article(workspace, author, title, body, category=None, excerpt='', status='draft'):
        """
        Creates a new article.
        Validates:
          - title and body are not empty
          - slug is unique within workspace (auto-generated from title; append suffix if collision)
        Sets published_at if status == 'published'.
        """

    @staticmethod
    def update_article(article, editor, title, body, category=None, excerpt='', status='draft'):
        """
        Updates an existing article.
        Validates:
          - editor is a workspace member (editor or admin role)
          - title and body are not empty
        Re-generates slug only if title changed and slug collision would occur.
        Sets published_at if status changes to 'published' and was previously not.
        """

    @staticmethod
    def delete_article(article, actor):
        """
        Deletes an article.
        Raises PermissionDenied if actor does not have admin or owner role in the workspace.
        """

    @staticmethod
    def _unique_slug(workspace, base_slug, exclude_id=None):
        """
        Returns a unique slug for the workspace.
        If base_slug is taken, appends -2, -3, etc.
        """
```

---

## 4. Django Views

### `WorkspaceMemberMixin`

```python
class WorkspaceMemberMixin(LoginRequiredMixin):
    def dispatch(self, request, *args, **kwargs):
        self.workspace = get_object_or_404(Workspace, id=kwargs['workspace_id'])
        self.membership = get_object_or_404(
            WorkspaceMember, workspace=self.workspace, user=request.user, status='active'
        )
        return super().dispatch(request, *args, **kwargs)
```

### Views to Build

```text
ArticleListView (GET /workspace/{id}/kb/)
  — WorkspaceMemberMixin
  — Members see published articles only
  — Editors/admins see published + their own drafts
  — Supports ?search= (title/excerpt contains, DB query — not client-side)
  — Supports ?category= (slug filter)
  — Paginated: 12 per page
  — Template: article_list.html

ArticleDetailView (GET /workspace/{id}/kb/<slug:slug>/)
  — WorkspaceMemberMixin
  — Members: 404 if article is draft and they are not the author or admin
  — Template: article_detail.html

ArticleCreateView (GET+POST /workspace/{id}/kb/new/)
  — WorkspaceMemberMixin + editor/admin role check (403 if member role)
  — GET: render empty ArticleForm
  — POST: validate → ArticleService.create_article() → redirect to article detail
  — Template: article_form.html

ArticleEditView (GET+POST /workspace/{id}/kb/<slug:slug>/edit/)
  — WorkspaceMemberMixin + editor/admin role check
  — GET: render ArticleForm prefilled
  — POST: validate → ArticleService.update_article() → redirect to article detail
  — Template: article_form.html (same template as create, different heading)

ArticleDeleteView (POST /workspace/{id}/kb/<slug:slug>/delete/)
  — WorkspaceMemberMixin + admin/owner role check (403 if editor or member)
  — POST only (from a form button with CSRF)
  — Calls ArticleService.delete_article()
  — Redirects to article list with success/error message

CategoryListView (GET /workspace/{id}/kb/categories/)
  — WorkspaceMemberMixin + admin/owner role check
  — Lists all categories for the workspace
  — Supports ?search= (name contains)
  — Template: category_list.html

CategoryCreateView (GET+POST /workspace/{id}/kb/categories/new/)
  — WorkspaceMemberMixin + admin/owner role check
  — GET: render empty CategoryForm
  — POST: validate → save category → redirect to category list
  — On duplicate slug: show form error "A category with this name already exists."
  — Template: category_form.html
```

**Run checklist:** [Django New View](../../../../checklists/django/new-view.md) for each view

---

## 5. Templates

All templates must:
- Extend `base.html`
- Have NO `<style>` blocks (all CSS in `static/css/kb.css`)
- Have NO inline `<script>` logic (all JS in `static/js/kb.js`)
- Load static with `{% load static %}`

### `article_list.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}Knowledge Base{% endblock %}

{% block content %}
  <!-- Search + category filter bar (GET form) -->
  <!-- Active filter chips: show if ?search= or ?category= is set -->
  <!-- "New Article" button — visible to editor/admin only -->
  <!-- Article cards grid -->
  <!-- Each card: title, excerpt, category badge, status badge (draft/published), author, date -->
  <!-- Empty state: "No articles found." -->
  <!-- Pagination links -->
{% endblock %}
```

### `article_detail.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}{{ article.title }}{% endblock %}

{% block content %}
  <!-- Breadcrumb: Knowledge Base / [Category] / [Title] -->
  <!-- Status badge (draft warning for editors/admins) -->
  <!-- Article title, author, published date -->
  <!-- Category badge linking to filtered list -->
  <!-- Article body (rendered as HTML from trusted source) -->
  <!-- Edit button — visible to editor/admin only -->
  <!-- Delete button — visible to admin/owner only, POST form with CSRF -->
{% endblock %}
```

### `article_form.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}{% if article %}Edit Article{% else %}New Article{% endif %}{% endblock %}

{% block content %}
  <!-- Heading: "Edit Article" or "New Article" -->
  <form method="post">
    {% csrf_token %}
    <!-- Render each field individually (not {{ form.as_p }}) -->
    <!-- Show field errors below each field -->
    <!-- Show non-field errors at top of form -->
    <!-- Excerpt field: show character counter (300 max) via data-* and JS -->
    <!-- Status field: dropdown (Draft / Published) -->
    <!-- Submit button: "Save Article" with loading state via data-* -->
    <!-- Cancel link back to article list or detail -->
  </form>
{% endblock %}
```

### `category_list.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}Categories{% endblock %}

{% block content %}
  <!-- "New Category" button (admin/owner only) -->
  <!-- Search bar (GET form) -->
  <!-- Categories table: Name | Description | Article Count | Actions -->
  <!-- Delete button per category — POST form with CSRF, confirmation via data-* and JS -->
  <!-- Empty state: "No categories yet." -->
{% endblock %}
```

### `category_form.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}New Category{% endblock %}

{% block content %}
  <form method="post">
    {% csrf_token %}
    <!-- Name field with error -->
    <!-- Description field with error -->
    <!-- Submit button -->
  </form>
{% endblock %}
```

**Run checklist:** [Django Template](../../../../checklists/django/template.md) for each template

---

## 6. Static CSS (`static/css/kb.css`)

```text
[ ] Article card styles (grid layout, hover shadow, border-radius)
[ ] Status badge styles using CSS variables:
    .badge-draft     { background: var(--color-warning); color: white; }
    .badge-published { background: var(--color-success); color: white; }
[ ] Category badge styles (.badge-category)
[ ] Search bar and filter chip styles
[ ] Article detail: body typography (headings, paragraphs, lists, code blocks)
[ ] Breadcrumb styles
[ ] Table styles for category list (striped rows, hover)
[ ] Form layout: label above field, character counter below excerpt
[ ] Pagination styles
[ ] Empty state styles (.empty-state — centered, muted text)
[ ] Responsive: card grid collapses to 1 column on mobile
[ ] NO hardcoded colors — all in CSS variables: --color-success, --color-warning, --color-danger
```

---

## 7. Static JS (`static/js/kb.js`)

```text
All JavaScript must use data-* attributes — never string interpolation in script tags.

DELETE CONFIRMATION:
<!-- HTML -->
<button
  class="btn-delete-article"
  data-article-title="{{ article.title }}"
  data-form-id="delete-form-{{ article.id }}"
>Delete</button>
<form id="delete-form-{{ article.id }}" method="post"
      action="{% url 'delete-article' workspace.id article.slug %}" style="display:none;">
  {% csrf_token %}
</form>

// JS
document.querySelectorAll('.btn-delete-article').forEach(btn => {
  btn.addEventListener('click', () => {
    const title = btn.dataset.articleTitle;
    if (confirm(`Delete "${title}"? This cannot be undone.`)) {
      document.getElementById(btn.dataset.formId).submit();
    }
  });
});

CHARACTER COUNTER FOR EXCERPT:
<!-- HTML -->
<textarea
  name="excerpt"
  class="excerpt-field"
  data-max-length="300"
  data-counter-id="excerpt-counter"
  maxlength="300"
></textarea>
<span id="excerpt-counter" class="char-counter">0 / 300</span>

// JS
document.querySelectorAll('.excerpt-field').forEach(field => {
  const counterId = field.dataset.counterId;
  const maxLength = parseInt(field.dataset.maxLength, 10);
  const counter = document.getElementById(counterId);
  const update = () => {
    counter.textContent = `${field.value.length} / ${maxLength}`;
  };
  field.addEventListener('input', update);
  update();
});

SUBMIT LOADING STATE:
<!-- HTML -->
<button type="submit" class="btn-submit-form" data-loading-text="Saving...">
  Save Article
</button>

// JS
document.querySelectorAll('.btn-submit-form').forEach(btn => {
  btn.closest('form').addEventListener('submit', () => {
    btn.disabled = true;
    btn.textContent = btn.dataset.loadingText;
  });
});
```

---

## URL Configuration

```python
urlpatterns = [
    path('workspace/<int:workspace_id>/kb/',
         ArticleListView.as_view(), name='article-list'),
    path('workspace/<int:workspace_id>/kb/new/',
         ArticleCreateView.as_view(), name='article-create'),
    path('workspace/<int:workspace_id>/kb/categories/',
         CategoryListView.as_view(), name='category-list'),
    path('workspace/<int:workspace_id>/kb/categories/new/',
         CategoryCreateView.as_view(), name='category-create'),
    path('workspace/<int:workspace_id>/kb/<slug:slug>/',
         ArticleDetailView.as_view(), name='article-detail'),
    path('workspace/<int:workspace_id>/kb/<slug:slug>/edit/',
         ArticleEditView.as_view(), name='article-edit'),
    path('workspace/<int:workspace_id>/kb/<slug:slug>/delete/',
         ArticleDeleteView.as_view(), name='article-delete'),
]
```

**Note:** Place the `new/` and `categories/` URLs before the `<slug:slug>/` pattern to avoid `new` and `categories` being matched as slugs.

---

## Permissions Summary

| Action | Member | Editor | Admin / Owner |
|--------|--------|--------|---------------|
| Read published articles | Yes | Yes | Yes |
| Read draft articles | Own only | Own only | All |
| Create / Edit articles | No | Yes | Yes |
| Delete articles | No | No | Yes |
| Manage categories | No | No | Yes |

---

## What You Should NOT Do

```text
× Filter draft/published visibility in the template — do it in the queryset
× Skip the slug uniqueness check — two articles with the same title would collide
× Let member-role users access the create/edit views (check role, raise PermissionDenied)
× Let editor-role users delete articles (admin/owner only)
× Pass article content to JavaScript via {{ variable }} in a script tag
× Use client-side search filtering — search must query the database
× Not extend base.html in every template
× Add <style> blocks to templates — all CSS in static/css/kb.css
× Skip CSRF token on any POST form (delete, category create)
× Use {{ form.as_p }} — render each field individually
× Generate slugs with collisions — use _unique_slug() with suffix fallback
```

---

## Checklists to Run (in order)

```text
[ ] Django Model — Category model
[ ] Django Model — Article model
[ ] Django Form / Serializer — ArticleForm, CategoryForm
[ ] Django New View — each of the 7 views
[ ] Django Template — article_list.html, article_detail.html, article_form.html, category_list.html
[ ] Permissions & Role-Based UI — draft visibility, create/edit/delete gating
[ ] Error Handling — form errors shown per field, service errors shown at top
[ ] Delete & Destructive Actions — article delete confirmation
[ ] Notifications & Toasts — success/error messages after redirect (Django messages framework)
```

---

## Done When

```text
[ ] Category model: name, slug (auto-generated), UniqueConstraint per workspace, ordering by name
[ ] Article model: all fields, Status TextChoices, UniqueConstraint on slug per workspace, indexes
[ ] ArticleService: create generates unique slug, update re-slugifies only on title change
[ ] ArticleService: published_at set when status becomes 'published'
[ ] ArticleService: delete raises PermissionDenied for non-admin actors
[ ] ArticleListView: members see published only; editors/admins see published + their own drafts
[ ] ArticleListView: ?search= filters title/excerpt in DB (not client-side)
[ ] ArticleListView: ?category= filters by category slug
[ ] ArticleDetailView: returns 404 for draft if requester is member (not author/admin)
[ ] ArticleCreateView / ArticleEditView: 403 for member-role users
[ ] ArticleDeleteView: 403 for editor and member roles; admin/owner only
[ ] CategoryCreateView: duplicate slug shows form error, not a crash
[ ] All POST views: redirect on success (POST→redirect→GET pattern)
[ ] All templates: extend base.html, no inline styles, no inline scripts
[ ] Status badges: .badge-draft and .badge-published using CSS variables
[ ] JS: delete confirmation uses data-* attributes (no string interpolation in script)
[ ] JS: excerpt character counter reads data-max-length from HTML (not hardcoded in JS)
[ ] JS: submit button loading state uses data-loading-text attribute
[ ] Filter bar: GET method, search/category filter applied from DB
[ ] Breadcrumb: on article detail showing Knowledge Base / Category / Title
[ ] Pagination: works on article list and category list pages
[ ] Django messages: success/error toast shown after redirect
[ ] Mobile: card grid collapses to single column at small screen width
[ ] CSRF token present on all POST forms
[ ] URL order: new/ and categories/ routes registered before <slug:slug>/ catch-all
```
