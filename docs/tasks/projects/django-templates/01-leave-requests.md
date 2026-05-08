# Project — Leave Request System

**Stack:** Django + Django Templates (HTML/CSS/JS)
**Estimated Time:** 8 hours
**Difficulty:** Intermediate

---

## What You Are Building

A server-rendered leave request system. Employees submit leave requests, managers approve or reject them. Django handles the full request-response cycle. No React. All pages extend `base.html`. All CSS in `static/css/`, all JS in `static/js/`. Data passed to JavaScript via `data-*` attributes only.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Django model: LeaveRequest | 45m |
| 2 | Django form: LeaveRequestForm | 30m |
| 3 | Django service: LeaveService | 45m |
| 4 | Django views: 5 views (list, submit, admin, approve, reject) | 1h 30m |
| 5 | Templates: my_leaves.html, submit_form.html, admin_leaves.html | 2h |
| 6 | Static CSS: badges, form styles, status table | 45m |
| 7 | Static JS: confirm dialog for cancel + approve/reject | 30m |

---

## 1. Django Model

### `LeaveRequest`

```python
from django.db import models
from django.conf import settings

class LeaveRequest(models.Model):

    class LeaveType(models.TextChoices):
        ANNUAL   = 'annual',   'Annual Leave'
        SICK     = 'sick',     'Sick Leave'
        PERSONAL = 'personal', 'Personal Leave'
        UNPAID   = 'unpaid',   'Unpaid Leave'

    class Status(models.TextChoices):
        SUBMITTED  = 'submitted',  'Submitted'
        APPROVED   = 'approved',   'Approved'
        REJECTED   = 'rejected',   'Rejected'
        CANCELLED  = 'cancelled',  'Cancelled'

    workspace     = models.ForeignKey('Workspace', on_delete=models.CASCADE, related_name='leave_requests')
    employee      = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE,
                                      related_name='leave_requests')
    leave_type    = models.CharField(max_length=20, choices=LeaveType.choices)
    start_date    = models.DateField()
    end_date      = models.DateField()
    total_days    = models.PositiveIntegerField()
    reason        = models.TextField(blank=True, default='')
    status        = models.CharField(max_length=20, choices=Status.choices,
                                     default=Status.SUBMITTED)
    reviewed_by   = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                      null=True, blank=True, related_name='reviewed_leaves')
    reviewed_at   = models.DateTimeField(null=True, blank=True)
    review_notes  = models.TextField(blank=True, default='')
    created_at    = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = 'Leave Request'
        verbose_name_plural = 'Leave Requests'
        constraints = [
            models.CheckConstraint(
                check=models.Q(end_date__gte=models.F('start_date')),
                name='leave_end_date_after_start'
            ),
            models.CheckConstraint(
                check=models.Q(total_days__gt=0),
                name='leave_total_days_positive'
            ),
        ]
        indexes = [
            models.Index(fields=['workspace', 'employee', 'status']),
            models.Index(fields=['workspace', 'status', '-created_at']),
        ]

    def __str__(self):
        return f"{self.employee.get_full_name()} — {self.get_leave_type_display()} ({self.start_date} to {self.end_date})"
```

**Run checklist:** [Django Model](../../../../checklists/django/model.md)

---

## 2. Django Form

### `LeaveRequestForm`

```python
class LeaveRequestForm(forms.ModelForm):
    class Meta:
        model = LeaveRequest
        fields = ['leave_type', 'start_date', 'end_date', 'reason']
        widgets = {
            'start_date': forms.DateInput(attrs={'type': 'date'}),
            'end_date':   forms.DateInput(attrs={'type': 'date'}),
        }

    def clean_start_date(self):
        start = self.cleaned_data.get('start_date')
        if start and start < date.today():
            raise forms.ValidationError("Start date cannot be in the past.")
        return start

    def clean(self):
        cleaned = super().clean()
        start = cleaned.get('start_date')
        end = cleaned.get('end_date')
        if start and end:
            if end < start:
                raise forms.ValidationError({'end_date': "End date must be on or after start date."})
        return cleaned
```

**Run checklist:** [Django Form / Serializer](../../../../checklists/django/form-serializer.md)

---

## 3. LeaveService

