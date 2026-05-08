# Project — Announcement Board

**Stack:** React (TypeScript) + Django REST Framework
**Estimated Time:** 8 hours
**Difficulty:** Intermediate

---

## What You Are Building

A workspace announcement board. Admins post announcements to the whole workspace. All members can view them. Announcements can be pinned to the top, set to expire on a date, and deleted. The list is paginated and shows pinned announcements first.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Django: Announcement model + migration | 45m |
| 2 | Django: AnnouncementSerializer + AnnouncementWriteSerializer | 30m |
| 3 | Django: AnnouncementService (create, pin, delete) | 30m |
| 4 | Django: DRF views + URLs + permission class | 1h |
| 5 | React: Announcement list page (skeleton, empty, error) | 1h 30m |
| 6 | React: Create/Edit announcement modal (shared component) | 1h 30m |
| 7 | React: Pin toggle + delete (admin only) | 30m |
| 8 | Permissions + all states | 30m |

---

## 1. Django Model

### `Announcement`

```python
class Announcement(models.Model):
    workspace   = models.ForeignKey('Workspace', on_delete=models.CASCADE, related_name='announcements')
    title       = models.CharField(max_length=200)
    body        = models.TextField()
    is_pinned   = models.BooleanField(default=False)
    expires_at  = models.DateTimeField(null=True, blank=True)
    created_by  = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                    null=True, related_name='announcements')
    created_at  = models.DateTimeField(auto_now_add=True)
    updated_at  = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-is_pinned', '-created_at']
        verbose_name = 'Announcement'
        verbose_name_plural = 'Announcements'

    def __str__(self):
        return f"{self.workspace.name} / {self.title}"
```

**Rules:**
- Pinned announcements always appear at the top (ordering by `-is_pinned` first)
- `expires_at` is optional — if set, expired announcements are excluded from member views

**Run checklist:** [Django Model](../../../../checklists/django/model.md)

---

## 2. Serializers

### `AnnouncementSerializer` (read — response)

```python
fields = ['id', 'title', 'body', 'is_pinned', 'expires_at',
          'created_by', 'created_at', 'updated_at', 'is_expired']
read_only_fields = ['id', 'created_by', 'created_at', 'updated_at']

# is_expired: SerializerMethodField — returns True if expires_at < now()
```

### `AnnouncementWriteSerializer` (write — create/update)

```python
fields = ['title', 'body', 'is_pinned', 'expires_at']

# Validation:
# - title: required, max 200 chars
# - body: required
# - expires_at: if provided, must be in the future
```

**Run checklist:** [Django Form / Serializer](../../../../checklists/django/form-serializer.md)

---

## 3. AnnouncementService

```python
# services.py

class AnnouncementService:

    @staticmethod
    def create_announcement(workspace, title, body, created_by,
                            is_pinned=False, expires_at=None):
        """
        Creates a new announcement.
        Raises ValidationError if title or body is blank.
        """

    @staticmethod
    def toggle_pin(announcement):
        """
        Toggles is_pinned on the announcement and saves.
        Returns the updated announcement.
        """

    @staticmethod
    def delete_announcement(announcement):
        """
        Deletes the announcement.
        """
```

No business logic in the view. Views call service methods.

---

## 4. DRF Views & URLs

### Permission Class: `IsWorkspaceMember`

```python
# permissions.py
class IsWorkspaceMember(BasePermission):
    def has_permission(self, request, view):
        workspace_id = view.kwargs.get('workspace_id')
        return WorkspaceMember.objects.filter(
            workspace_id=workspace_id, user=request.user, status='active'
        ).exists()
```

### `AnnouncementListCreateView`

```text
GET  /api/v1/workspaces/{workspace_id}/announcements/
  — Returns active (non-expired) announcements, pinned first
  — Pagination: 20 per page
  — For admins: include expired announcements (add ?include_expired=true)

POST /api/v1/workspaces/{workspace_id}/announcements/
  — Admin only
  — Calls AnnouncementService.create_announcement()
  — Returns 201 with created announcement
```

### `AnnouncementDetailView`

```text
GET    /api/v1/workspaces/{workspace_id}/announcements/{id}/
PATCH  /api/v1/workspaces/{workspace_id}/announcements/{id}/  — admin only
DELETE /api/v1/workspaces/{workspace_id}/announcements/{id}/  — admin only, returns 204
```

### `AnnouncementPinView`

```text
POST /api/v1/workspaces/{workspace_id}/announcements/{id}/pin/
  — Admin only
  — Calls AnnouncementService.toggle_pin()
  — Returns 200 with updated announcement
```

### URL names (kebab-case)

```python
name='announcement-list'
name='announcement-detail'
name='announcement-pin'
```

**Run checklist:** [DRF API View](../../../../checklists/django/api-view.md)

---

## 5. React: Announcement List Page

**URL:** `/workspaces/{id}/announcements`

```text
[ ] Pinned announcements shown at top with a 📌 pin indicator
[ ] Each announcement card shows: title, body preview (3 lines, truncated), created by, date
[ ] Clicking a card expands it to show full body
[ ] Admin actions on each card: Pin/Unpin toggle · Edit · Delete — hidden for member role
[ ] "New Announcement" button at top right — admin only
[ ] Pagination: 20 per page ("Load more" style acceptable)

TypeScript:
  interface Announcement {
    id: number;
    title: string;
    body: string;
    is_pinned: boolean;
    is_expired: boolean;
    expires_at: string | null;
    created_by: { id: number; name: string };
    created_at: string;
  }

Custom hook: useAnnouncements(workspaceId) → { data, isLoading, error, refetch }

STATES:
[ ] Loading: 3 skeleton cards (matching card shape)
[ ] Empty: "No announcements yet." + "Post your first announcement" (admin only)
[ ] Error: "Could not load announcements." + retry button
```

