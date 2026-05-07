# Frontend Standards — UI Components

> Modals, notifications, loading states, skeletons, and buttons. Every component must be accessible, consistent, and handle every state before it is considered done.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-modals--dialogs) | Modals & Dialogs |
| [2](#2-notifications--toasts) | Notifications & Toasts |
| [3](#3-loading-states) | Loading States |
| [4](#4-skeleton-screens) | Skeleton Screens |
| [5](#5-buttons) | Buttons |
| [6](#6-badges--status-indicators) | Badges & Status Indicators |
| [7](#7-tooltips) | Tooltips |
| [8](#8-dropdowns--select-menus) | Dropdowns & Select Menus |
| [9](#9-tabs) | Tabs |
| [10](#10-accordion--expandable) | Accordion & Expandable |

---

# 1. Modals & Dialogs

### Opening

```text
[ ] Triggered by a user action — not on page load (except critical interruptions like cookie consent)
[ ] Backdrop darkens behind the modal
[ ] Background scroll locked (body overflow: hidden)
[ ] Focus moves to the modal on open (first focusable element)
[ ] Screen reader announces the modal (role="dialog", aria-modal="true", aria-labelledby)
[ ] Opening animation: fade or slide, under 200ms
```

### Closing

```text
[ ] X close button in top-right
[ ] Escape key closes the modal
[ ] Clicking backdrop closes (unless unsaved form inside)
[ ] Focus returns to the trigger element on close
[ ] Background scroll restored on close
[ ] Unsaved form: "Discard changes?" confirmation before closing
[ ] In-progress action: Escape and backdrop disabled until complete
```

### Structure

```text
[ ] Visible title (h2 or h3 connected via aria-labelledby)
[ ] Primary action on the right, cancel on the left
[ ] Destructive primary action: red/danger styled
[ ] Content scrolls inside the modal — not the page behind it
[ ] Width appropriate to content: confirm = narrow, form = medium, preview = wide
[ ] Does not exceed viewport height without an internal scroll area
```

### Confirmation Modals

```text
Low risk (Archive, soft actions):
  Title:  "Archive Project?"
  Body:   What happens + whether it is reversible
  Button: Action label (amber if reversible, red if not)

Medium risk (Delete a record):
  Title:  "Delete Project?"
  Body:   "[Name]" specifically + consequence + affected count
  Body:   "This action cannot be undone." if irreversible
  Button: "Delete Project" — red/danger

High risk (Delete account, all data):
  Same as medium + type-to-confirm input ("Type DELETE to confirm")
  Button: Disabled until input matches exactly (case-sensitive)
```

### Accessibility

```text
[ ] role="dialog" + aria-modal="true" on modal root
[ ] aria-labelledby pointing to title element
[ ] Focus trapped inside modal (Tab cycles within modal only)
[ ] Background content aria-hidden="true" while modal is open
[ ] Escape always works as a cancel
```

---

# 2. Notifications & Toasts

### Types and Purpose

| Type | Color | When | Duration |
|------|-------|------|----------|
| Success | Green | Async action completed | 4 seconds |
| Error | Red | Action failed | 6 seconds |
| Warning | Amber | Completed with caveat | 5 seconds |
| Info | Blue | Neutral information | 5 seconds |

### Behavior

```text
[ ] Position: top-right (desktop), top-center or bottom-center (mobile)
[ ] Z-index above all content including modals
[ ] X button on every toast for manual dismiss
[ ] Hover pauses auto-dismiss timer
[ ] Multiple toasts stack vertically — max 3 visible at once, others queued
[ ] New toasts appear at top of stack, older shift down
```

### Content Rules

```text
[ ] Specific: "Project 'Marketing Q1' deleted." not "Success!"
[ ] Past tense for completed actions: "Project saved." not "Saving project."
[ ] Error messages: human-readable, no error codes, no stack traces
[ ] Optional action in toast: "Undo" (5-second window) or "View"
[ ] No exclamation marks on error messages
[ ] Same event always produces the same message (consistent vocabulary)
```

### What Triggers a Toast

```text
Always:
+ Create → success toast
+ Update/save → success toast
+ Delete → success toast (+ optional undo)
+ Send/invite → success toast
+ Any async failure → error toast or inline banner

Never:
× Page load or data fetch completing
× Navigation between pages
× Opening or closing a modal
× Client-side validation failure (use inline field errors)
× Background auto-save (use a subtle "Saved ✓" indicator, not a toast)
```

---

# 3. Loading States

### Buttons

```text
[ ] Spinner replaces or appears next to label while action is in progress
[ ] Button disabled during loading — no double submit
[ ] Button label changes to reflect action: "Saving..." / "Deleting..." / "Sending..."
[ ] Button width does not change between default and loading state (fixed width/min-width)
[ ] Button returns to normal after success or failure
```

### Standard Loading Labels

| Default | Loading |
|---------|---------|
| Create / Add | Creating... |
| Save Changes | Saving... |
| Delete | Deleting... |
| Send / Invite | Sending... |
| Upload | Uploading... |
| Export | Exporting... |
| Submit | Submitting... |

### Page & Section Loading

```text
[ ] Full page fetch: skeleton shown — never blank white page
[ ] Layout frame (header/nav) visible while skeleton loads
[ ] Skeleton is the initial render state — no flash of blank before skeleton
[ ] Spinner (centered): only for very short operations < 500ms; prefer skeleton for longer
[ ] Error state shown if load fails — with retry button, never a blank area
```

---

# 4. Skeleton Screens

```text
SHAPE
[ ] Skeleton matches the approximate shape of the real content:
  — Card grid → skeleton cards in a grid
  — Table → skeleton rows with column widths matching real data
  — Form in Edit mode → skeleton inputs and labels
  — Detail page → skeleton for each section

BEHAVIOR
[ ] Animated shimmer / pulse effect — not a static grey block
[ ] Number of skeleton items matches expected data count (page size for tables)
[ ] Skeleton disappears instantly when data arrives (no delayed swap)
[ ] No text content in skeleton — only grey placeholder shapes

WHEN TO USE
Skeleton:   Initial page/section load, tables, card grids, form pre-fill
Spinner:    Button action, small inline refresh, short operations < 300ms
```

---

# 5. Buttons

```text
TYPES
[ ] Primary: one per view — the main action (filled, brand color)
[ ] Secondary: supporting actions (outlined or ghost)
[ ] Danger: destructive actions (red fill or red outline)
[ ] Ghost/link: tertiary actions (no background)

RULES
[ ] Never use <div> or <span> as a button — always <button> or <a href>
[ ] <button> for actions (submit, open modal, delete)
[ ] <a href> for navigation (links to another page/URL)
[ ] All buttons have a visible focus state (keyboard accessible)
[ ] Disabled state: visually distinct (reduced opacity, cursor not-allowed)
[ ] Disabled buttons still need a tooltip if the reason is not obvious
[ ] Icon-only buttons have an aria-label: aria-label="Delete project"
[ ] Button text is an action verb: "Save Changes", "Delete Project" — not "OK" or "Submit"
[ ] Loading state handled per the Loading States section above
[ ] Touch target minimum: 44×44px on mobile

ICON BUTTONS
[ ] Icon + label preferred over icon-only for clarity
[ ] If icon-only: aria-label required
[ ] Consistent icon library — do not mix icon sets
```

---

# 6. Badges & Status Indicators

```text
[ ] Status values always shown as a badge — never plain text in a table
[ ] Badge colors are consistent throughout the app (Active = green, always)
[ ] Badge colors use CSS variables — never hardcoded hex per badge
[ ] Badge font size: small (12–13px), font weight: medium
[ ] Status badge classes:
  .badge-active    → green background
  .badge-on-hold   → amber background
  .badge-archived  → grey background
  .badge-error     → red background
[ ] Badge accessible: color is not the ONLY differentiator (also use text label)
[ ] New status values: add to the shared badge component/constants — not inline per-page
```

---

# 7. Tooltips

```text
[ ] Used for: explaining disabled UI elements, supplementary info, truncated text preview
[ ] NOT used for: critical information the user cannot find elsewhere
[ ] Tooltip appears on hover (desktop) and on long-press or tap (mobile)
[ ] Tooltip text is short: one sentence maximum
[ ] Tooltip position: avoids going off-screen (auto-flip if near edge)
[ ] Disabled buttons MUST have a tooltip explaining why they are disabled
[ ] Tooltip does not block nearby interactive elements
[ ] Tooltip has a slight delay (100–150ms) before appearing to avoid flicker on fast mouse-overs
[ ] Tooltips do not contain links or interactive elements — use a popover for that
```

---

# 8. Dropdowns & Select Menus

```text
[ ] Native <select> acceptable for simple cases, custom component for branded UI
[ ] Options fetched from backend — not hardcoded in frontend unless truly static
[ ] Loading state shown while options are fetching (spinner inside dropdown)
[ ] Searchable dropdown for > 7 options (user can type to filter)
[ ] Dropdown closes on: option select, outside click, Escape key
[ ] Selected value shown clearly inside the dropdown trigger
[ ] Placeholder shown when no value selected: "Select status..."
[ ] Disabled options visually distinct with a tooltip explaining why
[ ] Dropdown max height with internal scroll for long option lists
[ ] Keyboard navigable: arrow keys to move, Enter to select, Escape to close
[ ] Multi-select: selected values shown as chips inside or below the trigger
```

---

# 9. Tabs

```text
[ ] Active tab visually distinct (underline, background, or color change)
[ ] Tab content loads when tab is clicked — not all upfront (lazy loading)
[ ] Loading state inside tab panel while content loads
[ ] URL updated with active tab: ?tab=members (makes it shareable and bookmarkable)
[ ] Tab order restored when user navigates back
[ ] First tab active by default — or the tab from the URL param
[ ] Tab labels are short nouns: "Members" not "View Members"
[ ] Keyboard navigable: arrow keys to move between tabs
[ ] Tab content not destroyed on tab switch (maintain form state if inside a tab)
```

---

# 10. Accordion & Expandable

```text
[ ] Expand/collapse triggered by clicking the header area — not just a small arrow icon
[ ] Expand/collapse icon (chevron) rotates to show state
[ ] Only one section open at a time OR multiple — decide and be consistent
[ ] Content transition: smooth height animation (150–200ms)
[ ] Expanded state preserved across page interactions if relevant
[ ] Keyboard: Enter or Space to toggle
[ ] aria-expanded on the trigger, aria-controls pointing to the content panel
[ ] Default state: first item open OR all closed — document the choice
```