```python
# services.py
from datetime import date, datetime
from django.core.exceptions import ValidationError
from django.utils import timezone

class LeaveService:

    @staticmethod
    def submit_request(workspace, employee, leave_type, start_date, end_date, reason=''):
        """
        Submits a leave request.
        Validates:
          - start_date >= today
          - end_date >= start_date
          - No overlapping submitted/approved request for this employee
        Computes total_days.
        """

    @staticmethod
    def approve_request(leave_request, reviewed_by, review_notes=''):
        """
        Approves a leave request.
        Raises ValidationError if status != 'submitted'.
        Sets status = 'approved', reviewed_by, reviewed_at.
        """

    @staticmethod
    def reject_request(leave_request, reviewed_by, review_notes):
        """
        Rejects a leave request.
        review_notes is required.
        Raises ValidationError if status != 'submitted'.
        """

    @staticmethod
    def cancel_request(leave_request, actor):
        """
        Cancels a leave request.
        Only the employee who submitted it can cancel.
        Raises PermissionDenied if actor != employee.
        Raises ValidationError if status != 'submitted'.
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
MyLeavesView (GET /workspace/{id}/my-leaves/)
  — WorkspaceMemberMixin
  — Returns request.user's leave requests (paginated, 10 per page)
  — Supports ?status= filter
  — Template: my_leaves.html

SubmitLeaveView (GET+POST /workspace/{id}/my-leaves/submit/)
  — WorkspaceMemberMixin
  — GET: render empty LeaveRequestForm
  — POST: validate form → LeaveService.submit_request() → redirect to my-leaves
  — Template: submit_form.html

CancelLeaveView (POST /workspace/{id}/my-leaves/{leave_id}/cancel/)
  — WorkspaceMemberMixin
  — POST only (from a form button with CSRF)
  — Calls LeaveService.cancel_request()
  — Redirects back to my-leaves with a success/error message

AdminLeavesView (GET /workspace/{id}/admin/leaves/)
  — WorkspaceMemberMixin + admin role check (403 if not admin)
  — Returns all workspace leave requests (paginated, 20 per page)
  — Supports ?status= and ?employee_id= filters
  — Template: admin_leaves.html

ApproveLeaveView (POST /workspace/{id}/admin/leaves/{leave_id}/approve/)
  — WorkspaceMemberMixin + admin role check
  — Calls LeaveService.approve_request()
  — Redirects to admin-leaves with success message

RejectLeaveView (POST /workspace/{id}/admin/leaves/{leave_id}/reject/)
  — WorkspaceMemberMixin + admin role check
  — review_notes from POST data — required
  — Calls LeaveService.reject_request()
  — Redirects to admin-leaves with success/error message
```

**Run checklist:** [Django New View](../../../../checklists/django/new-view.md) for each view

---

## 5. Templates

All templates must:
- Extend `base.html`
- Have NO `<style>` blocks (all CSS in `static/css/leaves.css`)
- Have NO inline `<script>` logic (all JS in `static/js/leaves.js`)
- Load static with `{% load static %}`

### `my_leaves.html`

```html
{% extends "base.html" %}
{% load static %}

{% block title %}My Leave Requests{% endblock %}

{% block content %}
  <!-- Filter form (GET method, ?status=...) -->
  <!-- Leave request table -->
  <!-- Status badges using CSS classes -->
  <!-- Cancel button as a POST form — wrapped in confirmation check via data-* -->
  <!-- Pagination links -->
{% endblock %}
```

**Status badge CSS classes:**
```css
/* in static/css/leaves.css */
.badge-submitted  { background: #3b82f6; color: white; }
.badge-approved   { background: #22c55e; color: white; }
.badge-rejected   { background: #ef4444; color: white; }
.badge-cancelled  { background: #9ca3af; color: white; }
```

### `submit_form.html`

```html
{% extends "base.html" %}

{% block content %}
  <h1>Submit Leave Request</h1>
  <form method="post">
    {% csrf_token %}
    <!-- Render form fields individually (not {{ form.as_p }}) -->
    <!-- Show field errors below each field -->
    <!-- Show non-field errors at top of form -->
    <!-- Submit button with loading state via data-* and JS -->
  </form>
{% endblock %}
```

### `admin_leaves.html`

```html
{% extends "base.html" %}

{% block content %}
  <!-- Filter form -->
  <!-- All requests table: Employee | Leave Type | Dates | Days | Status | Actions -->
  <!-- Approve: POST form button (no confirmation — low risk action) -->
  <!-- Reject: POST form button — opens an inline rejection reason form via JS -->
  <!-- Pagination -->
{% endblock %}
```

**Run checklist:** [Django Template](../../../../checklists/django/template.md) for each template

---

## 6. Static CSS (`static/css/leaves.css`)

```text
[ ] Status badge styles (.badge-submitted, .badge-approved, .badge-rejected, .badge-cancelled)
[ ] Leave type label styles
[ ] Table styles (striped rows, hover)
[ ] Form field styles (label above field, error message below)
[ ] Responsive: table scrolls horizontally on mobile
[ ] Filter bar styles
[ ] Pagination styles
[ ] NO hardcoded colors in HTML — all in CSS
[ ] CSS uses variables for colors: --color-success, --color-danger, etc.
```

