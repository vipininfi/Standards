# Loading States & Skeletons Checklist

> **Core Rule:** Every async action must show a loading state. No button fires twice. No page renders blank. No action happens silently. Loading is not optional.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-button-loading-states) | Button Loading States |
| [2](#2-page--section-loading) | Page & Section Loading |
| [3](#3-skeleton-screens) | Skeleton Screens |
| [4](#4-table-loading) | Table Loading |
| [5](#5-inline--component-loading) | Inline & Component Loading |
| [6](#6-what-never-needs-a-loading-state) | What Never Needs a Loading State |
| [7](#7-loading-state-checklist--before-marking-done) | Loading State Checklist — Before Marking Done |

---

# 1. Button Loading States

Applied to any button that triggers an async action (form submit, API call, delete, send).

```text
[ ] Button shows a spinner while the async action is in progress
[ ] Button is disabled while loading (cannot click twice)
[ ] Button text changes to reflect the action: "Saving..." / "Deleting..." / "Sending..."
  — OR spinner replaces the icon on an icon-only button
[ ] Button returns to normal state after success or failure
[ ] Button width does not change when switching to loading state (use fixed width or min-width)
[ ] If the action succeeds and the view stays the same: button resets to its original state
[ ] If the action succeeds and the view navigates away: button stays in loading state until navigation completes
```

### Loading Button Labels

| Action | Default Label | Loading Label |
|--------|--------------|---------------|
| Create | "Create Project" | "Creating..." |
| Save | "Save Changes" | "Saving..." |
| Delete | "Delete" | "Deleting..." |
| Send | "Send Invite" | "Sending..." |
| Submit | "Submit" | "Submitting..." |
| Upload | "Upload" | "Uploading..." |
| Export | "Export" | "Exporting..." |

---

# 2. Page & Section Loading

```text
[ ] Full-page data fetch: skeleton shown immediately — never a blank white page
[ ] Page shows the header/nav/layout frame while content loads (only the data area uses a skeleton)
[ ] Page does not flash blank before the skeleton appears (skeleton is the initial render state)
[ ] "Page loading" spinner in the center of the page: only acceptable for very short operations (< 500ms)
[ ] For operations > 500ms: use a skeleton, not a centered spinner
[ ] Error state shown if the page data fails to load — not a blank page
[ ] Retry button visible on page-level error state
```

---

# 3. Skeleton Screens

Skeleton screens are preferred over spinners for content that has a predictable shape.

```text
[ ] Skeleton matches the approximate layout of the real content:
  — Card grid → skeleton cards in a grid
  — List → skeleton list rows
  — Form → skeleton inputs and labels
  — Detail page → skeleton for each section
[ ] Skeleton uses animated shimmer/pulse effect (not a static grey block)
[ ] Skeleton height and width approximate the real content (not a single full-width bar for everything)
[ ] Number of skeleton items matches expected data (e.g., 5 skeleton rows for a 5-item list)
[ ] Skeleton replaced instantly when data arrives (no flash of skeleton after data is ready)
[ ] Skeleton not used for actions that complete in < 300ms (fast enough that a skeleton would flash and disappear)
[ ] No text content in skeleton — only grey placeholder shapes
```

### When Skeleton vs Spinner

| Use Skeleton | Use Spinner |
|-------------|-------------|
| Initial page/section load | Button action in progress |
| Table/list loading | Small inline component refresh |
| Card grid loading | Overlay loading (full page blocking) |
| Form pre-fill in Edit mode | Short operations < 300ms |
| Dashboard widget loading | Single icon or avatar loading |

---

# 4. Table Loading

```text
[ ] Skeleton rows shown (not a spinner in the table area)
[ ] Number of skeleton rows = page size (shows expected density)
[ ] Column headers visible during skeleton loading
[ ] Skeleton rows have approximate column widths matching real data
[ ] Sort/filter controls disabled during loading
[ ] On re-fetch (filter change, sort change): show existing rows with a subtle loading overlay or spinner above the table — not full skeleton again
```

---

# 5. Inline & Component Loading

```text
[ ] Dropdown options: spinner inside the dropdown while options are fetching
[ ] Autocomplete/search: spinner or "Searching..." text while debounced fetch is in progress
[ ] Image lazy load: placeholder background color shown until image loads
[ ] Avatar: initials placeholder shown until avatar image loads
[ ] Dashboard widgets: individual widget skeletons (not the whole page)
[ ] Infinite scroll / load more: spinner at the bottom of the list while next page loads
[ ] File upload: progress bar or percentage shown during upload
[ ] Tab content: skeleton in the tab panel when tab is switched and content loads
```

---

# 6. What Never Needs a Loading State

```text
× Client-side navigation (instantaneous — no loading needed)
× Toggling a UI element (show/hide, expand/collapse) — synchronous
× Client-side form validation (no async operation)
× Hover/focus state changes
× Animations and transitions
× Reading from in-memory state (no API call)
```

---

# 7. Loading State Checklist — Before Marking Done

```text
BUTTONS
[ ] Every submit / delete / send / action button shows spinner on click
[ ] Button disabled during loading (no double submit)
[ ] Button label changes to reflect in-progress action
[ ] Button returns to normal after success or failure

PAGE LOAD
[ ] Initial data fetch shows skeleton (not blank page)
[ ] Layout frame visible while content skeleton loads
[ ] Error state shown if fetch fails (not blank)
[ ] Retry button on error state

SKELETON
[ ] Skeleton layout matches the shape of real content
[ ] Animated shimmer effect on skeleton
[ ] Correct number of skeleton items
[ ] Skeleton disappears instantly when data arrives

TABLE
[ ] Table shows skeleton rows during load
[ ] Column headers visible during skeleton
[ ] Re-fetch: overlay/spinner on existing data (not full skeleton)
[ ] Sort/filter disabled while loading

INLINE COMPONENTS
[ ] Dropdowns: spinner while options load
[ ] Search/autocomplete: loading indicator while fetching
[ ] Images: placeholder until loaded
[ ] Widgets: individual skeletons (not page-level)

GLOBAL RULE
[ ] No async action completes without a visible loading indicator
[ ] No button can be clicked twice during an async action
[ ] No page renders blank during data fetch
```
