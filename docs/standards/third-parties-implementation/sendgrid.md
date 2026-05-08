# SendGrid Integration Standard

Covers SendGrid account setup, API key management, when to use SendGrid vs other providers, the decision between code-rendered and SendGrid Dynamic Templates, transactional email standards, error handling, and bounce management.

---

## 1. When to Use SendGrid

| Use Case | Use SendGrid? | Why |
|----------|--------------|-----|
| Production transactional email (high volume) | Yes | Deliverability, analytics, bounce handling |
| Marketing and promotional emails | Yes | Campaign management, unsubscribe, tracking |
| A/B testing email content | Yes | Built-in testing tools |
| Non-technical team edits email copy | Yes | Dashboard template editor |
| Development / local testing | No | Use a local SMTP catcher (Mailhog, MailPit) |
| Small internal tool (< 50 emails/day) | Maybe | Gmail SMTP is simpler; see Gmail standard |
| System alerts to developers only | No | Gmail or internal Slack notification |

---

## 2. Account Setup

### Sender Authentication

Before sending any email, authenticate your sending domain:

1. **Domain Authentication** (preferred for production): adds SPF, DKIM, and DMARC records via DNS. Emails arrive from `@yourdomain.com` — best deliverability.
2. **Single Sender Verification** (acceptable for early development): verify a single email address. Less reliable for deliverability at scale.

Always use Domain Authentication in production.

### From Address Standards

| Setting | Rule |
|---------|------|
| From email | Must be from your authenticated domain — never `@gmail.com` or `@hotmail.com` |
| From name | `"Your App Name"` or `"Your App Name Team"` |
| Reply-to | Set to a **monitored** inbox (`support@yourdomain.com`) — not the same as no-reply |
| No-reply | Acceptable for purely transactional email; still set reply-to |

---

## 3. API Key Management

### Creating the API Key

1. Go to Settings → API Keys → Create API Key
2. Use **Restricted Access** — grant only what's needed:
   - Transactional email service: `Mail Send` permission only
   - If using the Stats API: add `Stats` read permission
3. Never use a Full Access key in application code

### Environment Variables

```
SENDGRID_API_KEY=SG.xxxxx
SENDGRID_FROM_EMAIL=no-reply@yourdomain.com
SENDGRID_FROM_NAME=Your App Name
```

Never commit the API key to version control. Rotate immediately if exposed.

---

## 4. Template Decision: Code-Rendered vs SendGrid Dynamic Template

This is the most important decision in your email setup. Pick one per email type — never maintain both.

### Use Code-Rendered Templates When

- Developers control all email content (no non-technical editors)
- Template content changes rarely
- Email is technical (error notifications, system alerts, developer digests)
- You need complex programmatic logic in the template (loops, nested conditions)
- You want email templates in version control with the rest of the code

### Use SendGrid Dynamic Templates When

- Marketing or product team needs to update copy without a deployment
- You need A/B testing between template variants
- You need multi-language versions of the same email
- The email is marketing/promotional (newsletter, onboarding drip)
- You want email performance analytics (open rate, click rate) per template

### Decision Table

| Email Type | Recommended Approach |
|------------|---------------------|
| Password reset | Code-rendered (developer-controlled, security-sensitive) |
| Email verification | Code-rendered |
| Invoice/receipt | Code-rendered (complex data, line items) |
| Workspace invite | Code-rendered |
| Welcome email | Dynamic template (marketing may want to iterate copy) |
| Promotional offer | Dynamic template |
| Newsletter | Dynamic template |
| System error alert | Code-rendered (or Slack) |
| Onboarding sequence | Dynamic template |

---

## 5. Code-Rendered Templates

### File Structure

```
templates/
  email/
    base_email.html         ← base layout with header/footer
    base_email.txt          ← plaintext base
    password_reset.html
    password_reset.txt
    workspace_invite.html
    workspace_invite.txt
    invoice.html
    invoice.txt
```

Always create both HTML and plaintext versions. Some email clients only render plain text.

### Base Email Template (HTML)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ subject }}</title>
</head>
<body style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;">
  <header>
    <img src="{{ LOGO_URL }}" alt="{{ APP_NAME }}" height="40">
  </header>
  <main>
    {% block content %}{% endblock %}
  </main>
  <footer style="margin-top: 40px; font-size: 12px; color: #666;">
    <p>{{ APP_NAME }} · {{ APP_ADDRESS }}</p>
    <p>This email was sent to {{ recipient_email }}.</p>
  </footer>
</body>
</html>
```

### Sending a Code-Rendered Email (Django)

```python
from django.template.loader import render_to_string
from django.core.mail import EmailMultiAlternatives
from django.conf import settings


def send_invite_email(to_email, workspace_name, invite_link, inviter_name):
    subject = f'{inviter_name} invited you to join {workspace_name}'
    context = {
        'workspace_name': workspace_name,
        'invite_link': invite_link,
        'inviter_name': inviter_name,
        'recipient_email': to_email,
        'APP_NAME': settings.APP_NAME,
    }
    html_body = render_to_string('email/workspace_invite.html', context)
    text_body = render_to_string('email/workspace_invite.txt', context)

    msg = EmailMultiAlternatives(
        subject=subject,
        body=text_body,
        from_email=f'{settings.SENDGRID_FROM_NAME} <{settings.SENDGRID_FROM_EMAIL}>',
        to=[to_email],
    )
    msg.attach_alternative(html_body, 'text/html')
    msg.send()
