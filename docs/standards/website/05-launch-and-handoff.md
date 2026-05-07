# Launch, Handoff & Definition of Done

> **Part of:** [Website Replication Standards](<README.md>)

**Covers:** Sections 27–36 — Header/footer, tracking/analytics, redirects/launch, required documentation, QA checklists, client review, common risks, platform risks. Sections 37–39 — Definition of done, best rule, client message template

---

# 27. Header/Footer Replication Standard

## Header Must Document

| Item            | Standard                        |
| --------------- | ------------------------------- |
| Logo            | Desktop/mobile size.            |
| Nav Links       | URLs and active states.         |
| Dropdowns       | Items and hover/click behavior. |
| CTA Button      | Text, link, style.              |
| Sticky Behavior | Yes/no.                         |
| Mobile Menu     | Drawer/fullscreen/dropdown.     |

## Footer Must Document

| Item         | Standard                             |
| ------------ | ------------------------------------ |
| Logo         | Correct asset.                       |
| Link Groups  | Company, services, resources, legal. |
| Contact Info | Email, phone, address.               |
| Social Links | Correct URLs.                        |
| Newsletter   | If present, integration.             |
| Copyright    | Dynamic year preferred.              |

---

# 28. Tracking & Analytics Standard

Before launch, confirm:

```text id="wolnlo"
GA4 installed
GTM installed if needed
Meta Pixel installed if needed
LinkedIn Pixel installed if needed
Form submit events tracked
CTA clicks tracked
Booking clicks tracked
Purchase/payment events tracked if ecommerce
Cookie consent respected if required
```

## Event Naming Example

```text id="24l3ns"
contact_form_submit
newsletter_signup
book_demo_click
pricing_cta_click
lead_form_submit
download_brochure
```

---

# 29. Redirection & Launch Standard

If replacing existing website, redirects are mandatory.

## Redirect Mapping

```text id="m7cnzr"
Old URL:
New URL:
Redirect Type: 301
Priority:
Tested: Yes/No
```

Example:

| Old URL                | New URL     | Type |
| ---------------------- | ----------- | ---- |
| `/about-us`            | `/about`    | 301  |
| `/services/web-design` | `/services` | 301  |
| `/contact-us`          | `/contact`  | 301  |

## Launch Checklist

| Step              | Standard                 |
| ----------------- | ------------------------ |
| Staging Approved  | Client signs off.        |
| DNS Access Ready  | Domain access confirmed. |
| SSL Ready         | HTTPS active.            |
| Redirects Ready   | Old URLs mapped.         |
| Forms Tested      | Emails/CRM working.      |
| SEO Checked       | Metadata/sitemap/robots. |
| Analytics Checked | Events firing.           |
| Backup Taken      | Old site backup.         |
| Rollback Plan     | Prepared.                |

---

# 30. Documentation Required for Replica Project

## Required Documents

| Document            | Purpose                             |
| ------------------- | ----------------------------------- |
| Scope Document      | Defines exact replica expectations. |
| Page Inventory      | All pages and URLs.                 |
| Design Token Sheet  | Colors, fonts, spacing.             |
| Component List      | Reusable elements.                  |
| Form Documentation  | Fields, validations, destination.   |
| Integration Map     | CRM, analytics, email, payments.    |
| SEO Migration Sheet | Metadata and redirects.             |
| QA Checklist        | Desktop/tablet/mobile comparison.   |
| Handoff Document    | How client/frontend/admin uses it.  |

---

# 31. Replica Scope Document Template

```md id="u5xx6t"
# Website Replication Scope

## Project Type
Figma to React / Figma to WeWeb / Existing Website to React / WordPress to WeWeb / Other

## Replication Type
Pixel-perfect / Functional / Migration / Modernized

## Source
Figma URL:
Existing Website URL:
Admin Access:
Assets Folder:

## Pages Included
1. Home
2. About
3. Services
4. Contact
5. Blog
6. Privacy Policy

## Pages Excluded
- Example: Blog migration not included
- Example: Payment flow not included

## Design Requirements
- Match colors
- Match typography
- Match spacing
- Match responsive behavior
- Match animations where practical

## Functional Requirements
- Contact form
- Newsletter form
- Booking link
- CRM integration
- Analytics tracking

## SEO Requirements
- Preserve URLs
- Add redirects
- Add metadata
- Generate sitemap

## Platform
Frontend:
Backend:
CMS:
Hosting:

## Acceptance Criteria
- All pages match approved design/source.
- All forms work.
- Mobile responsive tested.
- SEO metadata added.
- Redirects tested.
- Client approved staging before launch.
```

---

# 32. API/Form Handoff Template

For forms/API-connected sections:

```md id="ovxs3j"
# Form/API: Contact Form

## Page
/contact

## Purpose
Collects website visitor inquiries and sends them to CRM/email.

## Fields

| Field | Type | Required | Validation |
|---|---|---:|---|
| name | string | Yes | 2–100 chars |
| email | email | Yes | Valid email |
| phone | phone | No | Country code + valid number |
| message | text | Yes | 10–1000 chars |

## Submit Destination
- CRM: Zoho Leads
- Email: sales@example.com
- Database: leads table

## Success Message
"Thank you. Our team will contact you shortly."

## Error Message
"We could not submit your request. Please try again."

## Spam Protection
Honeypot / CAPTCHA / rate limit

## Frontend Behavior
- Disable submit while sending.
- Show loading state.
- Preserve fields on error.
- Clear fields on success.
```

---

# 33. QA Checklist

## Visual QA

