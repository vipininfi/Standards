# Task: Replicate a Landing Page — Hero + Features + CTA Sections

**Platform:** React (or WeWeb)
**Covers:** [Website Replication Standards](../../standards/website/README.md) · [Website Launch Checklist](../../checklists/website-launch-checklist.md) · [Responsive QA Standards](../../standards/website/04-responsive-qa-performance.md)

---

## Scenario

A client wants their existing marketing landing page rebuilt in React (or WeWeb). The original is built in WordPress and is slow. You have been provided a Figma file with the design. Your job is to replicate the **Hero section**, **Features section**, and **Call-to-Action section** as a pixel-perfect replica, fully responsive across desktop, tablet, and mobile.

This is a **Pixel-Perfect Replica** — low tolerance for layout differences.

---

## What to Build

Three reusable section components that compose the landing page:

| Component | Content |
|-----------|---------|
| `HeroSection` | Headline, subheadline, CTA button, hero image |
| `FeaturesSection` | Grid of 6 feature cards (icon + title + description) |
| `CtaSection` | Bold headline, short paragraph, email signup form |

---

## Design Specifications (extract from Figma before coding)

Before writing a single line of code, document these from the Figma file:

### Design Tokens

```text
Colors:
  Primary:   #0070F3
  Secondary: #111827
  Text:      #374151
  Muted:     #6B7280
  Background:#FFFFFF
  Card BG:   #F9FAFB
  Accent:    #F59E0B

Typography:
  Heading font:  Inter, 700 weight
  Body font:     Inter, 400 weight
  H1 size:       56px desktop / 36px mobile
  H2 size:       40px desktop / 28px mobile
  Body size:     18px desktop / 16px mobile
  Small:         14px

Spacing (8pt grid):
  Section padding: 96px top/bottom desktop / 64px mobile
  Card padding:    32px
  Gap between cards: 24px

Breakpoints:
  Mobile:  < 768px
  Tablet:  768px – 1024px
  Desktop: > 1024px
```

---

## Part 1 — Hero Section

### Layout

- Left: headline, subheadline, CTA button
- Right: hero image (provide your own placeholder if no image in Figma)
- On mobile: stacks vertically (image above text, or text above image — check Figma)

### Content

```
Headline:    "Build projects that matter."
Subheadline: "WorkFlow helps your team stay organized, move faster, and ship with confidence."
CTA Button:  "Get started free" (links to /signup)
Hero Image:  App dashboard screenshot (placeholder acceptable)
```

### Requirements

- Headline is an `<h1>` tag — one `<h1>` per page
- CTA button is an `<a>` tag linking to `/signup` — not a `<button>` tag (it navigates)
- Hero image has a meaningful `alt` attribute (not `alt=""` or `alt="image"`)
- Section has `id="hero"` for anchor linking
- Fully responsive: image and text side-by-side on desktop, stacked on mobile

---

## Part 2 — Features Section

### Layout

- Section heading centered above the grid
- 6 feature cards in a 3-column grid (desktop), 2-column (tablet), 1-column (mobile)
- Each card: icon + title + description

### Feature Cards Content

| # | Icon | Title | Description |
|---|------|-------|-------------|
| 1 | ✓ | Task Management | Organize work into projects and tasks with due dates and priorities. |
| 2 | ⚡ | Real-Time Updates | See changes from your team the moment they happen. |
| 3 | 🔒 | Role-Based Access | Control who can view, edit, or manage each project. |
| 4 | 📊 | Progress Tracking | Visual dashboards that show exactly where every project stands. |
| 5 | 🔗 | Integrations | Connect with Slack, GitHub, and the tools your team already uses. |
| 6 | 📱 | Mobile Ready | Full access from any device — phone, tablet, or desktop. |

### Requirements

- Section heading is an `<h2>` tag
- Card titles are `<h3>` tags
- Cards are built as a reusable `FeatureCard` component (not 6 separate blocks)
- Card grid collapses correctly at each breakpoint (test at 375px, 768px, 1280px)
- No hardcoded inline styles — all layout via CSS classes or CSS-in-JS consistent with your tech choice

