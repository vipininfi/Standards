# Templates

> Copy these into tickets, PRs, and documentation. Fill in the placeholders — do not start from scratch.

---

## Available Templates

| Template | Use When | What's Inside |
|----------|----------|--------------|
| [Work Log Template](work-log-template.md) | Logging time and posting status updates in a ticket | Three formats: In Progress update, Completed update, and Blocked update — each with what was done, time spent, current status, and next step |
| [Bug Report Template](bug-report-template.md) | Reporting any bug or blocker found during work | Steps to reproduce, expected vs actual behavior, severity level, environment details, and affected area |
| [Completion Comment Template](completion-comment-template.md) | Marking a task done with proper evidence | What was done, what was tested, how to verify, links/screenshots, time spent, and any known follow-ups |
| [API Documentation Template](api-doc-template.md) | Documenting any backend API for the frontend team | Endpoint name, method, URL, auth requirement, full request payload with types, success response, all error cases with codes, and a frontend code example |
| [Form Documentation Template](form-doc-template.md) | Documenting a form's fields, validation rules, and submission behavior | Field list with type/required/validation, submission endpoint, success and error handling, and any conditional logic |

---

## How to Use

1. Open the template that matches what you need to document.
2. Copy the entire content into your ticket comment, PR description, or documentation file.
3. Replace every `[placeholder]` with the actual value.
4. Remove any sections that do not apply.
5. Do not leave unfilled placeholders — reviewers will assume missing sections were skipped.

---

## Related

- [API Documentation Standard](../standards/supabase/03-api-documentation.md) — Full rules for documenting Supabase APIs
- [Team Ownership Standards](../standards/team-ownership.md) — Rules for work updates, ticket discipline, and completion comments
- [QA & Delivery Standards](../standards/qa-delivery-standards.md) — What counts as done and what evidence is required
