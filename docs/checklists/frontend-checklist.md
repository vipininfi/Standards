# Frontend Developer Checklist

Run this before marking any frontend task as complete.

---

## Functionality

```text
[ ] Main success flow works end-to-end.
[ ] All user actions produce the expected result.
[ ] API is called with the correct payload.
[ ] API response is handled correctly.
[ ] Success state is shown after action completes.
[ ] Empty state is handled (no blank/broken screens when no data).
```

## Forms

```text
[ ] Every field has a visible label.
[ ] Required fields have a star (*).
[ ] Required fields have validation that fires on submit.
[ ] Field-level errors are shown near the invalid field.
[ ] Form values are NOT cleared when the API call fails.
[ ] Success message or action is shown after successful submit.
[ ] Submit button is disabled and shows loading state during request.
[ ] Duplicate/conflict error is shown if backend returns 409.
```

## Error Handling

```text
[ ] API error is shown in the UI (not just console.log).
[ ] 401 UNAUTHENTICATED: redirects to login or shows session expired.
[ ] 403 FORBIDDEN: shows permission error, hides or disables action.
[ ] 422 VALIDATION_ERROR: shows field-level errors.
[ ] 500 INTERNAL_ERROR: shows retry message.
[ ] No silent failures anywhere.
[ ] Error messages are user-friendly (not raw API error codes).
```

## Loading States

```text
[ ] Submit button shows "Loading..." or spinner during API call.
[ ] Submit button is disabled during API call.
[ ] Table/list shows skeleton or loader while fetching data.
[ ] Delete button shows "Deleting..." during delete action.
[ ] Any async action shows visual feedback while in progress.
```

## Consistency

```text
[ ] If this is an Edit form — it matches the Add form for the same entity.
[ ] Same component is used for Add and Edit (not duplicated).
[ ] Same validation schema is used for Add and Edit.
[ ] UI follows the project design system (colors, spacing, button styles).
[ ] Page feels consistent with other similar pages in the product.
```

## Permissions & Access

```text
[ ] Correct actions are hidden or disabled for unauthorized roles.
[ ] Viewer role cannot see/trigger write actions.
[ ] Non-members are not exposed to other workspace data.
[ ] Permission-denied state is handled gracefully (not just blank).
```

## Mobile

```text
[ ] Mobile layout is checked (360px–414px width).
[ ] Tablet layout is checked (768px).
[ ] Forms are full-width on mobile.
[ ] Buttons are touch-friendly size.
[ ] No text overflow or broken alignment on small screens.
[ ] Mobile menu/nav works if affected.
```

## Code Quality

```text
[ ] No duplicate form/component code (Add and Edit share a component).
[ ] No hardcoded values that should come from config or constants.
[ ] No console.log statements left in production code.
[ ] No commented-out dead code blocks left in.
[ ] Component is reused if the same UI exists elsewhere.
```

## Ticket

```text
[ ] Ticket is updated with what was done and what was tested.
[ ] Screenshots or video attached if UI-related.
[ ] Any known issues or out-of-scope items noted.
[ ] Completion comment added before marking done.
```

---

## Before Marking Done — Final Check

Ask yourself:

```text
Would a normal user understand every error shown?
Can anything fail silently without the user knowing?
Is this consistent with how similar pages work?
Did I test it on mobile?
Did I test it as different user roles?
Is my ticket updated with evidence?
```
