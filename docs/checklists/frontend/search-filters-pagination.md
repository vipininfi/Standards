# Search, Filters & Pagination Checklist

> **Core Rule:** Search and filters must go to the backend — never filter a client-side array. State must live in the URL so results are shareable and preserved on back navigation.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-search) | Search |
| [2](#2-filters) | Filters |
| [3](#3-combining-search--filters) | Combining Search & Filters |
| [4](#4-pagination) | Pagination |
| [5](#5-infinite-scroll--load-more) | Infinite Scroll / Load More |
| [6](#6-url-state) | URL State |
| [7](#7-empty--no-results-states) | Empty & No-Results States |
| [8](#8-search-filters-pagination-checklist--before-marking-done) | Checklist — Before Marking Done |

---

# 1. Search

```text
[ ] Search input is debounced — wait 300ms after the user stops typing before sending the request
[ ] Do NOT fire a search request on every keystroke
[ ] Loading indicator shown while search request is in progress (spinner inside or below the search input)
[ ] Previous results stay visible while new search is loading (not replaced with blank)
[ ] Search is sent to the backend — never filtering a pre-loaded client-side array
[ ] Search input is parameterized — user input is passed as a query parameter, never interpolated into SQL or code
[ ] Minimum character count enforced if needed (e.g., require 2+ characters before searching)
[ ] Clear button (X) appears inside the search input when it has a value
[ ] Clearing the input resets results to the unfiltered state (not empty results)
[ ] Search query preserved in URL param: ?search=query
[ ] Search preserved when navigating away and back
[ ] Search input has placeholder: "Search projects..." (not just "Search")
[ ] Autocomplete suggestions: fetched from backend, debounced, keyboard-navigable (arrow keys + Enter)
[ ] Autocomplete: selecting a suggestion populates the input and triggers the search
```

---

# 2. Filters

```text
[ ] Filters sent to backend — not applied to the current page's data only
[ ] Multiple filters combinable (e.g., Status: Active AND Assigned to: Me)
[ ] Filters are additive by default (AND logic unless OR is explicitly designed)
[ ] Active filters shown as removable chips/tags above or near the results
[ ] Each active filter chip has an X to remove that specific filter
[ ] "Clear all filters" button visible whenever any filter is active
[ ] Filter state preserved in URL params: ?status=active&assignee=me
[ ] Date range filters: end date must be after start date — show validation if not
[ ] Dropdown filters: all possible values shown, not just values present in current results
[ ] Filter counts shown if useful: "Active (12)", "Archived (3)"
[ ] Loading state shown while filter request is processing
[ ] Filters reset to page 1 when changed
[ ] Filter panel/drawer has a "Apply" button OR filters apply immediately on change (pick one, be consistent)
```

---

# 3. Combining Search & Filters

```text
[ ] Search and filters work together — they are both sent in the same request
[ ] Search + filter: results must match BOTH the search term AND all active filters
[ ] Changing a filter does not reset the search input
[ ] Changing the search does not reset the filters
[ ] URL contains both: ?search=marketing&status=active
[ ] Clear all resets both search and all filters simultaneously
[ ] Sort state is preserved when search or filters change
[ ] Page resets to 1 when search or filters change
```

---

# 4. Pagination

```text
[ ] Total record count displayed: "Showing 1–20 of 143 results"
[ ] Per-page options available: 10, 25, 50, 100 (or app-appropriate defaults)
[ ] User's per-page preference saved (localStorage or user settings)
[ ] Previous button disabled on page 1
[ ] Next button disabled on last page
[ ] Page numbers shown for quick navigation (show first, last, and surrounding pages)
[ ] Current page highlighted
[ ] Pagination resets to page 1 when search, filter, or sort changes
[ ] Page number preserved in URL: ?page=3
[ ] Loading state between page changes (spinner or skeleton)
[ ] If total count is unavailable: show "Next" only (no total, no page numbers)
[ ] Jump-to-page input for large datasets (optional but helpful when > 50 pages)
```

---

# 5. Infinite Scroll / Load More

Use this instead of pagination when the use case is browsing rather than precise navigation (feeds, galleries).

```text
[ ] "Load more" button preferred over automatic infinite scroll (gives user control)
  — Automatic scroll only for feed/social patterns where position doesn't need to be tracked
[ ] "Load more" button shows a spinner while loading next page
[ ] "Load more" button disabled while loading (no double-click)
[ ] "No more results" / "You've reached the end" shown when all data is loaded
[ ] Previously loaded items remain in the list — not replaced
[ ] Scroll position preserved if user navigates away and comes back (using history state or URL page param)
[ ] Error loading next page: error shown near the "Load more" button with a retry option
[ ] First load: skeleton items shown (same as regular table loading)
```

---

# 6. URL State

All search, filter, sort, and page state must be in the URL so results are shareable and browser-back works.

```text
[ ] Search: ?search=query
[ ] Filters: ?status=active&assignee=me&created_after=2024-01-01
[ ] Sort: ?sort=name&order=asc
[ ] Page: ?page=3
[ ] Per page: ?per_page=25
[ ] URL state read on page load (so refreshing restores the exact view)
[ ] Navigating to a shared URL reproduces the same search/filter/sort/page
[ ] URL updated without full page reload (replaceState for filter changes, pushState for page navigation)
[ ] Back button restores the previous filter/search state correctly
```

---

# 7. Empty & No-Results States

```text
NO RECORDS EXIST (table is genuinely empty)
[ ] "No [entities] yet. [CTA to create first one]."
[ ] Illustration or icon shown
[ ] Create button visible

SEARCH RETURNED NO RESULTS
[ ] "No results for '[search term]'."
[ ] "Try a different search term." suggestion
[ ] Clear search button visible

FILTERS RETURNED NO RESULTS
[ ] "No [entities] match your filters."
[ ] "Clear filters" button visible
[ ] Show what filters are active so the user knows why results are empty

COMBINATION (search + filters) RETURNED NO RESULTS
[ ] "No results for '[term]' with these filters."
[ ] "Clear search" and "Clear filters" buttons both available

GENERAL RULE
[ ] Never show a blank table body — always show one of the above states
[ ] Empty state distinguishes between "nothing exists" and "nothing matches"
```

---

# 8. Search, Filters & Pagination Checklist — Before Marking Done

```text
SEARCH
[ ] Debounced (300ms delay before request)
[ ] Loading indicator while searching
[ ] Sent to backend (not client-side array filter)
[ ] Input is parameterized (no injection risk)
[ ] Clear button when input has value
[ ] Preserved in URL param

FILTERS
[ ] Sent to backend
[ ] Active filter chips shown with X to remove each
[ ] Clear all button when any filter active
[ ] Preserved in URL params
[ ] Date range: end after start validated
[ ] Resets to page 1 on change

SEARCH + FILTERS COMBINED
[ ] Both sent in same request
[ ] Changing one does not reset the other
[ ] URL has both params

PAGINATION
[ ] Total count shown
[ ] Previous disabled on page 1, next disabled on last page
[ ] Resets to page 1 on search/filter/sort change
[ ] Page in URL param
[ ] Loading state between pages

URL STATE
[ ] Search, filter, sort, page all in URL params
[ ] Refreshing page restores exact state
[ ] Shared URL reproduces same view
[ ] Back button works correctly

EMPTY STATES
[ ] No records: message + create CTA
[ ] No search results: message + clear search
[ ] No filter match: message + clear filters
[ ] No blank table body ever
```
