# Project — Expense Tracker

**Stack:** React (TypeScript) + Django REST Framework
**Estimated Time:** 8 hours
**Difficulty:** Intermediate–Advanced

---

## What You Are Building

A workspace expense tracker with an approval workflow. Employees submit expense records with a receipt file. Managers (admin role) approve or reject submissions. The system has two views: "My Expenses" for employees and "Review Expenses" for managers.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Django: Expense model + migration | 45m |
| 2 | Django: ExpenseSerializer + ExpenseWriteSerializer | 30m |
| 3 | Django: ExpenseService (submit, approve, reject, delete) | 45m |
| 4 | Django: DRF views + URLs + file upload handling | 1h |
| 5 | React: My Expenses page (list + submit form) | 1h 30m |
| 6 | React: Submit expense form with file upload | 1h |
| 7 | React: Manager review page (approve/reject) | 1h |
| 8 | Permissions + all states | 30m |

---

## 1. Django Model

### `Expense`

```python
class Expense(models.Model):

    class Status(models.TextChoices):
        SUBMITTED = 'submitted', 'Submitted'
        APPROVED  = 'approved',  'Approved'
        REJECTED  = 'rejected',  'Rejected'

    class Category(models.TextChoices):
        TRAVEL      = 'travel',      'Travel'
        MEALS       = 'meals',       'Meals'
        SOFTWARE    = 'software',    'Software'
        EQUIPMENT   = 'equipment',   'Equipment'
        OTHER       = 'other',       'Other'

    workspace    = models.ForeignKey('Workspace', on_delete=models.CASCADE, related_name='expenses')
    title        = models.CharField(max_length=200)
    amount       = models.DecimalField(max_digits=10, decimal_places=2)
    category     = models.CharField(max_length=20, choices=Category.choices)
    description  = models.TextField(blank=True, default='')
    receipt      = models.FileField(upload_to='receipts/%Y/%m/', null=True, blank=True)
    status       = models.CharField(max_length=20, choices=Status.choices,
                                    default=Status.SUBMITTED)
    submitted_by = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                     null=True, related_name='expenses')
    reviewed_by  = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                     null=True, blank=True, related_name='reviewed_expenses')
    reviewed_at  = models.DateTimeField(null=True, blank=True)
    review_notes = models.TextField(blank=True, default='')
    created_at   = models.DateTimeField(auto_now_add=True)
    updated_at   = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = 'Expense'
        verbose_name_plural = 'Expenses'
        constraints = [
            models.CheckConstraint(
                check=models.Q(amount__gt=0),
                name='expense_amount_positive'
            )
        ]

    def __str__(self):
        return f"{self.submitted_by} — {self.title} (${self.amount})"
```

**Run checklist:** [Django Model](../../../../checklists/django/model.md)

---

## 2. Serializers

### `ExpenseSerializer` (read)

```python
fields = ['id', 'title', 'amount', 'category', 'description',
          'receipt_url', 'status', 'submitted_by', 'reviewed_by',
          'reviewed_at', 'review_notes', 'created_at']
read_only_fields = ['id', 'status', 'submitted_by', 'reviewed_by',
                    'reviewed_at', 'created_at']

# receipt_url: SerializerMethodField — returns the file URL or None
```

### `ExpenseWriteSerializer` (write)

```python
fields = ['title', 'amount', 'category', 'description', 'receipt']

# Validation:
# - title: required, max 200 chars
# - amount: required, must be > 0, max 2 decimal places
# - category: required, must be a valid choice
# - receipt: optional, MIME type must be image/jpeg, image/png, or application/pdf, max 10 MB
```

**Run checklist:** [Django Form / Serializer](../../../../checklists/django/form-serializer.md)

---

## 3. ExpenseService

