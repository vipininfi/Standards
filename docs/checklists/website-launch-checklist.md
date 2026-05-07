# Website Launch Checklist

Run this before going live on any website build, replica, or migration.

---

## Pre-Launch: Client & Scope

```text
[ ] Client ownership/rights confirmed (for replicas or migrations)
[ ] Replication type defined and agreed (pixel-perfect / functional / migration / modernized)
[ ] Page inventory completed and all required pages built
[ ] Client has reviewed and approved the staging version
[ ] Any out-of-scope items are documented and communicated
```

## Design & Visual

```text
[ ] All pages match approved Figma or source design
[ ] Fonts match (family, weight, size, line-height)
[ ] Colors match (exact hex values)
[ ] Spacing and section padding consistent
[ ] Buttons match (radius, hover state, padding)
[ ] Cards, modals, and components match source
[ ] Images display correctly (no stretching, cropping, or broken images)
[ ] Logo is correct and renders cleanly on all backgrounds
```

## Responsive

```text
[ ] Desktop layout (1280px) tested
[ ] Laptop layout (1024px) tested
[ ] Tablet layout (768px) tested
[ ] Mobile layout (390px / 414px) tested
[ ] Small mobile (360px) tested
[ ] Header/mobile menu works on mobile
[ ] Forms are full-width on mobile
[ ] Card grids stack to single column on mobile
[ ] Buttons are touch-friendly size
[ ] No horizontal scroll on mobile
```

## Pages & Navigation

```text
[ ] All nav links work and point to the correct pages
[ ] All dropdown menus work (desktop and mobile)
[ ] All CTA buttons work
[ ] External links open in new tab
[ ] Internal links work correctly
[ ] 404 page is set up
[ ] Legal pages are present (Privacy Policy, Terms of Service if required)
```

## Forms

```text
[ ] All forms submit correctly
[ ] Form validation works (required fields, email format, phone, etc.)
[ ] Success message shows after submit
[ ] Error message shows if submit fails
[ ] Form fields preserved on error
[ ] Email notification sent to correct address
[ ] CRM / webhook integration working (if applicable)
[ ] Spam protection in place (CAPTCHA / honeypot / rate limit)
[ ] File uploads work and validate type/size (if applicable)
```

## SEO

```text
[ ] Page title added to every page
[ ] Meta description added to every page
[ ] Open Graph title, description, and image set
[ ] Canonical URL set correctly
[ ] H1 tag is present and correct on every page (one H1 per page)
[ ] Image alt text added for meaningful images
[ ] Sitemap generated and accessible at /sitemap.xml
[ ] Robots.txt configured correctly
[ ] No page accidentally set to noindex
[ ] Broken internal links fixed
[ ] Schema markup preserved or added (if required — business, article, product)
```

## Migration: Redirects (if replacing an existing site)

```text
[ ] All old URLs mapped to new URLs
[ ] 301 redirects implemented and tested
[ ] Old blog post URLs redirected
[ ] Old page URLs redirected
[ ] No old important URL returns 404
[ ] Redirect mapping document complete
```

## Analytics & Tracking

```text
[ ] Google Analytics 4 (GA4) installed and verified
[ ] Google Tag Manager installed (if used)
[ ] Meta Pixel installed and verified (if required)
[ ] LinkedIn Insight Tag installed (if required)
[ ] Form submission events tracked
[ ] CTA click events tracked
[ ] Cookie consent implemented (if required by region)
[ ] No tracking firing on localhost/staging (or excluded)
```

## Performance

```text
[ ] All images compressed (WebP preferred)
[ ] Large hero images optimized (ideally under 300KB)
[ ] Fonts load efficiently (only needed weights)
[ ] Unused third-party scripts removed
[ ] Lazy loading added for below-the-fold content
[ ] Mobile page speed acceptable
[ ] No large layout shifts on load
```

## Security

```text
[ ] HTTPS is active (SSL certificate valid)
[ ] No private API keys exposed in frontend code
[ ] Form inputs are validated and sanitized
[ ] Admin access protected with strong password (and 2FA if CMS)
[ ] WordPress: no outdated plugins or themes (if WordPress)
[ ] No-code apps: database privacy rules confirmed
```

## Accessibility

```text
[ ] Semantic HTML used (correct headings, buttons, not clickable divs)
[ ] Alt text on all meaningful images
[ ] Text color contrast meets readability standards
[ ] Form fields have labels
[ ] Keyboard navigation works for menus and forms
[ ] Focus state is visible on interactive elements
```

## Go-Live

```text
[ ] Staging client approval received in writing
[ ] DNS/domain access confirmed and ready
[ ] Hosting is set up and configured
[ ] Old site backup taken
[ ] Rollback plan prepared
[ ] SSL certificate verified after DNS switch
[ ] All redirects re-tested on live domain
[ ] Forms tested on live domain
[ ] Analytics firing on live domain
[ ] Site monitored for 24 hours after launch
```

## Handoff

```text
[ ] Handoff document delivered to client
[ ] Client trained on how to edit/manage content (if CMS or no-code)
[ ] All credentials handed over securely
[ ] Any known limitations documented
```

---

## Practice Task

Apply what you learned by replicating a real multi-section landing page, pixel-perfect and fully responsive.

**→ [Task 01: Replicate a Landing Page — Hero + Features + CTA](../../tasks/website/01-landing-page-section.md)**

Covers: design token extraction from Figma, Hero/Features/CTA as reusable components, one FeatureCard mapped over data, responsive layout at 3 breakpoints, semantic HTML, CTA form with loading/error/success states, image optimization, Lighthouse performance check.
