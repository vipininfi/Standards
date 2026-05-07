# Tables & Data Lists Checklist

> **Core Rule:** Every table or list must handle every state — loading, empty, error, populated — and every action — sort, filter, paginate, select — before it is considered done.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-columns--data-display) | Columns & Data Display |
| [2](#2-loading-state) | Loading State |
| [3](#3-empty-state) | Empty State |
| [4](#4-error-state) | Error State |
| [5](#5-sorting) | Sorting |
| [6](#6-filtering) | Filtering |
| [7](#7-pagination) | Pagination |
| [8](#8-row-actions) | Row Actions |
| [9](#9-bulk-actions) | Bulk Actions |
| [10](#10-mobile) | Mobile |
| [11](#11-table-checklist--before-marking-done) | Table Checklist — Before Marking Done |

---

# 1. Columns & Data Display

```text
[ ] Column headers clearly label the data below them
[ ] Column widths set appropriately — no column that always overflows or wastes space
[ ] Long text truncated with an ellipsis and a tooltip showing full value on hover
[ ] Dates formatted consistently (e.g., "15 Jan 2024") — never raw ISO timestamps shown to user
[ ] Status values shown as badges with color coding — not plain text
[ ] Numbers aligned right, text aligned left
[ ] Currency formatted with correct symbol and decimal places
[ ] Empty/null values shown as "—" (em dash) not blank, "null", or "undefined"
[ ] Avatars/images have fallback initials or placeholder if image fails to load
[ ] Boolean values shown as "Yes/No" or a toggle icon — not "true/false"
[ ] Table has a visible border or row separation so rows are clearly distinct
```

---

# 2. Loading State

```text
[ ] Skeleton rows shown while data is loading (not a blank table or spinner alone)
[ ] Number of skeleton rows matches the expected page size (e.g., show 10 skeleton rows if page size is 10)
[ ] Column headers remain visible during loading (skeleton only replaces row content)
[ ] Action buttons (Add, Export) are disabled while table is loading
[ ] Sorting and filter controls are disabled while loading
[ ] On refresh/re-fetch: existing data stays visible with a subtle loading indicator (not full skeleton again)
```

---

# 3. Empty State

```text
[ ] Empty state shown when there are genuinely no records (not an empty table with no message)
[ ] Empty state message is specific: "No projects yet." not just "No data."
[ ] Empty state has a clear call to action: "Add your first project" button
[ ] Empty state distinguishes between "no records exist" and "no records match your filter"
  — No records: "No projects yet. Create one to get started."
  — No filter match: "No projects match your search. Clear filters to see all."
[ ] Clear filters button visible when empty state is due to active filters
[ ] Empty state has an appropriate illustration or icon (not just text)
```

---

# 4. Error State

```text
[ ] Error state shown if data fails to load — not a blank table
[ ] Error message is human-readable: "Could not load projects. Please try again."
[ ] "Try again" / "Retry" button visible on error state
[ ] Error is shown in the table area — not only in the console or a toast
[ ] If a re-fetch fails after data was already loaded: show the previously loaded data with an error banner above
[ ] Specific error codes mapped to specific messages (e.g., 403 → "You don't have permission to view this.")
```

---

# 5. Sorting

```text
[ ] Sortable columns are visually indicated (sort icon in header)
[ ] Active sort column shows direction (ascending ↑ / descending ↓)
[ ] Clicking a column header once: sort ascending
[ ] Clicking the same column header again: sort descending
[ ] Clicking a third time or a clear button: reset to default sort
[ ] Default sort defined and documented (e.g., newest first)
[ ] Sort is sent to the backend (not client-side sorting of the current page only — that misses other pages)
[ ] Sort state preserved when navigating away and coming back (URL param or local state)
[ ] Loading state shown while sort request is in progress
```

---

# 6. Filtering

```text
[ ] Active filters clearly visible (chips/tags showing what is currently filtered)
[ ] Each active filter has an individual X to remove it
[ ] "Clear all filters" button visible when any filter is active
[ ] Filters sent to backend — not applied only to current page
[ ] Filter values preserved in URL params so the filtered view is shareable/bookmarkable
[ ] Filter inputs debounced (text search: 300ms) — not firing on every keystroke
[ ] Loading state shown while filter request is processing
[ ] Dropdown filters show all options (not just currently visible data options)
[ ] Date range filters: end date must be after start date
[ ] Filter state preserved across page refreshes (via URL params)
```

---

# 7. Pagination

```text
[ ] Page controls visible: previous, next, page numbers
[ ] Current page highlighted clearly
[ ] Total record count shown: "Showing 1–20 of 143 projects"
[ ] Items per page selector (10, 25, 50, 100) with user preference saved
[ ] Previous button disabled on page 1
[ ] Next button disabled on last page
[ ] Pagination resets to page 1 when sort or filter changes
[ ] Page number preserved in URL param so the page is bookmarkable
[ ] Loading state shown between page changes
[ ] If only 1 page of results: pagination controls hidden or disabled gracefully
[ ] Pagination fetches from backend — not slicing a client-side array (which only works for the loaded data)
```

---

# 8. Row Actions

```text
[ ] Row action buttons (Edit, Delete, View) have consistent placement (same column, same position)
[ ] Destructive actions (Delete) are visually differentiated (red color, trash icon)
[ ] Delete always shows a confirmation dialog before proceeding — see [Delete & Destructive Actions](delete-destructive-actions.md)
[ ] Edit navigates to edit page or opens edit modal — consistent with the Add pattern for the same entity
[ ] Hover state on rows is visible (background highlight) to indicate interactivity
[ ] Row actions disabled for rows the user doesn't have permission to edit/delete
[ ] Disabled row actions show a tooltip explaining why they are disabled
[ ] Actions available per row match the user's role and permissions
[ ] Loading state on the specific row being actioned (not the whole table)
```

---

# 9. Bulk Actions

```text
[ ] Select all checkbox in header — selects all rows on current page
[ ] Individual row checkboxes
[ ] Selected count shown: "3 selected" or "3 of 20 selected"
[ ] Bulk action bar appears when any rows are selected
[ ] Bulk action bar disappears when selection is cleared
[ ] Bulk actions are clearly labeled: "Delete selected", "Export selected"
[ ] Bulk delete requires confirmation — "Delete 3 projects? This cannot be undone."
[ ] Selecting all offers "Select all 143 records" option when all current-page rows are checked
[ ] Loading state shown during bulk operation
[ ] Success feedback after bulk operation: "3 projects deleted."
[ ] Table refreshes after bulk action completes
[ ] Select all state cleared after a bulk action
```

---

# 10. Mobile

```text
[ ] Table does not cause horizontal scroll on mobile (< 768px)
[ ] On mobile: non-essential columns hidden (keep Name, Status, one action)
[ ] Consider card-based layout instead of table on mobile (rows → stacked cards)
[ ] Row actions reachable on mobile (not hidden by table overflow)
[ ] Pagination controls usable on touch (large enough tap targets)
[ ] Filter and sort controls accessible on mobile (drawer or collapsible panel)
```

---

# 11. Table Checklist — Before Marking Done

```text
DATA DISPLAY
[ ] All columns labeled and appropriately sized
[ ] Long text truncated with tooltip
[ ] Null values shown as "—"
[ ] Dates formatted for humans
[ ] Status values shown as color-coded badges

STATES
[ ] Loading: skeleton rows shown
[ ] Empty (no records): message + create CTA
[ ] Empty (no filter match): message + clear filters button
[ ] Error: message + retry button
[ ] All states tested manually

SORTING
[ ] Sort icons in column headers
[ ] Active sort direction shown
[ ] Sort sent to backend (not client-side)
[ ] Sort preserved in URL param

FILTERING
[ ] Active filters shown as chips
[ ] Clear individual filter and clear all
[ ] Filters sent to backend
[ ] Text search debounced 300ms
[ ] Filter state in URL params

PAGINATION
[ ] Total count shown
[ ] Previous/next + page numbers
[ ] Page size selector
[ ] Resets to page 1 on sort/filter change
[ ] Page in URL param

ROW ACTIONS
[ ] Delete shows confirmation dialog
[ ] Actions match user's permissions
[ ] Disabled actions have tooltip
[ ] Loading on the specific row

MOBILE
[ ] No horizontal scroll on mobile
[ ] Essential columns only on mobile
[ ] All actions reachable on touch
```
