# Form Field Validation Checklist

Every input field must follow these validation rules and UX standards. Use this as a reference whenever you build any form field — frontend AND backend must both validate.

---

## Contents

| Field Type | Jump To |
|------------|---------|
| Text | [#text-field](#text-field) |
| Email | [#email-field](#email-field) |
| Password | [#password-field](#password-field) |
| Confirm Password | [#confirm-password-field](#confirm-password-field) |
| Phone Number | [#phone-number-field](#phone-number-field) |
| Number (integer) | [#number-field--integer](#number-field--integer) |
| Decimal / Currency / Price | [#decimal--currency--price-field](#decimal--currency--price-field) |
| Date | [#date-field](#date-field) |
| Time | [#time-field](#time-field) |
| Date + Time | [#datetime-field](#datetime-field) |
| Location / Address | [#location--address-field](#location--address-field) |
| URL / Website | [#url--website-field](#url--website-field) |
| File Upload | [#file-upload-field](#file-upload-field) |
| Image Upload | [#image-upload-field](#image-upload-field) |
| Dropdown / Select | [#dropdown--select-field](#dropdown--select-field) |
| Multi-Select | [#multi-select-field](#multi-select-field) |
| Checkbox | [#checkbox-field](#checkbox-field) |
| Radio Button Group | [#radio-button-group](#radio-button-group) |
| Textarea | [#textarea-field](#textarea-field) |
| Search / Autocomplete | [#search--autocomplete-field](#search--autocomplete-field) |
| OTP / Verification Code | [#otp--verification-code-field](#otp--verification-code-field) |
| Percentage | [#percentage-field](#percentage-field) |
| Color Picker | [#color-picker-field](#color-picker-field) |

---

## Text Field

**UI:**
```
Full Name *
[ Enter full name        ]
```

**Validation Rules:**
```text
[ ] Required check — must not be empty or whitespace only
[ ] Trim whitespace from both ends before validation and storage
[ ] Minimum length: 2 characters (after trim)
[ ] Maximum length: defined per field (e.g. 100 chars for name, 200 for title)
[ ] No HTML or script tags allowed (sanitize on backend)
[ ] No leading/trailing spaces stored in database
```

**Error Messages:**
```text
Empty:       "[Field name] is required."
Too short:   "[Field name] must be at least 2 characters."
Too long:    "[Field name] must be under 100 characters."
```

**Storage Format:** `string` — trimmed, sanitized

---

## Email Field

**UI:**
```
Email Address *
[ Enter email address    ]
```

**Validation Rules:**
```text
[ ] Required check
[ ] Trim whitespace
[ ] Convert to lowercase before storage
[ ] Valid email format: contains @, valid domain, no spaces
[ ] Maximum length: 254 characters (email RFC standard)
[ ] No duplicate emails where uniqueness is required
[ ] Backend must validate format — do not rely only on browser input type="email"
```

**Error Messages:**
```text
Empty:    "Email address is required."
Invalid:  "Enter a valid email address (e.g. name@example.com)."
Taken:    "This email is already registered."
```

**Storage Format:** `string` — lowercased, trimmed

---

## Password Field

**UI:**
```
Password *
[ ••••••••               ] [👁 Show]
Password must be at least 8 characters with one uppercase, one number and one special character.
[████████░░░░] Strength: Good
```

**Validation Rules:**
```text
[ ] Required check
[ ] Minimum length: 8 characters (12 recommended for secure apps)
[ ] At least one uppercase letter
[ ] At least one lowercase letter
[ ] At least one number
[ ] At least one special character (recommended)
[ ] No leading/trailing spaces
[ ] Do not block copy-paste
[ ] Show/hide toggle required
[ ] Password strength indicator recommended
[ ] Block common passwords (e.g. "password123", "12345678")
[ ] Never store plain text — bcrypt/argon2 only
[ ] Never log passwords anywhere
```

**Error Messages:**
```text
Empty:     "Password is required."
Too short: "Password must be at least 8 characters."
Weak:      "Password must include at least one uppercase letter, one lowercase letter, and one number."
```

**Storage Format:** Hashed (bcrypt/argon2) — never plain text

---

## Confirm Password Field

**UI:**
```
Confirm Password *
[ ••••••••               ] [👁 Show]
```

**Validation Rules:**
```text
[ ] Required check
[ ] Must match the password field exactly
[ ] Validate on blur (when user leaves field) AND on submit
[ ] Re-validate if password field changes after confirm password was entered
[ ] Show/hide toggle required
```

**Error Messages:**
```text
Empty:      "Please confirm your password."
No match:   "Passwords do not match."
```

---

## Phone Number Field

**UI:**
```
Phone Number *
[ 🇮🇳 +91 ▼ ] [ 9876543210   ]
```

**Validation Rules:**
```text
[ ] Country code selector required — do not use a plain text input
[ ] Number input accepts digits only (no spaces, dashes, or letters)
[ ] Minimum digits: based on selected country (e.g. 10 for India, UK varies)
[ ] Maximum digits: based on selected country
[ ] Strip all non-digit characters before validation and storage
[ ] Validate number length after stripping
[ ] If optional: allow completely empty, but not partial (e.g. flag selected, no number = invalid)
[ ] Backend must also validate — never trust frontend-only phone check
```

**Error Messages:**
```text
Empty (if required): "Phone number is required."
Too short:           "Enter a valid phone number."
Wrong format:        "Enter a valid phone number for the selected country."
Partial:             "Please complete the phone number."
```

**Storage Format:** E.164 format — `+919876543210` (country code + number, no spaces/dashes)

---

## Number Field — Integer

**UI:**
```
Quantity *
[ 1                      ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Must be a whole number — no decimals
[ ] Minimum value defined (e.g. 1 for quantities, 0 for counts)
[ ] Maximum value defined
[ ] No negative values unless allowed
[ ] Do not allow letters or special characters
[ ] Validate as number on backend — do not trust string parsing
```

**Error Messages:**
```text
Empty:        "Quantity is required."
Not a number: "Enter a valid whole number."
Too low:      "Quantity must be at least 1."
Too high:     "Quantity cannot exceed 999."
```

**Storage Format:** `integer`

---

## Decimal / Currency / Price Field

**UI:**
```
Price (USD) *
[ $ 1,250.00             ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Allow decimal point and up to 2 decimal places (for currency)
[ ] Minimum value: 0 (or defined minimum — not negative unless refund/credit)
[ ] Maximum value: defined per field
[ ] Strip currency symbols and formatting before validation
[ ] Do not accept commas in value (or strip them before parsing)
[ ] Backend computes all prices — never trust frontend-sent total/price
[ ] Store in smallest currency unit (cents/paise) where possible to avoid float errors
```

**Error Messages:**
```text
Empty:    "Price is required."
Invalid:  "Enter a valid price (e.g. 1250.00)."
Too low:  "Price must be greater than 0."
```

**Storage Format:** `integer` (cents/paise) or `decimal(10,2)` — never `float`

---

## Date Field

**UI:**
```
Date of Birth *
[ DD / MM / YYYY         ] [📅]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Valid date — day/month/year must form a real calendar date
[ ] Minimum date: defined (e.g. cannot be in the past for future bookings)
[ ] Maximum date: defined (e.g. cannot be more than 1 year in the future)
[ ] For date of birth: must be in the past, user must be minimum age (if required)
[ ] Use a date picker to prevent invalid manual entry
[ ] Allow manual typing with format hint if no date picker
[ ] Store as ISO date — never store as display string
```

**Error Messages:**
```text
Empty:       "Date is required."
Invalid:     "Enter a valid date."
Past only:   "Date must be in the past."
Future only: "Date must be in the future."
Too early:   "Date cannot be before [minimum date]."
Too late:    "Date cannot be after [maximum date]."
Age:         "You must be at least 18 years old."
```

**Storage Format:** `date` — ISO 8601 format `YYYY-MM-DD`

---

## Time Field

**UI:**
```
Start Time *
[ 10 : 30  AM ▼ ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Valid time — hour/minute must be in valid range
[ ] 12-hour (AM/PM) or 24-hour format — defined consistently per project
[ ] Minimum time: defined if needed (e.g. business hours only: 09:00–18:00)
[ ] Maximum time: defined if needed
[ ] If selecting a range: end time must be after start time
[ ] Use a time picker — avoid free-text input for time
```

**Error Messages:**
```text
Empty:        "Time is required."
Invalid:      "Enter a valid time."
Out of range: "Time must be between 9:00 AM and 6:00 PM."
End < Start:  "End time must be after start time."
```

**Storage Format:** `time` or `string` — 24-hour format `HH:MM:SS` in UTC

---

## Datetime Field

**UI:**
```
Meeting Date & Time *
[ 15 May 2026        ] [ 10 : 30  AM ▼ ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Both date AND time must be provided if field is required
[ ] Validate date and time independently first, then combined
[ ] Minimum datetime: e.g. cannot be in the past for scheduling
[ ] Maximum datetime: e.g. cannot be more than 6 months ahead
[ ] Timezone: capture user's timezone or store in UTC with timezone label
[ ] If selecting a range: end datetime must be after start datetime
```

**Error Messages:**
```text
Empty:         "Date and time are required."
Past not allowed: "Select a future date and time."
Range invalid: "End must be after start."
```

**Storage Format:** `timestamptz` — UTC with timezone `YYYY-MM-DDTHH:MM:SSZ`

---

## Location / Address Field

**UI:**
```
Location *
[ Start typing a location... ] (Google Places Autocomplete)

   — OR structured fields —

Street Address *     [ 123 Main Street              ]
City *               [ Mumbai                       ]
State / Province     [ Maharashtra                  ]
Country *            [ India              ▼         ]
Postal Code *        [ 400001                       ]
```

**Validation Rules:**
```text
[ ] If using Google Places Autocomplete:
    [ ] Require selection from dropdown — do not accept free-text without validation
    [ ] Store structured address components (street, city, state, country, postal, lat/lng)
    [ ] Store latitude and longitude for map display and geo queries
    [ ] Handle case where user types but does not select from dropdown

[ ] If using structured fields:
    [ ] Street address: required, min 5 characters
    [ ] City: required, text only
    [ ] Country: required, must be from allowed country list
    [ ] Postal code: validate format per country (e.g. 6 digits for India, ZIP+4 for US)
    [ ] State: required if country has states/provinces

[ ] Backend must validate address fields — not just the frontend
[ ] Never store raw user-typed location string as the canonical address
```

**Error Messages:**
```text
Empty:              "Location is required."
Not selected:       "Please select a location from the suggestions."
Invalid postal:     "Enter a valid postal code."
Country required:   "Please select a country."
```

**Storage Format:**
```text
address_line_1: string
city:           string
state:          string
country:        string (ISO 3166-1 alpha-2, e.g. "IN", "US")
postal_code:    string
latitude:       decimal(10, 8)
longitude:      decimal(11, 8)
```

---

## URL / Website Field

**UI:**
```
Website URL
[ https://www.example.com  ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Must start with http:// or https:// — reject bare domain names
[ ] Valid URL format (no spaces, valid characters)
[ ] Maximum length: 2048 characters
[ ] Trim whitespace
[ ] Do not auto-prefix http:// without user knowing
[ ] Backend must validate URL format
```

**Error Messages:**
```text
Empty:    "Website URL is required."
Invalid:  "Enter a valid URL starting with http:// or https://"
```

**Storage Format:** `string` — trimmed, stored as entered

---

## File Upload Field

**UI:**
```
Attach Document
[ 📎 Choose File ]  or drag and drop
Accepted: PDF, DOCX — Max size: 5MB
```

**Validation Rules:**
```text
[ ] Allowed file types: explicitly defined (e.g. PDF, DOCX, XLSX only)
[ ] Validate by MIME type on backend — not just file extension
[ ] Maximum file size: explicitly defined (e.g. 5MB)
[ ] Minimum file size: 1 byte (reject empty files)
[ ] File name: sanitize before storage — no path traversal characters
[ ] Storage path: must include user/workspace scope (e.g. workspace/{id}/files/{uuid}.pdf)
[ ] Do not expose original file name directly in storage path
[ ] Show file name and size to user after selection
[ ] Show upload progress indicator during upload
[ ] Handle upload failure gracefully — show error, allow retry
[ ] Frontend validation shows error before upload attempt
[ ] Backend must re-validate file type and size — frontend checks are bypassable
```

**Error Messages:**
```text
Wrong type:  "Only PDF and DOCX files are allowed."
Too large:   "File must be under 5MB."
Empty file:  "The selected file appears to be empty."
Upload fail: "Upload failed. Please try again."
```

**Storage Format:** Store in cloud storage (Supabase Storage / S3). Store URL and metadata (size, MIME type, original name) in database.

---

## Image Upload Field

**UI:**
```
Profile Photo
[ 📷 Upload Photo ]   [Preview circle]
Accepted: JPG, PNG, WEBP — Max: 2MB — Min: 100×100px
```

**Validation Rules:**
```text
[ ] Allowed types: JPG, JPEG, PNG, WEBP — no GIF, SVG, BMP unless required
[ ] Validate by MIME type on backend — not extension
[ ] Maximum file size: defined (e.g. 2MB for profile photos, 10MB for hero images)
[ ] Minimum dimensions: defined if needed (e.g. 100×100px for profile photos)
[ ] Maximum dimensions: defined if needed
[ ] Show preview before finalizing upload
[ ] Compress/resize on backend if needed
[ ] Strip EXIF data for privacy (GPS location, device info)
[ ] Storage path scoped to user/workspace
[ ] Backend must re-validate
```

**Error Messages:**
```text
Wrong type:  "Please upload a JPG or PNG image."
Too large:   "Image must be under 2MB."
Too small:   "Image must be at least 100×100 pixels."
```

---

## Dropdown / Select Field

**UI:**
```
Status *
[ Active         ▼ ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Value must match one of the defined allowed options
[ ] Do not trust the option value sent from frontend — validate against allowed list on backend
[ ] Default placeholder option should not be a valid selection (e.g. "-- Select status --")
[ ] If the list is dynamic (from database), validate that the selected ID exists and is accessible
[ ] For role selection: validate that the selected role is allowed for the current user's context
```

**Error Messages:**
```text
Empty:    "Please select a [status / category / role]."
Invalid:  "Selected option is not valid."
```

**Storage Format:** `string` (enum value) or `uuid` (if selecting a related record)

---

## Multi-Select Field

**UI:**
```
Tags
[ × React   × Supabase   × WeWeb   +Add ]
```

**Validation Rules:**
```text
[ ] Required minimum selection count (if required — e.g. "select at least 1")
[ ] Maximum selection count (if applicable)
[ ] Each selected value must be from the allowed list
[ ] Validate on backend that all submitted IDs/values exist
[ ] Prevent duplicate selections
```

**Error Messages:**
```text
Empty (required): "Please select at least one [tag/category]."
Too many:         "You can select a maximum of 5 items."
```

---

## Checkbox Field

**Single Checkbox (Agreement / Toggle):**

**UI:**
```
☑  I agree to the Terms of Service and Privacy Policy
```

**Validation Rules:**
```text
[ ] If required (e.g. terms acceptance): must be checked — cannot submit unchecked
[ ] Never pre-check a consent/agreement checkbox
[ ] Backend must verify terms version accepted if recording consent
```

**Error Messages:**
```text
Unchecked: "You must accept the Terms of Service to continue."
```

---

## Radio Button Group

**UI:**
```
Account Type *
◉ Individual
○ Business
○ Agency
```

**Validation Rules:**
```text
[ ] Required check — one option must be selected
[ ] Default selection is acceptable for non-sensitive choices
[ ] Value sent must match one of the defined options
[ ] Backend must validate selected value
[ ] If "Other" is an option, show a text field when selected
```

**Error Messages:**
```text
None selected: "Please select an account type."
```

---

## Textarea Field

**UI:**
```
Message *
[ Type your message here...              ]
[                                        ]
[                            250/1000    ]
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Trim whitespace before validation
[ ] Minimum length: defined (e.g. 10 characters for a meaningful message)
[ ] Maximum length: defined (e.g. 1000 characters)
[ ] Show character counter: current/max
[ ] Warn when approaching limit (e.g. at 80%)
[ ] No HTML injection — sanitize on backend
[ ] Newlines preserved in storage if multi-line content is expected
```

**Error Messages:**
```text
Empty:     "Message is required."
Too short: "Message must be at least 10 characters."
Too long:  "Message cannot exceed 1000 characters."
```

**Storage Format:** `text` — newlines preserved, HTML stripped

---

## Search / Autocomplete Field

**UI:**
```
Search projects...
[ Project N▊                ]
  ↓ Project Nova
  ↓ Project Network
  ↓ Project Neon
```

**Validation Rules:**
```text
[ ] Debounce search input — wait 300ms after last keystroke before firing API call
[ ] Minimum characters before search triggers: 2 or 3
[ ] Show loading indicator during search
[ ] Handle "no results" state gracefully
[ ] Sanitize search input before sending to backend — prevent injection
[ ] Backend must parameterize search queries — never interpolate into raw SQL
[ ] Limit results returned: e.g. max 10–20 suggestions
[ ] If user must select from results (not free-text): validate selection is from result set
```

**Error Messages:**
```text
No results: "No results found for '[term]'."
```

---

## OTP / Verification Code Field

**UI:**
```
Enter the 6-digit code sent to +91 9876543210

[ 3 ] [ 8 ] [ 4 ] [ 1 ] [ 9 ] [ 2 ]

Resend code in 00:45
```

**Validation Rules:**
```text
[ ] Digits only — no letters or special characters
[ ] Exact length required (e.g. 4 or 6 digits)
[ ] Auto-advance focus to next box after each digit entry
[ ] Allow paste — fill all boxes from pasted value
[ ] OTP expiry: token must be validated for expiry on backend
[ ] Max attempts: lock or throttle after N wrong attempts (e.g. 5)
[ ] OTP must be single-use — invalidate after correct verification
[ ] Resend cooldown: prevent spam (e.g. 60-second cooldown)
[ ] Backend generates and verifies OTP — never trust client-sent value
```

**Error Messages:**
```text
Incomplete:  "Please enter all 6 digits."
Wrong code:  "Incorrect code. Please try again."
Expired:     "This code has expired. Please request a new one."
Too many:    "Too many attempts. Please request a new code."
```

---

## Percentage Field

**UI:**
```
Discount *
[ 15                     ] %
```

**Validation Rules:**
```text
[ ] Required check (if required)
[ ] Minimum value: 0
[ ] Maximum value: 100
[ ] Decimal places: 0 or 2 — defined per use case
[ ] Do not accept negative values unless specifically required
[ ] Backend must validate range — frontend check is not enough
```

**Error Messages:**
```text
Empty:    "Discount percentage is required."
Too low:  "Percentage cannot be less than 0."
Too high: "Percentage cannot exceed 100."
```

**Storage Format:** `decimal(5,2)` — e.g. `15.00`

---

## Color Picker Field

**UI:**
```
Brand Color
[ 🎨 #2563EB        ] [████]
```

**Validation Rules:**
```text
[ ] Accept valid hex code: # followed by exactly 6 hex characters (0–9, A–F)
[ ] Accept 3-character hex shorthand and expand to 6 (e.g. #FFF → #FFFFFF)
[ ] Trim whitespace
[ ] Case-insensitive — store in consistent case (all uppercase or all lowercase)
[ ] Allow color picker UI alongside hex input
[ ] Validate on backend that value is a valid hex color
```

**Error Messages:**
```text
Invalid: "Enter a valid color code (e.g. #2563EB)."
```

**Storage Format:** `string` — 7 characters including `#`, uppercase: `#2563EB`

---

## Universal Field Rules (Apply to Every Field)

```text
[ ] Every field has a visible label — not just a placeholder
[ ] Required fields are marked with * (asterisk)
[ ] Optional fields are clearly labeled "(optional)" where helpful
[ ] Field-level error appears near the field — not only at the top of the form
[ ] Error disappears when user corrects the field (validate on change after first submit)
[ ] Placeholder text is a hint — not a substitute for a label
[ ] Disabled fields have muted style + cursor: not-allowed
[ ] Disabled fields show a tooltip explaining why they are disabled
[ ] Tab key order is logical (top to bottom, left to right)
[ ] All fields work on mobile — inputs are full width, touch-friendly
[ ] Backend validates every field — frontend validation is UX only
```

---

## Practice Task

Apply what you learned by building a real form with multiple field types, validation rules, and error states.

**→ [Task 01: Build a User Registration Form](../../tasks/frontend/01-registration-form.md)**

Covers: text, email, password strength + complexity, confirm password, phone (E.164 + country code), checkbox, loading state, duplicate email error, success state.
