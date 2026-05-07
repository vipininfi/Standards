# Website Replication Standards

> **Core Rule:** A website replica is not only about copying the look. It must preserve design, content, responsiveness, SEO, forms, integrations, tracking, performance, accessibility, and business behavior.

---

## Contents

| File | What It Covers |
|------|---------------|
| [01 — Scoping & Design](01-scoping-and-design.md) | Ownership check, replication type, client intake checklist, discovery audit, page inventory, design extraction from Figma/existing sites, design tokens |
| [02 — Building by Platform](02-building-by-platform.md) | Technology decision table (React vs WeWeb vs Bubble vs WordPress), Figma-to-React, Figma-to-WeWeb, Figma-to-Bubble build standards |
| [03 — Migration](03-migration.md) | Existing website to React/WeWeb/Bubble, WordPress migration, content migration, SEO preservation, forms replication, integration replication |
| [04 — Responsive, QA & Performance](04-responsive-qa-performance.md) | Responsive standards, pixel-perfect QA, performance, accessibility, security, CMS decisions, no-code naming standards, animation replication |
| [05 — Launch & Handoff](05-launch-and-handoff.md) | Header/footer, tracking/analytics, redirects, launch checklist, documentation, QA checklists, client review, risks, definition of done |

---

## Replication Types — Define This First

Before any build starts, the replication type must be agreed and documented.

| Type | Meaning |
|------|---------|
| **Pixel-Perfect Replica** | Same layout, spacing, colors, typography, responsiveness — very low tolerance. |
| **Functional Replica** | Same user flows and features, UI can be improved. |
| **Migration Replica** | Same website rebuilt on a new platform (React, WeWeb, Bubble). |
| **Modernized Replica** | Keep brand/content but improve UX, speed, SEO, accessibility. |
| **Backend Replica** | Same frontend, backend/CMS/API rebuilt. |
| **CMS Replica** | Same design but editable through a CMS. |

---

## Definition of Done

A website replica is complete only when:

```text
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

## Best Rule

For every replica project, confirm which of these are being replicated:

```text
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

If only the design is copied but forms, SEO, integrations, and responsive behavior are broken — the project is not complete.
