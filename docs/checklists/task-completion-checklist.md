# Task Completion Checklist

Run this before marking **any** task done — regardless of role.

---

## Understanding

```text
[ ] The full requirement was read and understood before starting.
[ ] Any unclear requirements were clarified in the ticket before building.
[ ] The correct scope was built — not more, not less.
```

## Building

```text
[ ] The work meets the requirement as described.
[ ] No shortcuts were taken that could cause silent failures.
[ ] Any shared components or logic were updated carefully with impact checked.
[ ] The code/build follows the project's standards (UI, API, naming).
```

## Self-Testing (Do Not Skip)

```text
[ ] The main success flow was tested and works.
[ ] The main error/failure case was tested and shows correctly in UI.
[ ] Required field validation was tested.
[ ] Mobile layout was checked.
[ ] Similar/related pages that may be affected were checked.
[ ] No silent failure exists anywhere in the work.
[ ] No console.log errors in the browser.
```

## Ticket Update

```text
[ ] Ticket was updated with what was done (not just "Done").
[ ] Time spent was logged.
[ ] Testing done was mentioned in the completion comment.
[ ] Any known issues or out-of-scope items were noted.
[ ] Screenshots or video were attached if the task was UI-related.
[ ] Any related issues found were raised as separate tickets or comments.
```

## Final Gate

Before marking done, answer these honestly:

```text
Would I be confident showing this to the client right now?
Is there anything that could silently fail that I have not handled?
Did I test it the way a real user would use it?
Is the ticket updated so someone else could understand what was done without asking me?
```

If any answer is "no" — do not mark done yet.

---

## Completion Comment

Always end with this in the ticket:

```text
Status: Completed

Work Done:
- [List what was built, fixed, or updated]

Testing Done:
- [List what was tested]
- Mobile checked: Yes / No

Notes:
- [Known issues for follow-up, or "None"]
```