---

## Part 3 — CTA Section

### Layout

- Full-width section with a dark/colored background
- Centered headline + paragraph + email input + submit button

### Content

```
Headline:  "Ready to get organized?"
Paragraph: "Join thousands of teams already using WorkFlow. Free plan available."
Input:     Email address placeholder ("Enter your email")
Button:    "Start for free"
```

### Requirements

- Email input: `type="email"`, `required`, `placeholder`, `autocomplete="email"`
- On submit:
  - Validate email format client-side before sending
  - Show loading state on the button while submitting
  - On success: replace the form with "Thanks! Check your inbox."
  - On error: show "Something went wrong. Please try again." — never just `console.log`
- Button is a `<button type="submit">` inside a `<form>` — not a standalone `<div>`
- Section has `id="cta"` for anchor linking

---

## Responsive Requirements

Test at all three breakpoints before marking done:

| Element | Desktop (1280px) | Tablet (768px) | Mobile (375px) |
|---------|-----------------|----------------|----------------|
| Hero layout | Side-by-side | Side-by-side | Stacked |
| Hero headline | 56px | 44px | 36px |
| Feature grid | 3 columns | 2 columns | 1 column |
| CTA form | Inline (input + button side by side) | Inline | Stacked |
| Section padding | 96px | 80px | 64px |

---

## Performance Requirements

Before marking done:

```text
[ ] All images have explicit width and height attributes (prevents layout shift)
[ ] Images use WebP format (or have a fallback)
[ ] Images have descriptive alt text
[ ] No image is wider than its display size (no loading a 2000px image to display at 400px)
[ ] Fonts loaded with font-display: swap
[ ] No render-blocking scripts
[ ] Lighthouse Performance score above 90 on desktop
```

---

## What You Should NOT Do

- Do not build all six feature cards as individual components — build one `FeatureCard` and map over data
- Do not hardcode colors, fonts, or spacing that should come from design tokens
- Do not use `<div>` for the CTA button — it is an interactive element, use `<button>`
- Do not use `<div>` for the CTA link — links that navigate should be `<a href>` tags
- Do not skip mobile testing — test on an actual 375px viewport (not just "it looks fine on my laptop")
- Do not use inline styles for layout — use CSS classes
- Do not `console.log(error)` in the CTA form submission — show the error to the user

---

## Checklist to Run When Done

Use the [Website Launch Checklist](../../checklists/website-launch-checklist.md) — Design/Visual, Responsive, and Performance sections.

---

## Done When

```text
DESIGN ACCURACY
[ ] Hero layout matches Figma at all breakpoints
[ ] Feature card layout matches Figma at all breakpoints
[ ] CTA section matches Figma at all breakpoints
[ ] Colors, fonts, and spacing match documented design tokens
[ ] No hardcoded colors that differ from design tokens

COMPONENTS
[ ] HeroSection is a reusable component
[ ] FeaturesSection uses a single FeatureCard component with data mapping
[ ] CtaSection is a reusable component

HTML SEMANTICS
[ ] One <h1> on the page (Hero headline)
[ ] Feature section uses <h2> and cards use <h3>
[ ] CTA button is <button type="submit">
[ ] Hero CTA is <a href="/signup">
[ ] Images have descriptive alt attributes

CTA FORM
[ ] Email validated before submission
[ ] Loading state on button during submit
[ ] Success: form replaced with thank you message
[ ] Error: shown to user (not just logged)

RESPONSIVE (tested at actual viewport sizes)
[ ] Hero: side-by-side at 1280px, stacked at 375px
[ ] Feature grid: 3 col at 1280px, 2 col at 768px, 1 col at 375px
[ ] CTA form: inline at desktop, stacked at mobile
[ ] No horizontal scroll at any breakpoint
[ ] Text is readable at all sizes (no overflow or cut-off)

PERFORMANCE
[ ] Images have explicit width and height
[ ] Images are optimized (WebP / correct dimensions)
[ ] Lighthouse performance > 90 on desktop
[ ] No images loading without alt text
```
