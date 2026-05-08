# Checklist — Gmail Integration

Run this checklist when using Gmail SMTP for sending email (dev/internal/low-volume tools).

**Standard:** [Gmail Standard](../../standards/third-parties-implementation/gmail.md)

---

## 1. Decision — Is Gmail Appropriate?

[ ] Confirmed this is NOT a production app sending to real users at scale
[ ] Confirmed daily volume is well within Gmail limits (< 500/day personal, < 2000/day Workspace)
[ ] Confirmed this is for: development, internal tool, admin notifications, or staging only
[ ] If any of the above are not true — switch to SendGrid before proceeding

## 2. Gmail Account

[ ] Using a dedicated Gmail/Workspace account for the app — not a personal email
[ ] 2-Step Verification is enabled on the account (required for App Passwords)

## 3. Authentication

[ ] App Password generated (if using SMTP): 16-character password from Google Security settings
[ ] App Password stored in environment variable: `GMAIL_APP_PASSWORD`
[ ] App Password NOT committed to version control
[ ] Main account password NOT used as EMAIL_HOST_PASSWORD anywhere in code

## 4. Django Configuration

[ ] `EMAIL_BACKEND` set correctly per environment:
  - Local: `console.EmailBackend` or `filebased.EmailBackend` (no real sending)
  - Staging: MailHog/MailPit (catches emails without sending)
  - Internal production: `smtp.EmailBackend` with Gmail SMTP credentials
[ ] `EMAIL_HOST = 'smtp.gmail.com'`
[ ] `EMAIL_PORT = 587`
[ ] `EMAIL_USE_TLS = True`
[ ] `EMAIL_HOST_USER` set from environment variable
[ ] `EMAIL_HOST_PASSWORD` set from environment variable (App Password, not main password)

## 5. Environment Variables

[ ] `GMAIL_USER` set in environment
[ ] `GMAIL_APP_PASSWORD` set in environment
[ ] No credentials in settings files, config files, or source code

## 6. Limits and Monitoring

[ ] Current daily email volume tracked (stay under limit)
[ ] If volume grows close to limit: SendGrid migration planned

## 7. When to Switch

[ ] Team knows the trigger to switch to SendGrid:
  - Approaching 500/day limit
  - Sending to external/real users
  - Need email analytics or deliverability tracking
  - Emails landing in spam

---

**Standard:** [Gmail Standard](../../standards/third-parties-implementation/gmail.md)
