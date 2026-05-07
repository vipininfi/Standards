# Responsive, QA, Performance & Accessibility

> **Part of:** [Website Replication Standards](<README.md>)

**Covers:** Sections 18–26 — Responsive replication standard, pixel-perfect QA, performance rules, accessibility, security, CMS decisions, no-code standards, naming standards, animation replication

---

# 18. Responsive Replication Standard

A replica must be tested across breakpoints.

## Breakpoints

| Device        |         Width |
| ------------- | ------------: |
| Small Mobile  |         360px |
| Mobile        | 390px / 414px |
| Tablet        |         768px |
| Small Laptop  |        1024px |
| Desktop       |        1280px |
| Large Desktop |       1440px+ |

## Responsive Checklist

| Area    | Standard                         |
| ------- | -------------------------------- |
| Header  | Mobile menu works.               |
| Hero    | Text/image stack correctly.      |
| Cards   | Grid becomes 1-column on mobile. |
| Forms   | Inputs full width on mobile.     |
| Tables  | Scroll or card layout.           |
| Images  | No stretching/cropping issues.   |
| Buttons | Touch-friendly size.             |
| Footer  | Columns stack cleanly.           |
| Popups  | Usable on mobile.                |

---

# 19. Pixel-Perfect QA Standard

Use visual QA page by page.

## Compare

| Area    | Check                                                     |
| ------- | --------------------------------------------------------- |
| Font    | Family, weight, size, line-height.                        |
| Colors  | Exact hex values.                                         |
| Spacing | Section padding, gaps, margins.                           |
| Layout  | Width, alignment, grid.                                   |
| Images  | Size, crop, quality.                                      |
| Buttons | Radius, hover, padding.                                   |
| Cards   | Shadow, border, spacing.                                  |
| Header  | Height, logo, nav spacing.                                |
| Footer  | Links, columns, typography.                               |
| Mobile  | Same visual intent, not necessarily exact desktop layout. |

## Acceptable Difference

Define upfront:

| Replica Type       | Tolerance                                       |
| ------------------ | ----------------------------------------------- |
| Pixel-perfect      | Very low tolerance, almost exact.               |
| Functional replica | Visual similarity acceptable.                   |
| Modernized replica | Improvements allowed.                           |
| Migration replica  | Preserve UX/content/SEO more than exact pixels. |

---

# 20. Performance Standard

## Mandatory Performance Rules

| Area       | Standard                               |
| ---------- | -------------------------------------- |
| Images     | Compress and lazy-load.                |
| Fonts      | Load only needed weights.              |
| Scripts    | Remove unused third-party scripts.     |
| CSS        | Avoid bloated styles.                  |
| Video      | Use optimized embed/streaming.         |
| Animations | Avoid heavy scroll/JS animations.      |
| API Calls  | Do not over-fetch.                     |
| Caching    | Use CDN/static caching where possible. |

## Common Mistakes

* Uploading 5MB hero images.
* Loading 8 font weights.
* Adding too many tracking scripts.
* Rebuilding a fast static website into a slow no-code app.
* Ignoring mobile performance.

---

# 21. Accessibility Standard

A proper replica should improve accessibility, not copy bad accessibility.

## Must-Have

| Area                | Standard                                 |
| ------------------- | ---------------------------------------- |
| Semantic HTML       | Use correct headings/sections/buttons.   |
| Alt Text            | Meaningful images need alt text.         |
| Color Contrast      | Text must be readable.                   |
| Keyboard Navigation | Menus/forms/modals usable with keyboard. |
| Focus State         | Visible.                                 |
| Form Labels         | Required.                                |
| Error Messages      | Associated with fields.                  |
| Buttons             | Real buttons, not clickable divs.        |

---

# 22. Security Standard

Even if it is “just a website,” security matters.

## Website Security Checklist

| Area         | Standard                          |
| ------------ | --------------------------------- |
| Forms        | Validate and sanitize input.      |
| Spam         | CAPTCHA/honeypot/rate limit.      |
| Admin Access | Strong passwords/2FA if CMS.      |
| API Keys     | Never expose private keys.        |
| Uploads      | Validate file type and size.      |
| HTTPS        | Required.                         |
| Headers      | Security headers where possible.  |
| Dependencies | Avoid abandoned plugins/packages. |
| No-Code Apps | Check privacy rules/API exposure. |

---

# 23. CMS Decision Standard

Ask whether client needs to edit content after launch.

## Static vs CMS

| Requirement                      | Solution                   |
| -------------------------------- | -------------------------- |
| Client rarely changes content    | Static React/Next.js page. |
| Client updates blogs             | CMS needed.                |
| Client updates services/projects | CMS collection needed.     |
| Client updates pricing           | CMS or admin config.       |
| Client updates images/text often | No-code/CMS preferred.     |

## CMS Options

| Platform           | Use Case                      |
| ------------------ | ----------------------------- |
| WordPress Headless | Existing WP blog/content.     |
| Sanity             | Structured content for React. |
| Strapi             | Self-hosted/custom CMS.       |
| Supabase           | Simple CMS/data tables.       |
| Webflow CMS        | Marketing CMS.                |
| WeWeb Collections  | No-code frontend content.     |
| Bubble DB          | Bubble app-native content.    |

---

# 24. No-Code Specific Standards

## WeWeb/Bubble Replica Must Still Have

```text id="2l09ga"
1. Global styles
2. Reusable components
3. Clean naming
4. Proper responsive layout
5. API security
6. Auth rules
7. Database privacy rules
8. Loading states
9. Error states
10. SEO settings
11. Versioning/backup strategy
12. Handoff documentation
```

No-code does not mean no engineering standards.

---

# 25. Naming Standards for No-Code Builds

## Pages

| Good                   | Bad             |
| ---------------------- | --------------- |
| `Home`                 | `Page 1`        |
| `Contact`              | `New Page Copy` |
| `Dashboard - Projects` | `dashboardtest` |

## Workflows/API Calls

| Good                  | Bad              |
| --------------------- | ---------------- |
| `Submit Contact Form` | `Workflow 1`     |
| `Fetch Project List`  | `API Call 3`     |
| `Create Lead in CRM`  | `Button clicked` |

## Components

| Good                          | Bad              |
| ----------------------------- | ---------------- |
| `Header - Public`             | `Group 1`        |
| `Card - Service`              | `Reusable thing` |
| `Modal - Delete Confirmation` | `Popup A`        |

---

# 26. Animation Replication Standard

## Document Animation Behavior

```text id="xj2xuy"
Element:
Animation Type:
Trigger:
Duration:
Delay:
Easing:
Desktop:
Mobile:
Fallback:
```

## Common Animations

| Animation                   | Use Carefully              |
| --------------------------- | -------------------------- |
| Fade in                     | Safe.                      |
| Slide up                    | Safe.                      |
| Parallax                    | Can hurt performance.      |
| Scroll-triggered animations | Test mobile.               |
| Sliders/carousels           | Avoid if not needed.       |
| Hover animations            | Must have mobile fallback. |

---

