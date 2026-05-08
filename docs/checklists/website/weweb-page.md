# Checklist — WeWeb Page Build

> Run this checklist every time you build a new page or significant section in WeWeb.

**Standard:** [Frontend & UI Standards](../../standards/frontend-ui-standards.md) · [Website Replication Standards](../../standards/website/README.md)

---

## 1. Before You Start

```text
[ ] Figma design reviewed — all states identified (empty, loading, error, populated)
[ ] Xano (or other backend) API endpoint confirmed and documented before building
[ ] API response structure known — not building against a guess
[ ] Reusable components checked — does this page need an existing component?
[ ] Global styles / design tokens confirmed (colors, font sizes, spacing variables)
```

---

## 2. Page Structure

```text
[ ] Page extends the global layout (header, sidebar, footer) — not rebuilt per page
[ ] Page sections use the correct layout components (rows, columns, containers)
[ ] Spacing uses WeWeb global variables — not hardcoded pixel values
[ ] Colors use WeWeb global color tokens — not hardcoded hex
[ ] Font sizes and weights use the global typography scale
[ ] No inline styles added unless a global variable doesn't exist
```

---

## 3. Data Binding

```text
[ ] All data comes from a WeWeb collection (bound to an API or Xano)
[ ] Collections prefetched at the page level — not triggered from inside a component
[ ] Data bound correctly: list → repeat element, single record → bound properties
[ ] Loading state: collection's isFetching state used to show skeleton or spinner
[ ] Error state: collection's error state used to show error message
[ ] Empty state: checked with v-if on the empty state element when data.length === 0
[ ] All four states tested: loading, empty, error, populated
```

---

## 4. Xano API / Backend Calls

```text
[ ] Auth token sent with every authenticated request (via WeWeb auth plugin — not manually)
[ ] Error from Xano parsed and shown to user — not swallowed
[ ] Success actions trigger a collection refresh or update the bound data
[ ] No duplicate API calls (not calling the same endpoint from multiple places on one page)
[ ] Workflow steps:
    [ ] 1. Disable button / show loading
    [ ] 2. Call Xano action
    [ ] 3. On success: show toast + refresh data
    [ ] 4. On error: show error message, re-enable button
```

---

## 5. Forms in WeWeb

```text
[ ] Each form field bound to a variable (not hardcoded)
[ ] Form variables reset after successful submit
[ ] Validation done before submitting to Xano:
    [ ] Required fields checked
    [ ] Format checked (email, phone)
    [ ] WeWeb's built-in validation or custom workflow condition
[ ] Error messages shown per field — not just a generic toast
[ ] Submit button disabled and shows loading text during submission
[ ] Form not re-submittable while in loading state
```

---

## 6. Reusable Components

```text
[ ] Repeated UI sections converted to WeWeb components — not duplicated
[ ] Component props defined for all variable content (title, status, actions)
[ ] Components do not make their own API calls — data passed in as props
[ ] Components emit events upward — do not write directly to a global collection
[ ] Naming: component names are descriptive: ProjectCard, StatusBadge, MemberRow
```

---

## 7. Navigation & Routing

```text
[ ] Page-level permissions checked via a workflow on page load
[ ] If user lacks access: redirect to login or show 403 message — not a blank page
[ ] Navigation links use WeWeb's router — not hardcoded anchor tags
[ ] Dynamic routes use path parameters correctly: /projects/:id
[ ] Back button works as expected (browser history)
[ ] Redirects after actions use the router — not window.location.href
```

---

## 8. Mobile & Responsive

```text
[ ] Page tested at 375px (mobile), 768px (tablet), 1280px (desktop)
[ ] No horizontal scroll on mobile
[ ] Font sizes readable on mobile — not scaled down from desktop
[ ] Touch targets minimum 44×44px
[ ] Flex columns switch to column layout on mobile (not side-by-side)
[ ] Tables: column-stacking or card layout on mobile
[ ] Dropdowns and modals usable on mobile
```

---

## 9. Performance

```text
[ ] Images have defined width and height (no layout shift)
[ ] Images from the backend have CDN URLs — not raw storage paths
[ ] Collections not fetching more data than needed (pagination used)
[ ] No collection fetch inside a repeat loop (N+1 problem)
[ ] Heavy components: consider lazy loading if not above the fold
```

---

## Done When

```text
[ ] All four data states work: loading, empty, error, populated
[ ] All forms validate, submit, show loading, show success/error
[ ] Reusable components extracted for repeated UI
[ ] Mobile tested at 375px and 768px
[ ] No hardcoded colors, sizes, or spacing (global tokens used)
[ ] Permissions checked on page load
[ ] All Xano errors shown to user — no silent failures
```