```text id="u64g3v"
[ ] Header matches source
[ ] Footer matches source
[ ] Fonts match
[ ] Colors match
[ ] Buttons match
[ ] Cards match
[ ] Images match
[ ] Spacing matches
[ ] Desktop layout checked
[ ] Tablet layout checked
[ ] Mobile layout checked
```

## Functional QA

```text id="uqgntn"
[ ] All nav links work
[ ] All CTAs work
[ ] Contact form works
[ ] Newsletter form works
[ ] Booking links work
[ ] External links open correctly
[ ] Social links work
[ ] Search works if present
[ ] CMS content loads if present
[ ] API errors are handled
```

## SEO QA

```text id="9kmojo"
[ ] Title tags added
[ ] Meta descriptions added
[ ] Open Graph images added
[ ] Canonical URLs added
[ ] Sitemap generated
[ ] Robots.txt configured
[ ] Redirects tested
[ ] No accidental noindex
[ ] Broken links checked
```

## Performance QA

```text id="5yve2w"
[ ] Images compressed
[ ] Fonts optimized
[ ] Unused scripts removed
[ ] Lazy loading added
[ ] Mobile speed checked
[ ] Large layout shifts fixed
```

---

# 34. Client Review Standard

Do not ask client to “check everything” vaguely.

Give structured review:

```text id="klsz32"
Please review these areas:

1. Visual match against old website/Figma
2. Text/content accuracy
3. Images/logos
4. Mobile responsiveness
5. Form submissions
6. Links and buttons
7. SEO-critical URLs
8. Any missing section/page
```

---

# 35. Common Risks & How to Handle

| Risk                                                    | Prevention                                            |
| ------------------------------------------------------- | ----------------------------------------------------- |
| Client expects exact copy but did not say pixel-perfect | Define replication type upfront.                      |
| Existing website has hidden pages                       | Crawl and create page inventory.                      |
| Missing original assets                                 | Ask for source assets; avoid low-quality screenshots. |
| Fonts are paid/licensed                                 | Confirm license or use fallback.                      |
| SEO loss after migration                                | Preserve URLs and metadata, add redirects.            |
| No-code platform cannot match exact layout              | Explain limitations before build.                     |
| Forms stop working                                      | Document and test every form.                         |
| Tracking lost                                           | Reinstall analytics/pixels.                           |
| CMS not planned                                         | Ask before build.                                     |
| Mobile layout differs                                   | Define responsive standard early.                     |

---

# 36. Platform-Specific Risk

## React Risk

| Risk                            | Fix                                        |
| ------------------------------- | ------------------------------------------ |
| Over-engineering simple website | Use static generation/simple architecture. |
| Poor SEO setup                  | Use Next.js metadata and sitemap.          |
| Component duplication           | Build reusable components.                 |
| Hardcoded content everywhere    | Use CMS if client needs editing.           |

## WeWeb Risk

| Risk                    | Fix                                    |
| ----------------------- | -------------------------------------- |
| Poor responsive control | Test each breakpoint.                  |
| Messy structure         | Use reusable components/global styles. |
| API keys exposed        | Use backend/proxy for secrets.         |
| No loading/error states | Add states for every API call.         |

## Bubble Risk

| Risk            | Fix                                            |
| --------------- | ---------------------------------------------- |
| Slow pages      | Reduce workflows/plugins, optimize DB queries. |
| Privacy leaks   | Configure privacy rules.                       |
| Messy workflows | Name and organize workflows.                   |
| Weak SEO        | Use Bubble SEO settings carefully.             |

## WordPress Migration Risk

| Risk                        | Fix                                  |
| --------------------------- | ------------------------------------ |
| Losing blog SEO             | Export/import posts with slugs/meta. |
| Plugin functionality missed | Audit plugins first.                 |
| Broken media                | Migrate media library properly.      |
| URL changes                 | Add 301 redirects.                   |

---

# 37. Definition of Done

A website replica is complete only when:

```text id="sje3jm"
[ ] Client ownership/permission confirmed
[ ] Source website/Figma reviewed
[ ] Page inventory completed
[ ] Design tokens documented
[ ] Components documented
[ ] All required pages built
[ ] Desktop responsive tested
[ ] Tablet responsive tested
[ ] Mobile responsive tested
[ ] Forms validated and tested
[ ] Emails/CRM/webhooks tested
[ ] SEO metadata added
[ ] Redirects added if migration
[ ] Analytics/tracking installed
[ ] Images optimized
[ ] Accessibility basics checked
[ ] Performance checked
[ ] Staging link approved
[ ] Production deployed
[ ] SSL active
[ ] Handoff document delivered
[ ] Client knows how to edit/manage content
```

---

# 38. Best Internal Rule

For every replica project, ask:

```text id="9hylwf"
Are we replicating:
1. Visual design?
2. Content?
3. User flows?
4. CMS/admin editing?
5. SEO structure?
6. Integrations?
7. Tracking?
8. Performance?
9. Mobile behavior?
10. Backend/database behavior?
```

If only the design is copied but forms, SEO, integrations, and responsive behavior are broken, the project is not complete.

---

# 39. Practical Client Message You Can Use

Before starting the website replication, we will first audit the existing website/Figma and create a clear page-by-page scope. This includes all pages, sections, forms, CTAs, responsive behavior, SEO metadata, integrations, analytics, and any CMS requirements.

Our goal is not just to copy the visual design, but to rebuild the website properly so it works the same or better across desktop, tablet, and mobile. We will also check forms, tracking, redirects, performance, and launch requirements to make sure nothing breaks during migration.

Once the audit is complete, we will confirm whether you need a pixel-perfect replica, a functional replica, or a modernized version of the current website.