```python
class ExpenseService:

    @staticmethod
    def submit_expense(workspace, title, amount, category,
                       submitted_by, description='', receipt=None):
        """Creates a new expense in 'submitted' status."""

    @staticmethod
    def approve_expense(expense, reviewed_by, review_notes=''):
        """
        Approves the expense.
        Raises ValidationError if status is not 'submitted'.
        Sets reviewed_by, reviewed_at, status = 'approved'.
        """

    @staticmethod
    def reject_expense(expense, reviewed_by, review_notes):
        """
        Rejects the expense.
        Raises ValidationError if status is not 'submitted'.
        review_notes is required for rejection.
        Sets reviewed_by, reviewed_at, review_notes, status = 'rejected'.
        """

    @staticmethod
    def delete_expense(expense, actor):
        """
        Deletes the expense and its receipt file from storage.
        Only the submitter or an admin can delete.
        Cannot delete an approved expense.
        """
```

---

## 4. DRF Views & URLs

### Views

```text
ExpenseListCreateView
  GET  /api/v1/workspaces/{workspace_id}/expenses/
    — Returns current user's own expenses
    — Filters: ?status=submitted, ?category=travel
    — Pagination: 20 per page

  POST /api/v1/workspaces/{workspace_id}/expenses/
    — Uses multipart/form-data (supports file upload)
    — Calls ExpenseService.submit_expense()
    — Returns 201

ExpenseDetailView
  GET    /api/v1/workspaces/{workspace_id}/expenses/{id}/
  DELETE /api/v1/workspaces/{workspace_id}/expenses/{id}/
    — Only submitter or admin can delete
    — Cannot delete approved expense (409)
    — Returns 204

ExpenseApproveView
  POST /api/v1/workspaces/{workspace_id}/expenses/{id}/approve/
    — Admin only
    — Calls ExpenseService.approve_expense()
    — Returns 200

ExpenseRejectView
  POST /api/v1/workspaces/{workspace_id}/expenses/{id}/reject/
    — Admin only
    — Body: { review_notes: "..." } — required
    — Calls ExpenseService.reject_expense()
    — Returns 200

AdminExpenseListView
  GET /api/v1/workspaces/{workspace_id}/expenses/all/
    — Admin only
    — Returns ALL workspace expenses
    — Filters: ?status=submitted&submitted_by={user_id}
    — Pagination: 25 per page
```

**Run checklist:** [DRF API View](../../../../checklists/django/api-view.md)

---

## 5. React: My Expenses Page

**URL:** `/workspaces/{id}/expenses`

```text
HEADER:
[ ] "My Expenses" title + "Submit Expense" button

FILTERS:
[ ] Status: All / Submitted / Approved / Rejected
[ ] Category: All / Travel / Meals / Software / Equipment / Other
[ ] Filter state in URL: ?status=submitted&category=travel

TABLE COLUMNS:
[ ] Title · Amount (right-aligned, 2dp) · Category · Status badge · Submitted On · Actions

STATUS BADGES:
[ ] submitted=blue, approved=green, rejected=red

ACTIONS:
[ ] View receipt link (if receipt exists)
[ ] Delete button — only shown when status = 'submitted' (cannot delete approved)

PAGINATION: 20 per page

STATES:
[ ] Loading: skeleton rows
[ ] Empty: "No expenses yet." + "Submit your first expense" button
[ ] Empty (filter): "No expenses match your filters." + clear button
[ ] Error: "Could not load expenses." + retry
```

**Run checklist:** [React Page Build](../../../../checklists/website/react-page.md) · [Tables & Data Lists](../../../../checklists/frontend/tables-data-lists.md)

---

## 6. Submit Expense Form (Modal)

```text
Fields:
[ ] Title: text, required, max 200 chars
[ ] Amount: number input, required, min 0.01, 2 decimal places
[ ] Category: dropdown, required
[ ] Description: textarea, optional
[ ] Receipt: file upload — JPG, PNG, PDF only, max 10 MB

File Upload:
[ ] Show accepted types: "JPG, PNG, or PDF up to 10 MB"
[ ] Validate MIME type before submitting (not just extension)
[ ] Show file size error if > 10 MB before upload
[ ] Preview filename + size after selection
[ ] Upload sent as multipart/form-data

Behavior:
[ ] Loading state on submit, button disabled
[ ] On success: "Expense submitted." toast, list refreshes
[ ] On error: banner above submit button (field errors below fields)
[ ] Validation: amount > 0, category selected, file MIME valid
```

