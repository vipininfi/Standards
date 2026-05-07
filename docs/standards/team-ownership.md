# Team Ownership & Process Standards

Core rule:

> Every team member must own their work end-to-end: understand the requirement, ask questions early, build with standards, test their own work, update clearly, raise issues in the ticket, and stay accountable for changes. Before marking things Done — meet the standards.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-self-ownership-standard) | Self-Ownership Standard |
| [2](#2-before-starting-any-task) | Before Starting Any Task |
| [3](#3-work-update-standard) | Work Update Standard |
| [4](#4-work-log-standard) | Work Log Standard |
| [5](#5-issue-raising-standard) | Issue Raising Standard |
| [6](#6-change-impact-standard) | Change Impact Standard |
| [7](#7-commenting--code-explanation-standard) | Commenting & Code Standard |
| [8](#8-accountability-for-bugs) | Accountability for Bugs |
| [9](#9-communication-standard) | Communication Standard |
| [10](#10-when-to-ask-for-help) | When to Ask for Help |
| [11](#11-ticket-discipline-standard) | Ticket Discipline |
| [12](#12-final-completion-standard) | Final Completion Standard |
| [13](#13-pull-request--code-review-standard) | Pull Request Standard |
| [14](#14-developer-mindset-standard) | Developer Mindset |
| [15](#15-non-negotiable-standards) | Non-Negotiables |
| [16](#16-team-standard-summary) | Team Standard Summary |

---

# 1. Self-Ownership Standard

## What Self-Ownership Means

Every team member is responsible for:

| Area | Expected Standard |
|------|-------------------|
| Understanding | Read the task properly before starting. |
| Clarification | Ask questions before building unclear requirements. |
| Research | Try to understand and solve independently first. |
| Accountability | Own your changes, bugs, and fixes. |
| Quality | Do not depend only on QA/client to find basic issues. |
| Documentation | Keep ticket updates clear and traceable. |
| Communication | Give concise updates with time and status. |
| Escalation | Ask for help when blocked after proper effort. |

## Good Ownership Behavior

```text
I reviewed the task, found that the add/edit form design needs to stay consistent across both pages.
I checked both flows, updated the shared component, and tested validation on create and edit.
```

## Poor Ownership Behavior

```text
Done.
```

or

```text
It was not mentioned in the ticket, so I did not check it.
```

If you are building something, you should think about the surrounding system — not only the exact line written in the ticket.

---

# 2. Before Starting Any Task

Before development starts, every member must check these points.

## Requirement Understanding Checklist

```text
[ ] What exactly needs to be built/fixed?
[ ] Which page/module does it affect?
[ ] Is this frontend, backend, database, no-code, API, or design work?
[ ] Are there similar existing pages/components?
[ ] Should this be reusable?
[ ] What validations are needed?
[ ] What error states are needed?
[ ] What loading states are needed?
[ ] What success/failure states are needed?
[ ] Does this affect any other page/API/component?
[ ] Is there any security or permission concern?
[ ] Is anything unclear that should be asked before starting?
```

## Rule

If something is unclear, do not assume silently.

Raise concise questions in the ticket:

```text
Need clarification before implementation:

1. Should this field be required or optional?
2. Should this be added to both Add and Edit forms?
3. Should users with Viewer role see this button disabled or hidden?
```

---

# 3. Work Update Standard

Updates must be clear, short, and useful.

## Update Must Include

| Item | Required |
|------|----------|
| What was done | Yes |
| Time spent | Yes |
| Current status | Yes |
| Issue/blocker if any | Yes |
| Next step | If still in progress |

## Standard Update Format

```text
Time Spent: 1h 20m

Status: In Progress

Work Done:
- Updated Add Project form layout.
- Added required field validation for name and phone.
- Matched UI spacing with Edit Project form.

Issue:
- Phone validation took extra time because country-code handling was missing.

Next:
- Testing mobile layout and error states.
```

## For Completed Work

```text
Time Spent: 2h

Status: Completed

Work Done:
- Added delete confirmation popup.
- Added loading state on delete button.
- Added success and error notifications.
- Tested delete success, permission error, and API failure cases.
```

## For Blocked Work

```text
Time Spent: 45m

Status: Blocked

Work Done:
- Checked current API integration.
- Verified frontend payload.
- Found backend is not returning field-level validation errors.

Blocker:
- Need backend response format confirmation before final UI error handling.
```

---

# 4. Work Log Standard

Work logs should be concise. They are not for long explanations.

## Good Work Log

```text
1h 30m — Built Add/Edit form validation and tested required fields.
```

```text
45m — Fixed mobile spacing issue in project card component.
```

```text
2h — Debugged API failure; issue was caused by missing workspace_id in payload.
```

## Bad Work Log

```text
Worked on task.
```

```text
Fixed issues.
```

```text
Did frontend.
```

## When Something Took More Time

Mention the reason briefly:

```text
2h 15m — Fixed form submission issue. Extra time was required because the API was silently failing without a proper error response.
```

Do not write long stories. Keep it clear and professional.

---

# 5. Issue Raising Standard

If you find a bug, blocker, unclear requirement, or related issue, raise it in the ticket/task system.

**Do not only mention it in chat.** Chat messages get lost. Tickets remain traceable.

## When to Raise an Issue

| Situation | Action |
|-----------|--------|
| Found a bug while working | Add comment in ticket. |
| Found related issue outside current scope | Create linked ticket or add note. |
| Requirement unclear | Ask in ticket. |
| API missing expected field | Raise issue in backend ticket. |
| UI inconsistency found | Mention in ticket with screenshot. |
| Another module may be affected | Add impact note. |

## Issue Format

```text
Issue Found:
The Edit Project form has different validation from Add Project form.

Impact:
Users can submit invalid phone numbers from Edit form.

Suggested Fix:
Use the same shared validation schema/component for both Add and Edit forms.
```

---

# 6. Change Impact Standard

Before changing anything, check where else it may affect.

## Impact Checklist

```text
[ ] Is this component reused anywhere?
[ ] Is this API used by another page?
[ ] Is this database field used in filters/search/export?
[ ] Is this validation also needed in edit flow?
[ ] Is this change affecting mobile layout?
[ ] Is this change affecting permissions?
[ ] Is this change affecting notifications?
[ ] Is this change affecting reports/analytics?
[ ] Is this change affecting old data?
```

## Good Change Note

```text
Impact Checked:
- Add Project page
- Edit Project page
- Project detail page
- Project list table
- API payload
- Form validation schema

Change applied through shared component to keep Add/Edit UI consistent.
```

## Bad Approach

Changing only one place without checking related pages.

Example:

```text
Added validation on Add form but forgot Edit form.
```

This creates inconsistent product behavior.

---

# 7. Commenting & Code Explanation Standard

Comments should explain **why**, not obvious **what**.

## Good Comment

```ts
// Keep submit button disabled during API request to prevent duplicate project creation.
```

## Bad Comment

```ts
// This is a button
```

## Ticket Comment Standard

Keep comments concise and useful:

```text
Updated ProjectForm to be shared by Add and Edit pages so validation and UI remain consistent.
```

---

# 8. Accountability for Bugs

If your change creates a bug, own it.

## Expected Behavior

```text
I found that my previous change affected the Edit flow. I have reopened the ticket, added the issue details, and I am fixing it now.
```

## Poor Behavior

```text
It was working on my side.
```

or

```text
QA should have checked it.
```

Ownership means fixing and improving the standard so the same bug does not happen again.

---

# 9. Communication Standard

Communication should be:

```text
Clear
Short
Specific
Traceable
Professional
```

## Bad Communication

```text
There is issue.
```

## Good Communication

```text
Issue:
The save button does not show loading state during API request.

Impact:
User can click multiple times and create duplicate records.

Fix:
I will disable the button while request is pending and show "Saving...".
```

---

# 10. When to Ask for Help

First, try to understand and debug yourself. But do not waste too much time silently.

## Escalation Rule

| Situation | Standard |
|-----------|----------|
| Requirement unclear | Ask immediately |
| Blocked by missing API/access | Ask immediately |
| Technical issue after reasonable debugging | Ask with details |
| Security/payment/auth issue | Ask early |
| Production issue | Escalate immediately |

## Help Request Format

```text
Need help on:
Project creation API returning 403.

What I checked:
- User is logged in.
- Token is being sent.
- workspace_id is correct.
- Same user can view workspace.

Possible Cause:
RLS insert policy may not allow manager role.

Request:
Can backend confirm insert policy for manager role?
```

---

# 11. Ticket Discipline Standard

All important information must be in the ticket.

## Ticket Must Contain

```text
[ ] Requirement
[ ] Clarifying questions
[ ] Work updates
[ ] Issues found
[ ] Blockers
[ ] Testing notes
[ ] Final completion note
[ ] Screenshots/video if UI-related
[ ] Related tickets if any
```

## Do Not Keep Only in Chat

These must not stay only in chat:

```text
Bugs
Blockers
Client decisions
Requirement changes
API changes
Design changes
Permission changes
Known issues
```

---

# 12. Final Completion Standard

Before marking a task done, add a final comment in the ticket.

## Completion Comment Template

```text
Status: Completed

Work Done:
- Built shared Add/Edit Project form.
- Added required field validation.
- Added loading, success, and error states.
- Added delete confirmation modal.
- Matched design system spacing and button styles.

Testing Done:
- Tested create success flow.
- Tested edit success flow.
- Tested required field validation.
- Tested API failure handling.
- Checked mobile layout.

Notes:
- No known blockers.
```

## If There Is a Known Issue

```text
Known Issue:
Bulk delete behavior is not included in this task and should be handled in a separate ticket.
```

---

# 13. Pull Request / Code Review Standard

Every PR should include:

```text
[ ] What changed
[ ] Why changed
[ ] Screenshots/video for UI changes
[ ] Test cases completed
[ ] Affected pages/modules
[ ] Any migrations/API changes
[ ] Known risks
```

## PR Description Example

```text
Summary:
Added shared ProjectForm component for Add and Edit project pages.

Changes:
- Created reusable ProjectForm.
- Added validation schema.
- Added loading/error/success states.
- Updated Add Project and Edit Project pages to use same component.

Testing:
- Create project success.
- Edit project success.
- Missing name validation.
- API error message.
- Mobile layout.

Impact:
Affects Add Project and Edit Project pages.
```

---

# 14. Developer Mindset Standard

Every team member should ask themselves before completing work:

```text
Would I accept this quality if I were the client?
Would a normal user understand this error?
Can this silently fail?
Is this consistent with similar pages?
Can this break another module?
Is this reusable?
Is this secure?
Did I test it myself?
Did I update the ticket properly?
```

---

# 15. Non-Negotiable Standards

These are mandatory. No exceptions.

```text
1.  No silent failures.
2.  No unclear "done" comments.
3.  No untested task completion.
4.  No inconsistent Add/Edit form behavior.
5.  No API error only in console.
6.  No missing required validation.
7.  No missing loading state on submit buttons.
8.  No destructive action without confirmation.
9.  No important issue only in chat.
10. No changing shared logic without checking impact.
11. No duplicate UI/code when a reusable component is possible.
12. No disabled field without clear visual indication.
13. No unclear work logs.
14. No skipping mobile check.
15. No marking complete without self-testing.
```

---

# 16. Team Standard Summary

Every task should follow this flow:

```text
Understand → Clarify → Plan → Build → Test → Update → Raise Issues → Complete with Evidence
```

The goal is not just to finish tickets. The goal is to build in a way that is:

```text
Accountable
Traceable
Consistent
Reusable
Secure
User-friendly
Maintainable
Tested
Professional
```

---

# One-Line Rule

```text
Own the work, think beyond the ticket, communicate clearly, test before handoff, and make every change traceable.
```
