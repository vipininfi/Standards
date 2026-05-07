# Building by Platform — React, WeWeb & Bubble

> **Part of:** [Website Replication Standards](<README.md>)

**Covers:** Sections 8–11 — Technology decision table, Figma-to-React standard, Figma-to-WeWeb standard, Figma-to-Bubble standard

---

# 9. Figma to React Standard

## Development Flow

```text id="5m1y0v"
1. Review Figma screens.
2. Confirm desktop/tablet/mobile frames.
3. Extract design tokens.
4. Define component library.
5. Set project structure.
6. Build layout shell.
7. Build reusable components.
8. Build pages.
9. Add responsive behavior.
10. Add forms and validation.
11. Add integrations.
12. Add SEO metadata.
13. Add animations carefully.
14. QA against Figma.
15. Deploy staging.
16. Client review.
17. Final production deploy.
```

## React/Next.js Standards

| Area          | Standard                                         |
| ------------- | ------------------------------------------------ |
| Framework     | Next.js preferred for SEO/marketing sites.       |
| Styling       | Tailwind or well-structured CSS modules.         |
| Components    | Reusable component-first architecture.           |
| Forms         | React Hook Form + validation schema recommended. |
| SEO           | Metadata per page.                               |
| Images        | Optimized image component/CDN.                   |
| Routing       | Clean URLs matching old site if migration.       |
| Accessibility | Semantic HTML, labels, keyboard support.         |
| Performance   | Lazy load heavy sections/assets.                 |
| Deployment    | Vercel/Netlify/custom server depending on app.   |

## Component Standards

Create reusable components:

```text id="6nczp2"
Button
Input
Textarea
Select
Checkbox
Modal
Navbar
Footer
HeroSection
ServiceCard
TestimonialCard
PricingCard
FAQAccordion
BlogCard
ContactForm
CTASection
Container
SectionWrapper
```

Do not duplicate the same button/card markup on every page.

---

# 10. Figma to WeWeb Standard

## Build Flow

```text id="tz6ptw"
1. Create global design system.
2. Configure colors, typography, spacing.
3. Create reusable sections/components.
4. Build page structure.
5. Connect backend/API if needed.
6. Configure collections/data bindings.
7. Add workflows/actions.
8. Add responsive behavior.
9. Add SEO metadata.
10. Test published staging link.
```

## WeWeb Standards

| Area                | Standard                                     |
| ------------------- | -------------------------------------------- |
| Global Styles       | Define before building pages.                |
| Reusable Components | Header, footer, cards, forms.                |
| Collections         | Use for dynamic data.                        |
| API Calls           | Centralize and name clearly.                 |
| Authentication      | Use proper auth plugin/backend auth.         |
| Variables           | Use clean naming.                            |
| Workflows           | Name workflow by action purpose.             |
| Responsiveness      | Test every breakpoint manually.              |
| SEO                 | Configure page title, description, OG image. |
| Forms               | Validate before API submission.              |

## WeWeb Anti-Patterns

Avoid:

* Hardcoding repeated content everywhere.
* No global styles.
* Using random spacing values on every section.
* Directly exposing private API keys.
* Building API calls without error states.
* No loading state for API-driven sections.
* Using one giant page with unorganized layers.

---

# 11. Figma to Bubble Standard

## Bubble Build Flow

```text id="k1iv94"
1. Define app data structure.
2. Create reusable styles.
3. Build reusable elements.
4. Build pages.
5. Add workflows.
6. Add database privacy rules.
7. Add responsive rules.
8. Add SEO/meta if public pages.
9. Test user roles and workflows.
```

## Bubble Standards

| Area              | Standard                                            |
| ----------------- | --------------------------------------------------- |
| Styles            | Use Bubble style system, not random inline styling. |
| Reusable Elements | Header, footer, sidebar, cards.                     |
| Database          | Clean data types and relationships.                 |
| Privacy Rules     | Mandatory for user-owned/private data.              |
| Workflows         | Named and organized.                                |
| Conditions        | Avoid too many messy conditions.                    |
| Responsive        | Use new responsive engine properly.                 |
| Plugins           | Use only necessary trusted plugins.                 |
| SEO               | Configure page metadata where needed.               |

## Bubble Anti-Patterns

Avoid:

* No privacy rules.
* Too many plugins.
* Massive workflows with no naming.
* Repeating same UI manually everywhere.
* Exposing sensitive data in page source.
* No database cleanup strategy.
* No role-based access control.

---

