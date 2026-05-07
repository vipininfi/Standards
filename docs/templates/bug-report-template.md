# Bug Report Template

Use this when reporting a bug, blocker, or unexpected behavior found during work. Post it as a comment in the ticket — do not keep it only in chat.

---

## Bug Found During Work

```text
Issue Found:
[Describe the bug clearly — what is happening vs what should happen]

Impact:
[Who is affected? What breaks? What is the risk?]

Steps to Reproduce:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Expected Behavior:
[What should happen]

Actual Behavior:
[What is actually happening]

Suggested Fix:
[Optional — your best guess at what caused it and how to fix it]

Screenshot/Video:
[Attach if UI-related]
```

---

## Blocker Report

```text
Status: Blocked

Blocked On:
[What is blocking progress]

What I Checked:
- [Step or thing checked]
- [Step or thing checked]

Possible Cause:
[Your best assessment]

What I Need:
[What is required to unblock — API response, design clarification, access, etc.]
```

---

## Related Issue Found (outside current task scope)

```text
Related Issue Found:
[Describe the issue found while working on another task]

Affects:
[Which page/module/component]

Impact:
[What could break or fail]

Suggested Action:
[Create separate ticket / needs discussion / can be fixed in same PR]
```

---

## Example — Bug Found

```text
Issue Found:
The Edit Project form does not validate the phone number field, but the Add Project form does.
Users can save any text in the phone field through the edit flow.

Impact:
Invalid phone numbers are stored in the database. This could break SMS/notification features downstream.

Steps to Reproduce:
1. Open any project.
2. Click Edit.
3. Enter "abc123" in the phone number field.
4. Click Save — it saves successfully with no validation error.

Expected Behavior:
Phone field should validate the country code and number format, same as Add Project.

Actual Behavior:
No validation on Edit — any text is accepted.

Suggested Fix:
Use the same shared validation schema for both Add and Edit forms.
```
