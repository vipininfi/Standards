# Scoping, Discovery & Design Extraction

> **Part of:** [Website Replication Standards](<README.md>)

**Covers:** Sections 1–7 — Confirm ownership, define replication type, client intake checklist, discovery audit, page inventory, design extraction from Figma/existing sites, design tokens

---

Below is a **complete website replication standard/SOP** for cases where a client comes with:

1. **Figma design and wants it built in React / Next.js**
2. **Figma design and wants it built in no-code like WeWeb / Bubble**
3. **Existing deployed website and wants it replicated**
4. **Existing WordPress/Webflow/Wix/custom site and wants migration to React / WeWeb / Bubble**
5. **Existing website and wants same UI but better backend/CMS/performance**

This should be your internal rule:

> A website replica is not only about copying the look. It must preserve design, content, responsiveness, SEO, forms, integrations, tracking, performance, accessibility, and business behavior.

---

# 1. First Rule: Confirm Ownership & Permission

Before replicating any website, confirm the client has rights to the design/content/assets.

## Mandatory Questions

| Question                                              | Why                           |
| ----------------------------------------------------- | ----------------------------- |
| Is this your own website?                             | Avoid copying third-party IP. |
| Do you own the Figma/design/assets?                   | Needed for legal usage.       |
| Do you want exact visual replica or improved rebuild? | Defines scope.                |
| Do you want same content or updated content?          | Impacts migration.            |
| Do you want same backend/CMS/forms/integrations?      | Impacts architecture.         |
| Do you want same URLs for SEO?                        | Important for migration.      |

## Anti-Pattern

Do not blindly replicate a competitor’s website. You can use it as **reference/inspiration**, but not copy branding, content, images, unique illustrations, or proprietary UX patterns.

---

# 2. Define Type of Replication

Every project must be classified first.

| Type                      | Meaning                                                           |
| ------------------------- | ----------------------------------------------------------------- |
| **Pixel-Perfect Replica** | Same layout, spacing, colors, typography, responsiveness.         |
| **Functional Replica**    | Same user flows/features, but UI can be improved.                 |
| **Migration Replica**     | Same website rebuilt on a new platform like React, WeWeb, Bubble. |
| **Modernized Replica**    | Keep brand/content but improve UX, speed, SEO, accessibility.     |
| **Backend Replica**       | Same frontend but backend/CMS/API rebuilt.                        |
| **CMS Replica**           | Same design but editable through CMS.                             |

The scope must clearly say which one is expected.

---

# 3. Initial Client Intake Checklist

## Required Inputs From Client

| Item                 |                 Required? | Notes                                            |
| -------------------- | ------------------------: | ------------------------------------------------ |
| Figma file           |          If design exists | Must have view/edit access.                      |
| Existing website URL |   If deployed site exists | Needed for audit.                                |
| Admin access         | If WordPress/Webflow/etc. | Needed for content/forms/plugins.                |
| Hosting access       |       If migration needed | cPanel, Vercel, Netlify, Cloudflare, etc.        |
| Domain/DNS access    |  If launching new version | Needed for go-live.                              |
| Brand assets         |                       Yes | Logo, colors, fonts, images.                     |
| Content copy         |                       Yes | Final text or permission to extract old content. |
| Analytics access     |            Scenario based | GA4, Meta Pixel, Hotjar, etc.                    |
| Form destination     |                       Yes | Email, CRM, webhook, database.                   |
| SEO requirements     |                       Yes | URLs, metadata, redirects.                       |
| CMS requirement      |            Scenario based | Static vs editable content.                      |

---

# 4. Discovery Audit Before Building

Before writing code or building in no-code, audit the source.

## Website Audit Checklist

| Area            | What to Check                                           |
| --------------- | ------------------------------------------------------- |
| Pages           | All public pages and hidden pages.                      |
| Navigation      | Header, footer, dropdowns, mobile menu.                 |
| Forms           | Contact, newsletter, booking, quote, login, lead forms. |
| CTAs            | Buttons, links, modals, popups.                         |
| Dynamic Content | Blog, case studies, products, listings.                 |
| SEO             | Page titles, meta descriptions, canonical URLs, schema. |
| Integrations    | CRM, email, chat widget, analytics, payment, calendar.  |
| Animations      | Scroll effects, sliders, hover effects, transitions.    |
| Responsiveness  | Desktop, tablet, mobile layouts.                        |
| Performance     | Image size, scripts, loading speed.                     |
| Accessibility   | Contrast, keyboard navigation, alt text.                |
| Tracking        | GA4, GTM, Meta Pixel, LinkedIn Pixel.                   |
| Legal Pages     | Privacy, terms, cookie policy.                          |
| Security        | Forms spam protection, SSL, old plugins if WordPress.   |

