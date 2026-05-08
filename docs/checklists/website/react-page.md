# Checklist — React Page Build

> Run this checklist every time you build a new React page or significant component.

**Standard:** [Frontend & UI Standards](../../standards/frontend-ui-standards.md)

---

## 1. Before You Start

```text
[ ] Figma design reviewed — all states identified (loading, empty, error, populated)
[ ] API endpoint confirmed and response shape known before building
[ ] Reusability checked — does a component for this already exist?
[ ] Folder structure correct: pages/ for pages, components/ for shared components
[ ] TypeScript types defined for all API response shapes before building
```

---

## 2. Component Structure

```text
[ ] One component per file
[ ] Props interface defined with TypeScript (not any)
[ ] No prop drilling more than 2 levels — use context or state management instead
[ ] Component does one thing — extract sub-components if it grows too large
[ ] All side effects in useEffect with correct dependency arrays
[ ] No direct DOM manipulation — use React state and refs
[ ] No business logic in JSX — extract to hooks or utility functions
```

---

## 3. Data Fetching

```text
[ ] Custom hook for each data fetch (useProjects, useWorkspaceMembers)
[ ] Hook returns: { data, isLoading, error, refetch }
[ ] Loading state shown while isLoading is true (skeleton or spinner)
[ ] Error state shown when error is not null — with retry button
[ ] Empty state shown when data is an empty array
[ ] All four states tested: loading, empty, error, populated
[ ] Data fetching triggered in useEffect or on user action — not on every render
[ ] Cleanup: abort controller or return cleanup function in useEffect
```

---

## 4. Forms

```text
[ ] Form state managed with react-hook-form or equivalent
[ ] Validation schema defined with Zod or Yup — shared between Add and Edit
[ ] Same component used for Add and Edit — only props differ
[ ] Errors displayed below each field (from form state — not manual state)
[ ] Submit button disabled during submission
[ ] Form not re-submittable while loading
[ ] Success: show toast, navigate or close modal
[ ] Error: show API error in a form banner — do not clear the form
[ ] Form fields reset only after successful submission
```

---

## 5. State Management

```text
[ ] Local state (useState) for UI state: modal open/closed, selected items
[ ] Server state (React Query, SWR, or custom hook) for fetched data
[ ] Global state (Context, Zustand, Redux) only when truly global: auth, theme
[ ] No business logic in the store — only data and simple computed values
[ ] Optimistic updates considered for common actions (toggle, archive)
```

---

## 6. Error Handling

```text
[ ] Every async call has an error handler
[ ] Errors shown in the UI — never only in console.log
[ ] API error codes parsed and mapped to user-friendly messages
[ ] Network errors caught and shown: "Connection lost. Please try again."
[ ] Error boundary used at page level to catch unexpected rendering errors
[ ] No swallowed catch blocks: catch (e) {} (always show something)
```

---

## 7. TypeScript

```text
[ ] No any types — use explicit types or unknown
[ ] API response types defined in a types/ or types.ts file
[ ] Props interfaces defined for all components
[ ] Event handler types correct: React.ChangeEvent<HTMLInputElement>
[ ] Avoid type assertions (as Type) unless unavoidable — prefer type guards
[ ] All hook return types explicitly typed
```

---

## 8. Accessibility

```text
[ ] All interactive elements are keyboard accessible
[ ] Buttons use <button>, links use <a href>
[ ] Images have alt text
[ ] Form labels linked to inputs with htmlFor / id
[ ] Focus management in modals and dialogs (focus trap)
[ ] ARIA attributes used correctly — not overused
[ ] Color is not the only differentiator (status badges also have text labels)
```

---

## 9. Performance

```text
[ ] Components wrapped in React.memo only if they actually cause performance issues
[ ] useCallback and useMemo only for expensive computations or stable references
[ ] Images: width and height defined to prevent layout shift
[ ] Lazy loading used for pages and heavy components (React.lazy + Suspense)
[ ] No unnecessary re-renders: check dependencies in useEffect and useCallback
[ ] Large lists: virtualization considered if list > 100 items
```

---

## 10. Mobile

```text
[ ] Tested at 375px, 768px, and 1280px
[ ] No horizontal scroll on any breakpoint
[ ] Touch targets minimum 44×44px
[ ] Tap targets not too close together (minimum 8px gap)
[ ] Input focus does not zoom the page on mobile (use font-size: 16px on inputs)
```

---

## Done When

```text
[ ] All four data states work: loading, empty, error, populated
[ ] All forms validate, show loading, show success/error, share Add/Edit component
[ ] No any TypeScript types
[ ] All errors shown in UI — no silent failures
[ ] Mobile tested at 375px and 768px
[ ] Accessibility: keyboard navigation and ARIA labels checked
[ ] No hardcoded styles — design tokens / Tailwind classes used
```
