# Gmail Integration Standard

Covers when to use Gmail for sending email, OAuth2 setup, App Passwords for SMTP, Django configuration, rate limits, and when you should switch to SendGrid instead.

---

## 1. When to Use Gmail

Gmail is appropriate in specific, limited scenarios:

| Scenario | Use Gmail? |
|----------|-----------|
| Local development and testing | Yes — simple SMTP setup |
| Internal tools with < 100 emails/day | Yes — within personal Gmail limits |
| Admin notification emails to your team | Yes — internal only |
| Staging environment (low volume) | Yes — acceptable |
| Production app with real users | **No** — use SendGrid |
| Marketing/bulk email | **No** — violates Gmail ToS, poor deliverability |
| Transactional email at scale (> 500/day) | **No** — daily limits apply |

---

## 2. Gmail Limits

| Account Type | Daily Limit |
|-------------|------------|
| Free personal Gmail | 500 emails/day |
| Google Workspace (paid) | 2,000 emails/day |

Exceeding limits results in your account being temporarily blocked from sending. This is why Gmail is not suitable for production applications with real user activity.

---

## 3. Authentication Options

### Option A: App Passwords (Recommended for SMTP with 2FA)

Google disabled basic authentication (username + password) in 2022. If you have 2-Step Verification enabled (required for Workspace accounts), use App Passwords:

1. Enable 2-Step Verification on the Gmail account
2. Go to: Google Account → Security → 2-Step Verification → App passwords
3. Create an App Password: select "Mail" and your device type
4. Google generates a 16-character password — use this as your SMTP password

```
GMAIL_USER=your@gmail.com
GMAIL_APP_PASSWORD=xxxx xxxx xxxx xxxx   # 16-char app password (spaces are fine)
```

### Option B: OAuth2 (Recommended for Production Internal Tools)

For production use, OAuth2 is more secure than App Passwords because:
- No stored password
- Can be revoked per-application from Google's security console
- Tokens expire and auto-refresh

OAuth2 setup requires:
1. A Google Cloud project with Gmail API enabled
2. OAuth2 credentials (client_id + client_secret)
3. A refresh token obtained via the OAuth2 consent flow

```
GMAIL_OAUTH_CLIENT_ID=xxx.apps.googleusercontent.com
GMAIL_OAUTH_CLIENT_SECRET=xxx
GMAIL_OAUTH_REFRESH_TOKEN=xxx
GMAIL_USER=your@gmail.com
```

---

## 4. Django Configuration

### Using App Passwords (SMTP)

```python
# settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = env('GMAIL_USER')
EMAIL_HOST_PASSWORD = env('GMAIL_APP_PASSWORD')
DEFAULT_FROM_EMAIL = env('GMAIL_USER')
```

### Local Development — Console Backend (No Emails Sent)

```python
# settings/local.py
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

This prints emails to your terminal instead of sending them. Use this in development so you don't send real emails when testing.

### Local Development — File Backend

```python
# settings/local.py
EMAIL_BACKEND = 'django.core.mail.backends.filebased.EmailBackend'
EMAIL_FILE_PATH = '/tmp/app-emails'  # emails written as files
```

Useful when you want to view the full email HTML in a browser during development.

### Staging — MailHog (Recommended)

Run MailHog or MailPit locally/in staging — it catches all outgoing emails and lets you view them in a web UI without actually sending them:

```python
# settings/staging.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'localhost'   # or 'mailhog' in Docker
EMAIL_PORT = 1025
EMAIL_USE_TLS = False
```

---

## 5. Sending Email (Django)

```python
from django.core.mail import send_mail, EmailMultiAlternatives
from django.conf import settings

# Simple plaintext email
send_mail(
    subject='Your invite link',
    message='Click here to join: https://...',
    from_email=settings.DEFAULT_FROM_EMAIL,
    recipient_list=['user@example.com'],
    fail_silently=False,   # Let exceptions propagate for critical emails
)

# HTML + plaintext (preferred for most emails)
msg = EmailMultiAlternatives(
    subject='Your invite link',
    body='Click here to join: https://...',   # plaintext version
    from_email=settings.DEFAULT_FROM_EMAIL,
    to=['user@example.com'],
)
msg.attach_alternative('<h1>Click here</h1>', 'text/html')
msg.send()
```

---

## 6. When to Switch from Gmail to SendGrid

Switch when any of the following are true:

| Trigger | Switch To |
|---------|----------|
| You're getting close to 500 emails/day | SendGrid |
| You need to send to users who are not in your team | SendGrid |
| You need email open/click analytics | SendGrid |
| Your emails are landing in spam | SendGrid with domain auth |
| You need unsubscribe management | SendGrid |
| You need marketing/campaign emails | SendGrid |
| Customer-facing transactional email in production | SendGrid |

Gmail is a development convenience, not a production email service.

---

## 7. Security Rules for Gmail Integration

- Never commit Gmail credentials to version control
- Use App Passwords or OAuth2 — never use your main account password
- Use a dedicated Gmail account for the application (not your personal email)
- If using a Google Workspace account: enable 2-Step Verification — required for App Passwords
- Rotate App Passwords if they are exposed (revoke in Google Security settings, generate a new one)
- Monitor sent email activity: Google Workspace Admin → Reports → Email Log Search
