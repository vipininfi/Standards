# Checklist — SendGrid Integration

Run this checklist when setting up SendGrid for the first time or adding a new email type.

**Standard:** [SendGrid Standard](../../standards/third-parties-implementation/sendgrid.md)

---

## 1. Account Setup

[ ] Domain authentication completed (SPF, DKIM, DMARC DNS records added) — not single sender only
[ ] From email is from your authenticated domain — not `@gmail.com` or `@hotmail.com`
[ ] From name set to your app/company name — not a personal name
[ ] Reply-to set to a monitored inbox (`support@yourdomain.com`)

## 2. API Key

[ ] API key created with **Restricted Access** — `Mail Send` permission only (minimum required)
[ ] API key stored in environment variable: `SENDGRID_API_KEY`
[ ] API key never committed to version control
[ ] API key not exposed in frontend code

## 3. Environment Variables

[ ] `SENDGRID_API_KEY` set in all environments (staging, production)
[ ] `SENDGRID_FROM_EMAIL` set
[ ] `SENDGRID_FROM_NAME` set
[ ] Dynamic template IDs stored as env vars (e.g. `SENDGRID_TEMPLATE_INVITE=d-xxx`)

## 4. Template Decision Made

[ ] For each email type, a decision has been made: code-rendered vs SendGrid Dynamic Template
[ ] Code-rendered: used for developer-controlled, security-sensitive, or rarely-changing emails
[ ] Dynamic template: used for marketing content, A/B tests, or non-technical editors
[ ] NOT maintaining both a code template and a dynamic template for the same email

## 5. Code-Rendered Templates (if used)

[ ] Templates stored in `templates/email/` directory
[ ] Base email template created: `base_email.html` and `base_email.txt`
[ ] Both HTML and plaintext versions exist for every email type
[ ] Subject line passed via context — not hardcoded inside the template file
[ ] Templates tested by sending to a test account (not just visual preview)

## 6. SendGrid Dynamic Templates (if used)

[ ] Template created in SendGrid Dashboard and named clearly
[ ] Template tested using "Send Test" in the Dashboard before going live
[ ] Template ID stored in environment variable — not hardcoded in code
[ ] All variables used in the template are documented
[ ] Handlebars syntax used correctly: `{{variable_name}}`

## 7. Sending Implementation

[ ] Django EMAIL_BACKEND configured to use SendGrid SMTP or the sendgrid-python SDK
[ ] `fail_silently=False` for critical emails (password reset, verification)
[ ] `fail_silently=True` or exception caught gracefully for non-critical (welcome, notification)
[ ] EmailMultiAlternatives used for HTML + plaintext (not HTML-only)

## 8. Email Standards

[ ] Subject lines are specific and descriptive — no "Action Required", "URGENT", or spam-trigger words
[ ] Subject max ~50 characters — tested on mobile width
[ ] Unsubscribe link included for any marketing or promotional emails
[ ] Transactional emails do not require unsubscribe but include contact info in footer
[ ] Password reset links: 1-hour expiry, invalidated after single use
[ ] Invite links: 7-day expiry, marked used after acceptance

## 9. Error Handling

[ ] Email failures logged to DB or logging service (not silently dropped)
[ ] Retry logic implemented for transient failures (max 3 attempts, exponential backoff)
[ ] Critical email failures surface an error to the user (password reset fails → show error)
[ ] Non-critical failures logged only (welcome email, notification)

## 10. Bounce and Suppression

[ ] App does NOT retry sends to known-bounced addresses
[ ] SendGrid bounce/spam dashboard monitored
[ ] Bounce rate checked — below 2% target
[ ] Suppression list not bypassed manually

---

**Standard:** [SendGrid Standard](../../standards/third-parties-implementation/sendgrid.md)
