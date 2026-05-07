# Form Documentation Template

Use this to document every form before or during development. Backend and frontend must both know this information before building.

---

## Template

```md
# Form: [Form Name]

## Page
[URL or page name where this form lives]

## Purpose
[What does this form do? Who fills it out?]

---

## Fields

| Field | Type | Required | Validation | Placeholder |
|-------|------|----------|------------|-------------|
| [name] | text | Yes | 2–100 chars | Enter full name |
| [email] | email | Yes | Valid email | Enter email address |
| [phone] | phone | No | Country code + valid number | +91 9876543210 |
| [message] | textarea | Yes | 10–1000 chars | Type your message |
| [dropdown] | select | Yes | Must be one of: [options] | Select an option |
| [file] | file | No | PDF/JPG, max 5MB | Upload file |

---

## Submit Destination

- CRM: [CRM name and lead field mapping]
- Email Notification: [Who gets notified and to which email]
- Database: [Table and fields]
- Webhook: [Endpoint if applicable]

---

## Success Behavior

Message shown to user:
```text
[Success message text]
```

Actions after success:
- [ ] Form fields cleared
- [ ] Redirect to: [URL]
- [ ] Email confirmation sent to user
- [ ] Notification sent to team

---

## Error Behavior

Message shown to user on API failure:
```text
[Error message text]
```

Field-level error examples:

```text
Name: "Please enter your full name."
Email: "Enter a valid email address."
Phone: "Enter a valid phone number with country code."
Message: "Message must be at least 10 characters."
```

---

## Spam Protection

- [ ] CAPTCHA (reCAPTCHA / hCaptcha)
- [ ] Honeypot field
- [ ] Rate limiting
- [ ] None (explain why)

---

## Frontend Behavior

```text
[ ] Submit button disables while request is pending.
[ ] Loading state shown on submit button ("Sending...").
[ ] Form values preserved if request fails.
[ ] Fields cleared on successful submit.
[ ] Success message shown after submit.
[ ] Field-level errors shown near each invalid field.
[ ] Mobile layout tested.
```

---

## Backend / API Used

[API name, table name, RPC function, or Edge Function]

See: [Link to API documentation]

---

## File Upload (if applicable)

| Rule | Value |
|------|-------|
| Allowed Types | PDF, JPG, PNG |
| Max Size | 5MB |
| Storage Path | workspace/{workspace_id}/uploads/{file_id} |
| Virus Scan | Yes / No |
| Owner | User who uploaded |
```

---

## Example — Contact Form

```md
# Form: Contact Form

## Page
/contact

## Purpose
Collects website visitor inquiries and routes them to the sales team.

---

## Fields

| Field | Type | Required | Validation | Placeholder |
|-------|------|----------|------------|-------------|
| name | text | Yes | 2–100 chars | Enter your full name |
| email | email | Yes | Valid email | Enter your email |
| phone | phone | No | Country code + valid number | +91 9876543210 |
| message | textarea | Yes | 10–1000 chars | How can we help you? |

---

## Submit Destination

- CRM: Zoho CRM — creates lead with name, email, phone, message
- Email Notification: sales@example.com
- Database: public.leads table

---

## Success Behavior

Message shown to user:
"Thank you. Our team will contact you within 24 hours."

Actions after success:
- Form fields cleared
- No redirect (stay on contact page)

---

## Error Behavior

Message on failure:
"We could not submit your request. Please try again."

---

## Spam Protection
- Honeypot field
- Rate limiting: 3 submissions per IP per hour

---

## Frontend Behavior

- Disable submit while sending.
- Show "Sending..." on button.
- Preserve fields on error.
- Clear fields on success.
- Show success notification.
```