**Run checklist:** [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## 7. React: Manager Review Page

**URL:** `/workspaces/{id}/expenses/review`

```text
[ ] Visible only to admin/owner — 403 state for members
[ ] Default filter: status=submitted (pending review)
[ ] Filters: Status · Employee (dropdown of workspace members) · Category
[ ] Filter chips with X + "Clear all" button

TABLE COLUMNS:
[ ] Employee Name · Title · Amount · Category · Submitted On · Status badge · Actions

ACTIONS (when status = 'submitted'):
[ ] "Approve" (green) · "Reject" (red)

Approve flow:
[ ] Confirmation: "Approve this expense?" + optional note input
[ ] On confirm: calls /approve endpoint, row updates in place

Reject flow:
[ ] Modal: "Reject Expense" with required review_notes textarea
[ ] On submit: calls /reject endpoint, row updates in place
[ ] review_notes required — cannot reject without a reason

After action:
[ ] Approve/Reject buttons replaced by the result badge
[ ] Toast: "Expense approved." / "Expense rejected."

STATES: Loading · Empty · Empty (filter) · Error
```

**Run checklist:** [Permissions & Role-Based UI](../../../../checklists/frontend/permissions-role-ui.md) · [Search, Filters & Pagination](../../../../checklists/frontend/search-filters-pagination.md)

---

## What You Should NOT Do

```text
× Put business logic (approve, reject, delete) in DRF views — use ExpenseService
× Accept receipt file without MIME type validation (check actual MIME, not extension)
× Let a member-role user access /expenses/all/ or approve/reject endpoints
× Let a user delete an approved expense
× Skip the receipt file cleanup when deleting an expense
× Use DecimalField with max_digits < what's needed (use max_digits=10, decimal_places=2)
× Return 200 for a submitted expense (new resource) — use 201
× Accept any file type — only JPG, PNG, PDF
```

---

## Checklists to Run (in order)

```text
[ ] Django Model — Expense model
[ ] Django Form / Serializer — both serializers + file field validation
[ ] DRF API View — all views
[ ] React Page Build — My Expenses page + Manager Review page
[ ] Tables & Data Lists — both tables
[ ] Modals & Dialogs — submit expense modal, approve/reject modals
[ ] Search, Filters & Pagination — both pages
[ ] Loading States & Skeletons — both pages
[ ] Error Handling — all API calls
[ ] Delete & Destructive Actions — delete expense
[ ] Notifications & Toasts — approve/reject/submit/delete
[ ] Permissions & Role-Based UI — manager page access, delete visibility
```

---

## Done When

```text
[ ] Expense model: all fields, CheckConstraint for amount > 0, migration
[ ] ExpenseWriteSerializer: MIME type validated (not just extension), amount > 0 validated
[ ] ExpenseService: submit, approve, reject, delete all work correctly
[ ] delete_expense: removes file from storage AND database record
[ ] Approve endpoint: admin only, 409 if not 'submitted' status
[ ] Reject endpoint: admin only, review_notes required, 409 if not 'submitted'
[ ] Delete endpoint: submitter or admin only; 409 if status = 'approved'
[ ] AdminExpenseListView: admin only, 403 for member role
[ ] My Expenses: filters by current user only (not all workspace expenses)
[ ] My Expenses: all 4 states (loading/empty/filter-empty/error)
[ ] Submit form: MIME type checked, 10 MB limit enforced before upload
[ ] Submit form: multipart/form-data, file included in request
[ ] Delete: only shown for 'submitted' expenses — not approved
[ ] Manager page: 403 state for member-role React users
[ ] Manager page: reject requires review_notes (shown as required field in modal)
[ ] After approve/reject: buttons replaced by badge in table
[ ] Receipt files: served correctly, URL included in serializer
[ ] TypeScript: no any types, Expense interface defined
[ ] Mobile tested at 375px and 768px
```