```

### Django Email Backend (settings.py)

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.sendgrid.net'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'apikey'                          # literal string 'apikey'
EMAIL_HOST_PASSWORD = env('SENDGRID_API_KEY')
DEFAULT_FROM_EMAIL = env('SENDGRID_FROM_EMAIL')
```

---

## 6. SendGrid Dynamic Templates

### Creating a Template

1. Go to Email API → Dynamic Templates → Create Template
2. Name it clearly: `workspace-invite`, `password-reset`, `welcome-email`
3. Create a version inside the template
4. Design using the editor or import HTML
5. Use Handlebars syntax for variables: `{{variable_name}}`
6. Always click "Send Test" before saving to production

### Template Variables (Handlebars)

```html
<h1>Hi {{first_name}},</h1>
<p>{{inviter_name}} invited you to join <strong>{{workspace_name}}</strong> as a {{role}}.</p>
<a href="{{invite_link}}">Accept Invite</a>

{{#if expires_in_hours}}
<p>This invite expires in {{expires_in_hours}} hours.</p>
{{/if}}
```

### Storing Template IDs

Store template IDs in environment variables — never hardcode:

```
SENDGRID_TEMPLATE_INVITE=d-abc123def456...
SENDGRID_TEMPLATE_PASSWORD_RESET=d-xyz789...
SENDGRID_TEMPLATE_WELCOME=d-...
```

Keep a `sendgrid-templates.md` in your repo (not committed to production branches) or a shared doc mapping `template_id → purpose → variables` for reference.

### Sending with a Dynamic Template (Python)

```python
import sendgrid
from sendgrid.helpers.mail import Mail, To

def send_invite_email_dynamic(to_email, workspace_name, invite_link, inviter_name):
    sg = sendgrid.SendGridAPIClient(api_key=settings.SENDGRID_API_KEY)
    
    message = Mail(
        from_email=(settings.SENDGRID_FROM_EMAIL, settings.SENDGRID_FROM_NAME),
        to_emails=To(to_email),
    )
    message.template_id = settings.SENDGRID_TEMPLATE_INVITE
    message.dynamic_template_data = {
        'workspace_name': workspace_name,
        'invite_link': invite_link,
        'inviter_name': inviter_name,
        'first_name': to_email.split('@')[0],  # fallback if no first name
    }
    
    response = sg.send(message)
    return response.status_code
```

---

## 7. Transactional Email Standards

### Subject Lines

| Rule | Example |
|------|---------|
| Be specific — say what the email is | "Your password reset link" not "Action required" |
| Include the app or workspace name | "[YourApp] Your invoice for March 2025" |
| No spammy words: FREE, URGENT, !!!, WINNER | Avoid — triggers spam filters |
| Max 50 characters for mobile | Test on mobile preview |

### Email Link Standards

| Link Type | Expiry | Rules |
|-----------|--------|-------|
| Password reset | 1 hour | Invalidate after single use |
| Email verification | 24 hours | Invalidate after single use |
| Workspace invite | 7 days | Mark as used on accept |
| Unsubscribe link | Permanent | Required for marketing emails |

### Transactional vs Marketing — Legal Difference

- **Transactional** (password reset, invoice, order confirmation): no unsubscribe link required
- **Marketing** (newsletter, promotional offer, onboarding drip): unsubscribe link **legally required** in most regions (CAN-SPAM, GDPR)
- When in doubt, add the unsubscribe link

---

## 8. Error Handling and Logging

### Log Every Send Attempt

```python
class EmailLog(models.Model):
    to_email = models.EmailField()
    template_name = models.CharField(max_length=100)
    subject = models.CharField(max_length=200, blank=True)
    status = models.CharField(max_length=20)   # 'sent', 'failed'
    sendgrid_message_id = models.CharField(max_length=200, blank=True)
    error_message = models.TextField(blank=True)
    sent_at = models.DateTimeField(auto_now_add=True)
```

### Retry Logic

```python
import time

def send_email_with_retry(send_fn, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            send_fn()
            return
        except Exception as e:
            if attempt == max_attempts - 1:
                log_email_failure(e)
                # Do NOT raise for non-critical emails (notifications)
                # DO raise for critical emails (password reset)
            time.sleep(2 ** attempt)  # exponential backoff: 1s, 2s, 4s
```

### Error Severity

| Email Type | On Failure |
|-----------|-----------|
| Password reset | Raise — user cannot proceed |
| Email verification | Raise — user cannot verify |
| Workspace invite | Log + show error in UI — admin can resend |
| Welcome email | Log only — non-critical |
| System notification | Log only — non-critical |

---

## 9. Bounce and Suppression Handling

SendGrid automatically maintains a suppression list for bounced addresses, spam reports, and unsubscribes.

- Do NOT attempt to re-send to suppressed addresses — it damages your sender reputation
- Check suppressions before sending bulk email campaigns
- For important transactional emails (password reset) where the address bounced: show an error to the user asking them to check their email address

### Viewing Suppressions (API)

```python
sg = sendgrid.SendGridAPIClient(settings.SENDGRID_API_KEY)
response = sg.client.suppression.bounces.get()
```

Monitor your bounce rate in the SendGrid dashboard — keep it below 2%.
