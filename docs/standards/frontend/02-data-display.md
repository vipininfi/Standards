# Frontend Standards — Data Display

> Tables, lists, cards, search, filters, and pagination. Every data display component must handle every state — loading, empty, error, populated — and every user action must go to the backend.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-core-rule) | Core Rule |
| [2](#2-table--list-standards) | Table & List Standards |
| [3](#3-data-display-rules) | Data Display Rules |
| [4](#4-state-requirements) | State Requirements |
| [5](#5-sorting) | Sorting |
| [6](#6-search) | Search |
| [7](#7-filters) | Filters |
| [8](#8-pagination) | Pagination |
| [9](#9-infinite-scroll--load-more) | Infinite Scroll / Load More |
| [10](#10-url-state) | URL State |
| [11](#11-empty-states) | Empty States |
| [12](#12-row--item-actions) | Row & Item Actions |
| [13](#13-bulk-actions) | Bulk Actions |
| [14](#14-cards--list-items) | Cards & List Items |
| [15](#15-mobile) | Mobile |

---

# 1. Core Rule

```text
Search and filters always go to the backend.
Never filter or sort a pre-loaded client-side array.
A client-side filter only works on the current page — it misses all other pages.
```

---

# 2. Table & List Standards

```text
[ ] Every table has a defined default sort (newest first is default if no other business reason)
[ ] Column headers describe the data below — no vague "Info" or "Details" headers
[ ] Column widths set appropriately for their content (fixed width for status/date, flexible for name)
[ ] Tables do not cause horizontal scroll on desktop (1280px+)
[ ] Skeleton rows shown during initial load — not a blank table or a centered spinner
[ ] Skeleton row count matches the expected page size
[ ] Column headers remain visible during skeleton loading
[ ] All interactive elements in the table (sort icons, row actions) disabled during loading
```

---

# 3. Data Display Rules

```text
FORMATTING
[ ] Null / empty values displayed as "—" (em dash) — not blank, "null", "N/A", or "undefined"
[ ] Dates formatted consistently: "15 Jan 2024" — never raw ISO strings shown to users
[ ] Times include timezone context if relevant: "2:00 PM EST" or converted to user's local time
[ ] Currency: correct symbol + 2 decimal places aligned right: "$1,234.50"
[ ] Large numbers: comma-separated: "12,345" not "12345"
[ ] Percentages: "87%" not "0.87"
[ ] Long text: truncated at column width with ellipsis + full value in tooltip on hover
[ ] Boolean values: "Yes / No" or a check/cross icon — never "true/false"
[ ] Status values: color-coded badges — not plain text

IMAGES & AVATARS
[ ] Avatar has fallback: initials in a colored circle if image fails or is missing
[ ] Fallback color is deterministic (based on user's name or ID — same color every time)
[ ] Images have meaningful alt text
[ ] Images never load without explicit width and height (prevents layout shift)
```

---

# 4. State Requirements

Every table or list must handle all four states:

| State | What to Show |
|-------|-------------|
| **Loading** | Skeleton rows matching page size — column headers visible |
| **Empty (no data)** | Illustration/icon + specific message + create action CTA |
| **Empty (filter mismatch)** | "No results match your filters." + clear filters button |
| **Error** | Error message + retry button — never a blank area |

```text
[ ] Loading state: skeleton rows with shimmer animation
[ ] Empty (no records exist): "No [entities] yet." + create button
[ ] Empty (filter/search match): "No [entities] match your search." + clear button
[ ] Error: "Could not load [entities]. Please try again." + retry button
[ ] Error: if data was previously loaded, keep showing it with an error banner above
[ ] All four states tested manually before marking done
```

---

# 5. Sorting

```text
[ ] Sortable columns shown with a sort icon in the header
[ ] Active sort column shows direction: ↑ ascending, ↓ descending
[ ] Click once → ascending; click same column again → descending; click again or clear → default
[ ] Sort request sent to backend — not client-side array sorting
[ ] Sort state preserved in URL: ?sort=name&order=asc
[ ] Sort state preserved when navigating away and back
[ ] Default sort documented per table (usually created_at DESC)
[ ] Loading state while sort request is in progress
[ ] Pagination resets to page 1 when sort changes
```

---

# 6. Search

```text
BEHAVIOR
[ ] Debounced: 300ms delay after user stops typing before sending the request
[ ] Never fire on every keystroke
[ ] Loading indicator shown while search is processing
[ ] Previous results stay visible while new search loads (not replaced with blank)
[ ] Minimum character length enforced if needed (e.g., 2+ chars before searching)
[ ] Clear (X) button inside input when it has a value
[ ] Clearing input returns to unfiltered results — not empty results

TECHNICAL
[ ] Search request sent to backend — never filtering a pre-loaded array
[ ] User input is parameterized — not interpolated into queries or code
[ ] Search preserved in URL: ?search=query
[ ] Pagination resets to page 1 when search changes
[ ] Sort state preserved when search changes

AUTOCOMPLETE / SUGGESTIONS
[ ] Suggestions fetched from backend — debounced, same as search
[ ] Keyboard navigable: arrow keys to move, Enter to select, Escape to close
[ ] Selecting a suggestion populates the input and triggers the search
[ ] No suggestion list shown until user has typed the minimum character count
[ ] Suggestion list closes on outside click or Escape
```

---

# 7. Filters

```text
DISPLAY
[ ] Active filters shown as chips/tags above the results
[ ] Each chip shows: filter name + value: "Status: Active"
[ ] Each chip has an X to remove that specific filter
[ ] "Clear all filters" button visible when any filter is active
[ ] Number of active filters shown in filter button badge if filters are in a panel

BEHAVIOR
[ ] Filters sent to backend — not applied to current page data only
[ ] Multiple filters combine with AND logic (unless OR is explicitly designed)
[ ] Filter state preserved in URL params: ?status=active&assignee=me
[ ] Pagination resets to page 1 when any filter changes
[ ] Sort state preserved when filters change
[ ] Loading state while filter request is processing

FILTER INPUTS
[ ] Dropdown filters: show ALL possible values — not just values in current results
[ ] Date range: end date must be after start date (validate, show error if not)
[ ] Text filter: debounced same as search
[ ] Multi-select filter: selected values shown inside the input or as chips
[ ] Filter panel/drawer: "Apply" button OR immediate on-change — be consistent throughout the app
```

---

# 8. Pagination

```text
DISPLAY
[ ] Total record count shown: "Showing 1–20 of 143 results"
[ ] Page navigation: previous button, page numbers, next button
[ ] Current page clearly highlighted
[ ] Previous button disabled on page 1
[ ] Next button disabled on last page
[ ] Items per page selector: 10, 25, 50, 100 (or app-appropriate defaults)

BEHAVIOR
[ ] User's items-per-page preference saved (localStorage or user settings)
[ ] Pagination state preserved in URL: ?page=3&per_page=25
[ ] Page resets to 1 when search, filter, or sort changes
[ ] Pagination fetches from backend — never slicing a pre-loaded array
[ ] Loading state between page changes
[ ] If only 1 page of results: pagination controls hidden or shown as inactive
[ ] Jump-to-page input for large datasets (optional, useful when > 50 pages)
```

---

# 9. Infinite Scroll / Load More

Use for browsing contexts (feeds, galleries, activity logs). Not for data that needs precise navigation.

```text
[ ] "Load more" button preferred over automatic scroll trigger (gives user control)
[ ] "Load more" shows a spinner while next page loads
[ ] "Load more" disabled while loading (no double-click)
[ ] "End of results" message shown when all data is loaded
[ ] Previously loaded items remain in the list — not replaced on each load
[ ] Scroll position preserved if user navigates away and returns
[ ] Error loading next page: error shown near "Load more" button + retry
```

---

# 10. URL State

All search, filter, sort, and pagination state must live in URL params.

```text
[ ] Search: ?search=query
[ ] Filters: ?status=active&assignee=me
[ ] Sort: ?sort=name&order=asc
[ ] Page: ?page=3
[ ] Per page: ?per_page=25
[ ] URL updated without full page reload (replaceState for filter changes, pushState for page changes)
[ ] Refreshing the page restores the exact view
[ ] Sharing the URL reproduces the same search/filter/sort/page for another user
[ ] Back button restores the previous filter/search state
```

---

# 11. Empty States

```text
NO DATA EXISTS
[ ] Illustration or icon — not just text
[ ] Specific message: "No projects yet." — not "No data." or "Empty."
[ ] Call to action: "Create your first project" button or link

SEARCH RETURNED NO RESULTS
[ ] "No results for '[search term]'."
[ ] Suggestion: "Try a different search term."
[ ] Clear search button visible

FILTER RETURNED NO RESULTS
[ ] "No [entities] match your current filters."
[ ] Shows what filters are active
[ ] "Clear all filters" button prominent

COMBINATION (search + filter)
[ ] "No results for '[term]' with these filters."
[ ] Both "Clear search" and "Clear filters" available

RULE: Never show a blank table body or an empty list without a message.
```

---

# 12. Row & Item Actions

```text
[ ] Actions (Edit, Delete, View, Archive) in a consistent column or menu — same position in every row
[ ] Destructive actions (Delete) visually differentiated: red icon, trash icon
[ ] Delete always shows a confirmation dialog — see [Delete & Destructive Actions](../../checklists/frontend/delete-destructive-actions.md)
[ ] Edit navigates to edit page or opens edit modal — must be consistent with the Add pattern
[ ] Row actions are role-appropriate: members cannot see delete if they cannot delete
[ ] Disabled row actions have a tooltip: "Admin access required to delete this."
[ ] Loading state on the specific row being actioned — not the whole table
[ ] After delete: row removed from list without full page refresh
[ ] Hover state on rows shows background highlight to signal interactivity
```

---

# 13. Bulk Actions

```text
[ ] Header checkbox selects/deselects all rows on the current page
[ ] Individual row checkboxes
[ ] Selection count shown: "3 selected" or "3 of 20 selected"
[ ] Bulk action bar appears only when rows are selected, disappears when cleared
[ ] Bulk action bar clearly labeled: "Delete selected (3)", not just "Delete"
[ ] "Select all N records" option when all current-page rows are checked
[ ] Bulk delete requires confirmation: "Delete 3 projects? This cannot be undone."
[ ] Loading state during bulk operation
[ ] Success: "3 projects deleted." toast shown
[ ] Partial success: "2 of 3 projects deleted. 1 could not be deleted." with reason
[ ] Selection cleared after operation, table refreshes
```

---

# 14. Cards & List Items

```text
[ ] Card padding consistent with design system spacing (not random per-card padding)
[ ] Card hover state visible (shadow lift or background change)
[ ] Cards link to the entity's detail page — entire card or a dedicated "View" link
[ ] Action buttons inside cards (Edit, Delete) have consistent placement
[ ] Card images have aspect ratio maintained — never distorted
[ ] Loading: skeleton cards in the same grid layout as real cards
[ ] Empty state: empty card grid with a centered "Add first..." card or message
```

---

# 15. Mobile

```text
[ ] Tables: no horizontal scroll on mobile — essential columns only, or card layout instead
[ ] On mobile (< 768px): hide non-essential table columns (keep: Name, Status, one action)
[ ] Card grids: 1 column on mobile, 2 on tablet, 3 on desktop
[ ] Row actions reachable on mobile: not hidden by table overflow
[ ] Pagination controls large enough to tap (min 44×44px touch targets)
[ ] Filter and sort accessible on mobile: drawer or collapsible panel
[ ] Search input full-width on mobile
[ ] "Load more" button full-width on mobile
```
