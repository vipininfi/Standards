# Completion Comment Template

Use this when marking a task as done. Post it as a final comment in the ticket.

---

## Standard Completion Comment

```text
Status: Completed

Work Done:
- [What was built]
- [What was fixed]
- [What was updated]

Testing Done:
- [What success flows were tested]
- [What validation/error cases were tested]
- [Mobile checked: Yes / No]

Notes:
- [Known issues, follow-up tickets, or "None"]
```

---

## Example — Feature Completion

```text
Status: Completed

Work Done:
- Built shared Add/Edit Project form component.
- Added required field validation (name, workspace, phone).
- Added loading state on submit button.
- Added success and error notifications.
- Added delete confirmation modal.
- Matched design system spacing and button styles.

Testing Done:
- Tested create project success flow.
- Tested edit project success flow.
- Tested missing name field validation.
- Tested invalid phone number validation.
- Tested API failure error message.
- Verified Add and Edit UI are identical.
- Checked mobile layout.

Notes:
- No known blockers.
- Bulk delete is out of scope for this task — separate ticket needed.
```

---

## Example — Bug Fix Completion

```text
Status: Completed

Work Done:
- Fixed phone number validation missing on Edit Project form.
- Shared the same validation schema between Add and Edit forms.
- Added E.164 format validation with country code selector.

Testing Done:
- Tested valid phone number saves correctly.
- Tested invalid text input shows validation error.
- Tested missing phone number shows required field error.
- Confirmed Add and Edit now behave identically.
- Checked mobile layout.

Notes:
- No known blockers.
```

---

## Rules

- Never write just "Done" — always include what was done and what was tested.
- If there is a known issue that is out of scope, mention it with a note to create a separate ticket.
- Always include whether mobile was checked.
- If testing could not be completed for a specific case, mention why.