**Run checklist:** [React Page Build](../../../../checklists/website/react-page.md) · [Loading States & Skeletons](../../../../checklists/frontend/loading-states-skeletons.md)

---

## 6. Create / Edit Announcement Modal

One shared `AnnouncementModal` component.

### Props

```ts
interface AnnouncementModalProps {
  mode: 'add' | 'edit';
  announcement?: Announcement;  // provided in edit mode
  workspaceId: number;
  onSuccess: () => void;
  onClose: () => void;
}
```

### Form Fields

| Field | Type | Rules |
|-------|------|-------|
| Title | text input | Required, max 200 chars |
| Body | textarea | Required |
| Pin this announcement | checkbox | Default: unchecked |
| Expires on | date-time picker | Optional, must be future |

### Behavior

```text
ADD MODE: title "New Announcement", submit "Post Announcement"
EDIT MODE: title "Edit Announcement", pre-filled, submit "Save Changes"

[ ] Loading state on submit button, fields disabled
[ ] Validation errors shown below each field
[ ] API error shown in a banner above submit (not toast)
[ ] On success: close modal, call onSuccess (triggers list refetch), "Announcement posted." toast
[ ] Escape + X close — prompt "Discard changes?" if dirty
```

**Run checklist:** [Add/Edit Consistency](../../../../checklists/frontend/add-edit-consistency.md) · [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## 7. Pin & Delete

### Pin Toggle

```text
[ ] Pin/Unpin button on each card (admin only)
[ ] Calls POST /announcements/{id}/pin/
[ ] Optimistic UI: pin indicator toggles immediately, reverts on error
[ ] Loading: button spinner during API call
[ ] Toast: "Announcement pinned." / "Announcement unpinned."
```

### Delete

```text
[ ] Delete button on card (admin only)
[ ] Confirmation dialog:
    Title:  "Delete Announcement?"
    Body:   "'[Title]' will be permanently deleted."
    Buttons: Cancel · "Delete" (red)
[ ] On confirm: loading state, card removed on success
[ ] Toast: "Announcement deleted."
```

**Run checklist:** [Delete & Destructive Actions](../../../../checklists/frontend/delete-destructive-actions.md) · [Notifications & Toasts](../../../../checklists/frontend/notifications-toasts.md)

---

## Error Handling

```text
[ ] All API errors parsed: code → user-friendly message
[ ] 403 errors: "You don't have permission to do this."
[ ] 404 errors: "This announcement was not found."
[ ] Network errors: "Connection lost. Please try again."
[ ] Never show raw API error strings to user
[ ] No catch (e) {} empty blocks
```

**Run checklist:** [Error Handling](../../../../checklists/frontend/error-handling.md)

---

## Permissions Summary

```text
| Action | Member | Admin | Owner |
|--------|--------|-------|-------|
| View announcements | ✓ | ✓ | ✓ |
| Create announcement | ✗ | ✓ | ✓ |
| Edit announcement | ✗ | ✓ | ✓ |
| Pin announcement | ✗ | ✓ | ✓ |
| Delete announcement | ✗ | ✓ | ✓ |

[ ] New Announcement button hidden for member role
[ ] Edit, Pin, Delete actions hidden per card for member role
[ ] Django: permission class checks workspace membership AND role for write operations
```

---

## What You Should NOT Do

```text
× Put create/pin/delete logic in the DRF view — delegate to AnnouncementService
× Use fields = '__all__' in any serializer
× Return 200 for a newly created announcement — use 201
× Show Create/Edit/Delete to member-role React users
× Forget to check if expires_at is in the future during validation
× Show expired announcements to members (filter on backend, not frontend)
× Let the Delete work without a confirmation dialog
```

---

## Checklists to Run (in order)

```text
[ ] Django Model — Announcement model
[ ] Django Form / Serializer — both serializers
[ ] DRF API View — list/create view, detail view, pin view
[ ] React Page Build — announcement list page
[ ] Add/Edit Consistency — AnnouncementModal
[ ] Modals & Dialogs — AnnouncementModal
[ ] Loading States & Skeletons — list page
[ ] Error Handling — all API calls
[ ] Delete & Destructive Actions — delete announcement
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — all admin-only actions
```

---

## Done When

```text
[ ] Announcement model: all fields, ordering, __str__, migration
[ ] Serializers: read-only fields set, expires_at future validation, is_expired computed
[ ] AnnouncementService: create, toggle_pin, delete — no logic in views
[ ] IsWorkspaceMember permission class queries database (not just JWT)
[ ] List endpoint: non-expired announcements for members, pinned first, paginated
[ ] Create endpoint: admin only (403 for member), returns 201
[ ] Pin endpoint: toggles is_pinned, returns 200
[ ] Delete endpoint: returns 204
[ ] React list: useAnnouncements hook with { data, isLoading, error, refetch }
[ ] React list: all 3 states (loading skeletons / empty / error with retry)
[ ] AnnouncementModal: single component, pre-fill in edit mode works
[ ] AnnouncementModal: loading state on submit, fields disabled
[ ] Pin toggle: optimistic UI, reverts on error
[ ] Delete: confirmation dialog required, 204 response removes card from list
[ ] TypeScript: no any types, Announcement interface defined
[ ] Admin actions (Create/Edit/Pin/Delete) hidden for member-role React users
[ ] Mobile tested at 375px and 768px
[ ] All errors shown in UI — no silent failures
```
