# Multi-Step Forms & Wizards Checklist

> **Core Rule:** Each step validates before advancing. The user can always go back without losing their data. Progress is visible. The final step — and only the final step — submits to the backend.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-step-structure) | Step Structure |
| [2](#2-navigation-between-steps) | Navigation Between Steps |
| [3](#3-validation-per-step) | Validation Per Step |
| [4](#4-progress-indicator) | Progress Indicator |
| [5](#5-data-preservation) | Data Preservation |
| [6](#6-final-step--submission) | Final Step & Submission |
| [7](#7-saving-progress--draft) | Saving Progress & Draft |
| [8](#8-mobile) | Mobile |
| [9](#9-multi-step-form-checklist--before-marking-done) | Checklist — Before Marking Done |

---

# 1. Step Structure

```text
[ ] Total number of steps is fixed and known upfront (not dynamically changing mid-flow unless intentional branching)
[ ] Each step has a clear title and short description of what the user is doing in that step
[ ] Each step contains a logically grouped set of fields — related fields together, unrelated fields in a different step
[ ] No single step has so many fields that it becomes overwhelming (aim for 3–6 fields per step)
[ ] The final step is a Review & Submit step (optional but recommended for longer flows):
  — Shows a summary of all entered data
  — Allows the user to go back and edit any step
  — Final "Submit" button on this step
[ ] Step order is logical — information collected in earlier steps should not become obsolete in later steps
```

---

# 2. Navigation Between Steps

```text
[ ] "Next" button advances to the next step
[ ] "Back" button returns to the previous step — always available after step 1
[ ] Back does NOT submit or save data — it just navigates
[ ] Back does NOT lose the data entered in the current step
[ ] Steps can be clicked directly in the progress indicator if already visited (optional but good UX)
[ ] Do NOT allow advancing to the next step until the current step's validation passes
[ ] Current step number and total shown: "Step 2 of 4"
[ ] "Cancel" or "Exit" link available to abandon the flow — with a "Discard progress?" confirmation
[ ] Refresh / page reload: user is warned about losing progress (if data is not saved to backend per step)
```

---

# 3. Validation Per Step

```text
[ ] Validation runs only for fields in the current step — not all steps at once
[ ] "Next" button triggers validation of the current step's fields before advancing
[ ] Validation errors shown inline (same as standard form validation — see [Form Field Validation](form-field-validation.md))
[ ] User cannot advance with any invalid field in the current step
[ ] Revisiting a previous step preserves data AND re-shows any errors that were already present
[ ] On the Review step: each section is validated before final submit
[ ] Cross-step validation (e.g., end date in step 3 must be after start date set in step 2): validated on the step where the dependent field exists, OR on the review/submit step
[ ] Schema for each step is a subset of the full form schema — not duplicated, but derived from the same schema object
```

---

# 4. Progress Indicator

```text
[ ] Progress indicator is visible throughout the entire flow
[ ] Shows: completed steps (✓), current step (active/highlighted), future steps (inactive)
[ ] Step labels shown (not just numbers): "1. Account", "2. Workspace", "3. Invite", "4. Review"
[ ] On mobile: compact progress indicator (step number only, or abbreviated labels)
[ ] Clicking a completed step navigates back to it (if editing is allowed)
[ ] Future steps are not clickable (cannot skip ahead without completing current step)
[ ] Progress indicator does not change height/layout between steps (stable layout)
```

---

# 5. Data Preservation

```text
[ ] Going back to a previous step: all previously entered data is still present
[ ] Advancing from a step and returning: data is still present
[ ] Page refresh: prompt user ("You'll lose your progress") OR preserve data in:
  — localStorage (for non-sensitive data)
  — URL params (for simple, non-sensitive values)
  — Backend draft (for important or sensitive flows — see Section 7)
[ ] Browser back button: if supported, treat the same as the Back button
[ ] Data from step 1 used in later steps (e.g., workspace name used in a heading): updates automatically if step 1 is edited
[ ] Sensitive data (passwords, card numbers) NOT stored in localStorage or URL — always in memory or backend session
```

---

# 6. Final Step & Submission

```text
[ ] Only the final step triggers the backend API call (not each step individually — unless saving draft per step)
[ ] Submit button shows a loading state during submission
[ ] Submit button disabled during loading — no double submit
[ ] If submission fails: stay on the final step, show the error, allow retry — do NOT reset the form
[ ] If submission fails for a field in an earlier step: navigate back to that step and show the error there
[ ] On success: navigate to the success page or the created entity — do NOT stay on the form
[ ] Success page shows what was created/completed with a clear next step
[ ] Form data cleared from memory/localStorage after successful submission
[ ] No re-submission if the user presses back after success
```

---

# 7. Saving Progress & Draft

For longer flows or flows where abandonment is common:

```text
[ ] Draft saved to backend when user exits or abandons mid-flow (if the feature supports it)
[ ] Draft save indicator shown: "Progress saved automatically." or a "Save and exit" button
[ ] User can return to resume from where they left off
[ ] Drafts have an expiry (don't accumulate forever)
[ ] Draft data treated as unverified — re-validated on final submit
[ ] If no draft support: "You'll lose your progress. Are you sure you want to exit?" confirmation when cancelling
```

---

# 8. Mobile

```text
[ ] Progress indicator compact on mobile (step number, abbreviated labels)
[ ] Each step is a full-screen view on mobile — no horizontal scroll
[ ] Back and Next buttons at the bottom, full width (large tap targets)
[ ] Form fields full width and touch-friendly
[ ] Keyboard does not cover the active input on mobile (scroll form when keyboard appears)
[ ] Review step scrollable on mobile — no content cut off
```

---

# 9. Multi-Step Form Checklist — Before Marking Done

```text
STRUCTURE
[ ] Fixed number of steps
[ ] Each step has a title and description
[ ] Logical grouping of fields per step
[ ] Review step present for longer flows (3+ steps)

NAVIGATION
[ ] Next validates current step before advancing
[ ] Back always available after step 1
[ ] Back does not lose current step data
[ ] Cancel shows "Discard progress?" confirmation
[ ] Step counter visible: "Step X of Y"

VALIDATION
[ ] Validation runs for current step fields only
[ ] Cannot advance with invalid fields
[ ] Errors shown inline (same as standard form)
[ ] Schema shared across steps (not duplicated)

PROGRESS INDICATOR
[ ] Completed, current, and future steps visually distinct
[ ] Step labels shown
[ ] Completed steps clickable to edit
[ ] Future steps not clickable

DATA PRESERVATION
[ ] Back: all previous data preserved
[ ] Refresh: prompt or persistence implemented
[ ] Sensitive data not in localStorage or URL
[ ] Step 1 data reflected in later steps dynamically

SUBMISSION
[ ] API called only on final step
[ ] Loading state on submit button
[ ] No double submit possible
[ ] Failure: stay on form, show error, allow retry
[ ] Success: navigate away, clear form data
[ ] No re-submission on back after success

MOBILE
[ ] Compact progress indicator
[ ] Full-screen steps (no horizontal scroll)
[ ] Large tap targets for navigation
[ ] Keyboard does not cover inputs
```