---

# 5. Page Inventory Standard

Create a page inventory before development.

```text id="3f7ax2"
Page Name:
URL:
Page Type: Static / CMS / Dynamic / Form / Landing Page
Source: Figma / Existing Website / Client Content
Responsive Required: Yes
SEO Required: Yes
CMS Editable: Yes/No
Forms: Yes/No
Integrations: Yes/No
Priority: P0/P1/P2
```

Example:

| Page           | URL               | Type             | CMS Needed | Notes                 |
| -------------- | ----------------- | ---------------- | ---------: | --------------------- |
| Home           | `/`               | Static/Marketing |         No | Main landing page.    |
| About          | `/about`          | Static           |         No | Company content.      |
| Services       | `/services`       | Static/CMS       |      Maybe | Service cards.        |
| Blog           | `/blog`           | CMS              |        Yes | Dynamic posts.        |
| Contact        | `/contact`        | Form             |         No | Sends to CRM/email.   |
| Privacy Policy | `/privacy-policy` | Legal            |      Maybe | SEO noindex optional. |

---

# 6. Design Extraction Standard

## From Figma

Check:

| Area         | Standard                                             |
| ------------ | ---------------------------------------------------- |
| Typography   | Font family, sizes, weights, line heights.           |
| Colors       | Primary, secondary, background, text, border, error. |
| Spacing      | Section padding, grid gap, container width.          |
| Components   | Buttons, cards, inputs, modals, navbars.             |
| Breakpoints  | Desktop, tablet, mobile frames.                      |
| Assets       | Export SVG where possible, WebP/PNG for images.      |
| Icons        | Use SVG/icon library consistently.                   |
| Interactions | Hover, focus, active, disabled states.               |
| Variants     | Button variants, card variants, form variants.       |

## From Deployed Website

Extract carefully:

| Area        | Standard                                                      |
| ----------- | ------------------------------------------------------------- |
| Screenshots | Capture desktop, tablet, mobile.                              |
| Fonts       | Identify font family and fallback.                            |
| Colors      | Extract actual hex values.                                    |
| Spacing     | Measure section padding and layout widths.                    |
| Images      | Get original assets from client/admin when possible.          |
| Icons       | Use licensed/open-source replacement if original unavailable. |
| Content     | Copy only with client permission.                             |
| Animations  | Document expected behavior.                                   |

Do not rely only on visual guesswork. Create a design token sheet.

---

# 7. Design Token Standard

Every replica should have design tokens.

```text id="l7j2z1"
Colors:
- Primary:
- Secondary:
- Text:
- Muted Text:
- Background:
- Border:
- Success:
- Warning:
- Error:

Typography:
- Font Family:
- H1:
- H2:
- H3:
- Body:
- Small:
- Button:

Layout:
- Max Container Width:
- Desktop Padding:
- Tablet Padding:
- Mobile Padding:
- Section Gap:

Components:
- Button Radius:
- Card Radius:
- Input Height:
- Navbar Height:
```

For React, this becomes Tailwind/theme variables.

For WeWeb/Bubble, this becomes global style variables/classes.

---

# 8. Technology Decision Standard

## React / Next.js Use When

| Use React/Next.js When               |
| ------------------------------------ |
| High performance is required.        |
| SEO matters strongly.                |
| Custom UI/animations are needed.     |
| App has complex logic.               |
| Backend/API integration is advanced. |
| Long-term scalability matters.       |
| Client wants full code ownership.    |
| Custom dashboard/SaaS is needed.     |

## WeWeb Use When

| Use WeWeb When                                              |
| ----------------------------------------------------------- |
| Client wants no-code frontend with API/backend flexibility. |
| Backend is Supabase/Xano/custom API.                        |
| App is data-driven but frontend should be editable.         |
| Fast iteration is important.                                |
| Client may manage screens later.                            |

## Bubble Use When

| Use Bubble When                                          |
| -------------------------------------------------------- |
| Client wants full no-code app with frontend + backend.   |
| Internal tool/MVP is required fast.                      |
| Complex custom frontend performance is not the priority. |
| Workflows and database logic can live in Bubble.         |

## WordPress Use When

| Use WordPress When                         |
| ------------------------------------------ |
| Client wants content-heavy marketing site. |
| Blog/editorial workflow is important.      |
| Non-technical users need easy editing.     |
| Existing SEO/blog migration is large.      |

---

