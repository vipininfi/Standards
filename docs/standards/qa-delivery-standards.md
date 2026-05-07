# QA & Delivery Standards

Core rule:

> A task is not done because it is built — it is done because it is tested, working, and documented.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-self-testing--before-handing-off) | Self-Testing Before Handoff |
| [2](#2-backend-api--definition-of-done) | Backend API — Definition of Done |
| [3](#3-backend-api--test-cases-required) | Backend Test Cases Required |
| [4](#4-rls-testing-matrix) | RLS Testing Matrix |
| [5](#5-frontend-task--definition-of-done) | Frontend Task — Definition of Done |
| [6](#6-website--definition-of-done) | Website — Definition of Done |
| [7](#7-website-visual-qa) | Website Visual QA |
| [8](#8-task-completion-comment-standard) | Task Completion Comment |
| [9](#9-qa-handoff-checklist-from-developer-to-qa) | QA Handoff Checklist |
| [10](#10-definition-of-done--summary) | Definition of Done Summary |

---

# 1. Self-Testing — Before Handing Off

Every developer must test their own work. QA should not catch issues that the developer could have found with a basic check.

## Universal Self-Test Checklist

```text
[ ] Main success flow works end-to-end.
[ ] Required field validation works.
[ ] Invalid input shows correct error.
[ ] API failure is shown in UI (not just console).
[ ] Loading state shows during async actions.
[ ] Empty state is handled (no broken/blank screens).
[ ] Mobile layout is checked.
[ ] Similar Add/Edit pages are both checked.
[ ] Permission-based behavior is correct.
[ ] No broken layout on any breakpoint.
[ ] No silent failure anywhere.
[ ] Browser console has no obvious errors.
```

---

# 2. Backend API — Definition of Done

An API is complete only when all of these are checked:

```text
[ ] Migration created
[ ] Table schema reviewed
[ ] Constraints added
[ ] Indexes added
[ ] RLS enabled
[ ] SELECT policy added
[ ] INSERT policy added
[ ] UPDATE policy added
[ ] DELETE policy added (or intentionally blocked)
[ ] RPC/Edge Function created if needed
[ ] Auth requirement implemented
[ ] Role checks implemented
[ ] Input validation implemented
[ ] Error responses standardized
[ ] Logs/audit logs added
[ ] Storage policy added if files are involved
[ ] Webhook signature/idempotency added if webhook exists
[ ] API documented for frontend
[ ] Frontend sample code provided
[ ] Test cases documented
[ ] Tested as: owner / admin / member / viewer / non-member / anonymous
[ ] Staging verified
```

---

# 3. Backend API — Test Cases Required

Every API must be tested for all these cases:

| Test Case | Required |
|-----------|----------|
| Authenticated success | Yes |
| Anonymous request | Yes |
| Expired token | Yes |
| Wrong role | Yes |
| Wrong workspace | Yes |
| Missing required field | Yes |
| Invalid field type | Yes |
| Duplicate record | Yes |
| Related record missing | Yes |
| Database constraint failure | Yes |
| RLS denial | Yes |
| External service failure | If applicable |
| Retry/idempotency | If applicable |

---

# 4. RLS Testing Matrix

For every table, test all combinations:

| User Type | SELECT | INSERT | UPDATE | DELETE |
|-----------|--------|--------|--------|--------|
| Anonymous | No | No | No | No |
| Owner | Yes | Yes | Yes | Yes |
| Admin | Yes | Yes | Yes | Depends |
| Manager | Yes | Yes | Yes | No |
| Member | Yes | Depends | Depends | No |
| Viewer | Yes | No | No | No |
| Non-member | No | No | No | No |

---

# 5. Frontend Task — Definition of Done

A frontend task is complete only when:

```text
[ ] Main success flow works.
[ ] Required field validation works.
[ ] Invalid input shows field-level error.
[ ] API error is shown in UI (not just console).
[ ] Loading state is shown during submit/fetch/delete.
[ ] Empty state is handled gracefully.
[ ] Mobile layout tested.
[ ] Add/Edit pages use same component if for same entity.
[ ] Permission-based UI behavior correct (hide/disable/show).
[ ] No silent failures.
[ ] Ticket updated with testing evidence.
```

---

# 6. Website — Definition of Done

A website build or replica is complete only when:

```text
[ ] Client ownership/permission confirmed (if replication)
[ ] Source website/Figma reviewed
[ ] Page inventory completed
[ ] Design tokens documented
[ ] Components documented
[ ] All required pages built
[ ] Desktop responsive tested
[ ] Tablet responsive tested
[ ] Mobile responsive tested
[ ] Forms validated and tested
[ ] Emails/CRM/webhooks tested
[ ] SEO metadata added
[ ] Redirects added (if migration)
[ ] Analytics/tracking installed
[ ] Images optimized
[ ] Accessibility basics checked
[ ] Performance checked
[ ] Staging link approved by client
[ ] Production deployed
[ ] SSL active
[ ] Handoff document delivered
[ ] Client knows how to edit/manage content
```

---

# 7. Website Visual QA

## Visual Comparison

```text
[ ] Header matches source/Figma
[ ] Footer matches source/Figma
[ ] Fonts match (family, weight, size)
[ ] Colors match (exact hex)
[ ] Buttons match (radius, hover, padding)
[ ] Cards match (shadow, border, spacing)
[ ] Images match (size, crop, quality)
[ ] Spacing matches (section padding, gaps)
[ ] Desktop layout checked
[ ] Tablet layout checked
[ ] Mobile layout checked
```

## Functional QA

```text
[ ] All nav links work
[ ] All CTAs work
[ ] Contact form works and submits
[ ] Newsletter form works
[ ] Booking links work
[ ] External links open correctly (new tab if needed)
[ ] Social links work
[ ] Search works if present
[ ] CMS content loads if present
[ ] API errors are handled gracefully
```

## SEO QA

```text
[ ] Title tags added to all pages
[ ] Meta descriptions added
[ ] Open Graph images added
[ ] Canonical URLs set
[ ] Sitemap generated and submitted
[ ] Robots.txt configured
[ ] Redirects tested (if migration)
[ ] No accidental noindex on important pages
[ ] Broken links checked
```

## Performance QA

```text
[ ] Images compressed (WebP preferred)
[ ] Fonts optimized (only needed weights)
[ ] Unused scripts removed
[ ] Lazy loading added below fold
[ ] Mobile speed checked
[ ] Large layout shifts fixed
```

---

# 8. Task Completion Comment Standard

Before marking any task done, add a final comment to the ticket.

## Template

```text
Status: Completed

Work Done:
- [What was built or fixed]
- [What was updated]

Testing Done:
- [What success flows were tested]
- [What error/edge cases were tested]
- [Mobile checked: Yes/No]

Notes:
- [Any known issues or follow-up items, or "None"]
```

## Example

```text
Status: Completed

Work Done:
- Built shared Add/Edit Project form.
- Added required field validation.
- Added loading, success, and error states.
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

---

# 9. QA Handoff Checklist (From Developer to QA)

Before handing off to QA, confirm:

```text
[ ] Task is self-tested.
[ ] Ticket is updated with what was done.
[ ] Test cases are listed in the ticket.
[ ] Screenshots/video attached if UI-related.
[ ] Edge cases that could not be tested are noted.
[ ] Any known issues are documented.
[ ] Related pages/modules that may be affected are listed.
[ ] Mobile was checked by developer.
```

---

# 10. Definition of Done — Summary

| Role | Done When |
|------|-----------|
| Backend Developer | Schema + RLS + Validation + Error handling + Logs + Tests + Documentation all complete |
| Frontend Developer | Main flow + Validation + Loading + Error UI + Mobile + Self-tested + Ticket updated |
| Website Builder | All pages + Forms + SEO + Redirects + Analytics + Mobile + Client approved staging |
| Any Team Member | Requirement met + Self-tested + Ticket updated with evidence + No silent failures |
