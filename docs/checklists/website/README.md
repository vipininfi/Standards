# Website Checklists

> Use these before marking any website build or migration task done.

---

| Checklist | Use When |
|-----------|----------|
| [Website Launch Checklist](../website-launch-checklist.md) | Full pre-launch gate — covers all platforms (React, WeWeb, Bubble, WordPress) |
| [WeWeb Page Build](weweb-page.md) | Building a WeWeb page — global layout, data binding (all 4 states), forms, Xano calls, reusable components, mobile |
| [React Page Build](react-page.md) | Building a React page — component structure, data fetching hooks, forms (shared Add/Edit), TypeScript, accessibility, mobile |

---

## Quick Reference — Core Website Build Rules

```text
[ ] Confirm client ownership/rights before replicating any design or content
[ ] Audit all pages BEFORE writing any code — complete page inventory first
[ ] Define the replication type before starting: Pixel-Perfect / Functional / Migration / Modernized
[ ] Design tokens documented before building components (colors, fonts, spacing)
[ ] All pages responsive: desktop, tablet (768px), mobile (375px)
[ ] Forms tested end-to-end: submission, validation, confirmation, CRM/email integration
[ ] SEO metadata on every page (title, description, canonical URL)
[ ] 301 redirects for all changed URLs if this is a migration
[ ] Analytics tracking verified (events fire correctly)
[ ] Images optimized (WebP/AVIF where possible, correct dimensions, alt text)
[ ] Performance checked (Lighthouse score / Core Web Vitals)
[ ] Accessibility basics: alt text, keyboard navigation, sufficient color contrast
[ ] Client approved staging before going live
[ ] SSL active on production domain
[ ] Handoff document delivered to client
```

---

## Replication Type — Agree Before Starting

| Type | Meaning |
|------|---------|
| Pixel-Perfect Replica | Same layout, spacing, colors, typography — very low tolerance |
| Functional Replica | Same user flows, UI can be improved |
| Migration Replica | Same website rebuilt on a new platform |
| Modernized Replica | Keep brand/content but improve UX, speed, SEO, accessibility |
| Backend Replica | Same frontend, backend/CMS rebuilt |
| CMS Replica | Same design but editable through a CMS |

---

## Related Standards

- [Website Replication Standards](../../standards/website/README.md) — Full reference
  - [01 — Scoping & Design](../../standards/website/01-scoping-and-design.md)
  - [02 — Building by Platform](../../standards/website/02-building-by-platform.md)
  - [03 — Migration](../../standards/website/03-migration.md)
  - [04 — Responsive, QA & Performance](../../standards/website/04-responsive-qa-performance.md)
  - [05 — Launch & Handoff](../../standards/website/05-launch-and-handoff.md)