---

## 7. Static JS (`static/js/leaves.js`)

```text
All JavaScript must use data-* attributes to read values — never string interpolation.

CANCEL CONFIRMATION:
<!-- HTML -->
<button
  class="btn-cancel-leave"
  data-leave-id="{{ leave.id }}"
  data-leave-type="{{ leave.get_leave_type_display }}"
  data-form-id="cancel-form-{{ leave.id }}"
>Cancel</button>
<form id="cancel-form-{{ leave.id }}" method="post"
      action="{% url 'cancel-leave' workspace.id leave.id %}" style="display:none;">
  {% csrf_token %}
</form>

// JS — reads from data-* attributes
document.querySelectorAll('.btn-cancel-leave').forEach(btn => {
  btn.addEventListener('click', () => {
    const leaveType = btn.dataset.leaveType;
    if (confirm(`Cancel your ${leaveType} request?`)) {
      document.getElementById(btn.dataset.formId).submit();
    }
  });
});

REJECT REASON INLINE FORM:
[ ] Clicking "Reject" shows an inline form (hidden div) asking for rejection reason
[ ] Reason text passed as a hidden input in the reject POST form
[ ] All shown/hidden via CSS class toggle — no inline style manipulation
```

---

## URL Configuration

```python
urlpatterns = [
    path('workspace/<int:workspace_id>/my-leaves/',
         MyLeavesView.as_view(), name='my-leaves'),
    path('workspace/<int:workspace_id>/my-leaves/submit/',
         SubmitLeaveView.as_view(), name='submit-leave'),
    path('workspace/<int:workspace_id>/my-leaves/<int:leave_id>/cancel/',
         CancelLeaveView.as_view(), name='cancel-leave'),
    path('workspace/<int:workspace_id>/admin/leaves/',
         AdminLeavesView.as_view(), name='admin-leaves'),
    path('workspace/<int:workspace_id>/admin/leaves/<int:leave_id>/approve/',
         ApproveLeaveView.as_view(), name='approve-leave'),
    path('workspace/<int:workspace_id>/admin/leaves/<int:leave_id>/reject/',
         RejectLeaveView.as_view(), name='reject-leave'),
]
```

---

## What You Should NOT Do

```text
× Put business logic (approve/reject/cancel validation) in a view — use LeaveService
× Add <style> blocks to any template — all CSS in static/css/leaves.css
× Add <script> blocks with logic to templates — all JS in static/js/leaves.js
× Pass data to JavaScript via {{ variable }} inside a <script> tag — use data-* attributes
× Not extend base.html in every template
× Let the cancel view succeed for someone else's leave request
× Let the admin views be accessible to a member-role user (check role, raise PermissionDenied)
× Skip CSRF token on any POST form
× Render form errors with just {{ form.as_p }} — render each field individually
× Overlap validation missing — two approved requests for same dates must be blocked
```

---

## Checklists to Run (in order)

```text
[ ] Django Model — LeaveRequest model
[ ] Django Form / Serializer — LeaveRequestForm
[ ] Django New View — each of the 5 views
[ ] Django Template — my_leaves.html, submit_form.html, admin_leaves.html
[ ] Permissions & Role-Based UI — admin views inaccessible to members
[ ] Error Handling — form errors shown per field, service errors shown at top
[ ] Delete & Destructive Actions — cancel confirmation
[ ] Notifications & Toasts — success/error messages after redirect (Django messages framework)
```

---

## Done When

```text
[ ] LeaveRequest model: all fields, constraints, indexes, migration
[ ] LeaveRequestForm: start_date >= today, end_date >= start_date, both validated
[ ] LeaveService: submit checks overlapping requests, approve/reject check status
[ ] LeaveService: cancel checks actor == employee AND status == 'submitted'
[ ] WorkspaceMemberMixin: used in all views
[ ] Admin views: PermissionDenied (403) if caller is not admin/owner
[ ] All POST views: redirect on success (POST→redirect→GET pattern)
[ ] All templates: extend base.html, no inline styles, no inline scripts
[ ] Leaves CSS: all badge classes defined with CSS variables
[ ] Leaves JS: cancel uses data-* attributes, no string interpolation in script
[ ] Reject: inline reason form shown via JS class toggle, reason passed as hidden input
[ ] Filter forms: GET method, filter results from DB (not client-side)
[ ] Status badges: correct CSS class applied based on leave status
[ ] Pagination: works on my-leaves and admin pages
[ ] Django messages: success/error messages shown after redirect
[ ] Migration generated and reviewed
[ ] Mobile: table scrolls horizontally on small screens
[ ] CSRF token present on all POST forms
```
