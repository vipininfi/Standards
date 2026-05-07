# Migration from Existing Sites

> **Part of:** [Website Replication Standards](<README.md>)

**Covers:** Sections 12–17 — Existing website to React/WeWeb/Bubble, WordPress migration, content migration, SEO preservation, forms replication, integration replication

---

# 12. Existing Website to React / WeWeb / Bubble Standard

When client gives a deployed website, first create a reverse-engineering document.

## Reverse Engineering Checklist

| Area          | Questions                             |
| ------------- | ------------------------------------- |
| Pages         | What pages exist?                     |
| URLs          | Should old URLs remain same?          |
| CMS           | Is blog/content dynamic?              |
| Forms         | Where do forms submit?                |
| Tracking      | What analytics/pixels are installed?  |
| SEO           | What metadata/schema exists?          |
| Assets        | Where are original images/logos?      |
| Integrations  | CRM, email, chat, booking, payment?   |
| Animations    | Are they required?                    |
| Admin Editing | Does client need CMS/no-code editing? |
| Performance   | What should be improved?              |

## Migration Flow

```text id="byuvmu"
1. Crawl/list all existing pages.
2. Identify static vs dynamic pages.
3. Extract content and assets.
4. Document existing SEO metadata.
5. Document current forms and integrations.
6. Decide new platform architecture.
7. Rebuild UI page by page.
8. Reconnect forms/integrations.
9. Add redirects if URL changes.
10. QA old vs new.
11. Deploy staging.
12. Client approval.
13. Switch DNS/domain.
14. Monitor after launch.
```

---

# 13. WordPress to React / WeWeb Migration Standard

## WordPress Audit

| Item              | Check                                        |
| ----------------- | -------------------------------------------- |
| Theme             | Custom theme or purchased theme?             |
| Plugins           | Forms, SEO, cache, security, builder.        |
| Pages             | Static pages.                                |
| Posts             | Blog posts/categories/tags.                  |
| Media Library     | Images/PDFs/videos.                          |
| Forms             | Contact Form 7, WPForms, Gravity Forms, etc. |
| SEO Plugin        | Yoast, RankMath, All in One SEO.             |
| Custom Post Types | Services, projects, testimonials, products.  |
| Users             | Admin/editor accounts.                       |
| Redirects         | Existing SEO redirects.                      |

## Migration Decision

| Existing WordPress Feature | New Platform Equivalent                                    |
| -------------------------- | ---------------------------------------------------------- |
| Blog Posts                 | Headless CMS / Supabase / WordPress headless / Webflow CMS |
| Media Library              | S3/R2/Supabase Storage                                     |
| Contact Forms              | API/Edge Function/CRM webhook                              |
| SEO Plugin Metadata        | Next.js metadata / WeWeb SEO config                        |
| Custom Post Types          | CMS collections/database tables                            |
| Plugins                    | Custom APIs/integrations                                   |
| Shortcodes                 | Rebuilt components                                         |

---

# 14. Content Migration Standard

## Content Must Be Cleaned

| Content Area | Standard                                 |
| ------------ | ---------------------------------------- |
| Headings     | Proper H1/H2/H3 hierarchy.               |
| Paragraphs   | No broken copied formatting.             |
| Images       | Optimized and renamed.                   |
| Alt Text     | Added for meaningful images.             |
| Links        | Internal links updated.                  |
| Buttons      | Clear CTA text.                          |
| Blog Posts   | Preserve author/date/category if needed. |
| Legal Pages  | Confirm latest version with client.      |

## Image Standard

| Rule         | Standard                                      |
| ------------ | --------------------------------------------- |
| Format       | WebP preferred, SVG for logos/icons.          |
| Size         | Compress before upload.                       |
| Naming       | `service-name-hero.webp`, not `IMG_1234.png`. |
| Alt Text     | Required for meaningful images.               |
| Lazy Loading | Required below fold.                          |

---

# 15. SEO Preservation Standard

This is critical when replacing an existing site.

## Mandatory SEO Checklist

| Item              | Standard                                            |
| ----------------- | --------------------------------------------------- |
| Same URLs         | Keep old URLs where possible.                       |
| Redirects         | 301 redirect old URLs to new URLs.                  |
| Page Titles       | Preserve or improve.                                |
| Meta Descriptions | Preserve or improve.                                |
| H1 Tags           | One primary H1 per page.                            |
| Canonical URLs    | Set correctly.                                      |
| Open Graph        | Title, description, image.                          |
| Sitemap           | Generate and submit.                                |
| Robots.txt        | Configure properly.                                 |
| Schema Markup     | Preserve business/article/product schema if needed. |
| Image Alt Text    | Add/retain.                                         |
| Internal Links    | Update broken links.                                |
| 404 Check         | No broken important pages.                          |

## SEO Migration Anti-Patterns

Avoid:

* Changing URLs without redirects.
* Forgetting blog posts.
* Losing metadata.
* Blocking site with `noindex`.
* Removing schema markup.
* Replacing text content with images.
* Launching without sitemap.

---

# 16. Forms Replication Standard

Forms are often where replicas fail.

## For Every Form, Document

```text id="ej377u"
Form Name:
Page:
Fields:
Required Fields:
Validation Rules:
Submit Destination:
Email Notification:
CRM Integration:
Success Message:
Error Message:
Spam Protection:
File Upload:
Frontend Behavior:
Backend/API Used:
```

## Field Validation Standard

| Field    | Validation                            |
| -------- | ------------------------------------- |
| Name     | Required, 2–100 chars.                |
| Email    | Required, valid email.                |
| Phone    | Country code + valid number length.   |
| Message  | Required if contact form, max length. |
| Company  | Optional/required depending on B2B.   |
| File     | Type + size validation.               |
| Dropdown | Must match allowed options.           |

## Form UX Standards

* Disable submit while sending.
* Show loading state.
* Preserve user input on error.
* Show field-level errors.
* Show success confirmation.
* Send backend email/CRM notification.
* Add spam protection.
* Log failed submissions.
* Never silently fail.

---

# 17. Integration Replication Standard

Existing site may have hidden integrations.

## Common Integrations to Check

| Integration          | What to Verify                     |
| -------------------- | ---------------------------------- |
| Google Analytics     | Measurement ID.                    |
| Google Tag Manager   | Container ID.                      |
| Meta Pixel           | Pixel ID/events.                   |
| LinkedIn Insight Tag | B2B tracking.                      |
| HubSpot              | Forms/chat/CRM.                    |
| Zoho CRM             | Lead capture.                      |
| Mailchimp            | Newsletter.                        |
| SendGrid             | Transactional email.               |
| Calendly             | Booking.                           |
| Stripe/Razorpay      | Payments.                          |
| Chat Widget          | Intercom, Tawk, Crisp, custom bot. |
| Maps                 | Google Maps/Mapbox.                |

## Integration Documentation

```text id="t5n19p"
Integration Name:
Purpose:
Current Website Behavior:
New Platform Implementation:
Required API Keys:
Frontend Changes:
Backend Changes:
Test Case:
Failure Handling:
Owner:
```

---

