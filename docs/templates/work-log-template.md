# Work Log Template

Use this when posting status updates in a ticket.

---

## In Progress Update

```text
Time Spent: [e.g. 1h 20m]

Status: In Progress

Work Done:
- [What was completed so far]
- [What was fixed or updated]

Issue:
- [Any problem encountered, or "None"]

Next:
- [What will be done next]
```

---

## Completed Update

```text
Time Spent: [e.g. 2h]

Status: Completed

Work Done:
- [What was built or fixed]
- [What was tested]
- [What was updated]
```

---

## Blocked Update

```text
Time Spent: [e.g. 45m]

Status: Blocked

Work Done:
- [What was attempted and checked]

Blocker:
- [What is blocking progress]
- [What is needed to unblock]
```

---

## Work Log Line (for time tracking)

```text
[Duration] — [What was done]. [Reason if it took extra time.]
```

Examples:

```text
1h 30m — Built Add/Edit form validation and tested required fields.

45m — Fixed mobile spacing issue in project card component.

2h — Debugged API failure; issue was caused by missing workspace_id in payload.

2h 15m — Fixed form submission issue. Extra time required because API was silently failing without a proper error response.
```

---

## Rules for Work Logs

- Include time spent, status, what was done, and next step.
- Keep it short — no long stories.
- When something took extra time, mention the reason briefly.
- Never write vague logs like "Worked on task" or "Fixed issues."
