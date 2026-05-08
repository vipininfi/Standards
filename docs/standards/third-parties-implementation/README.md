# Third-Party Implementation Standards

Standards for integrating external services into our applications. Each standard covers full setup, configuration, usage patterns, metadata conventions, and error handling specific to that service.

---

## Standards in This Folder

| File | Service | Covers |
|------|---------|--------|
| [stripe.md](stripe.md) | Stripe | Payment Intents, Checkout, Subscriptions, Webhooks, Metadata, Refunds |
| [razorpay.md](razorpay.md) | Razorpay | Orders, Payment Verification, Subscriptions, Webhooks, Notes, Refunds |
| [sendgrid.md](sendgrid.md) | SendGrid | Setup, Template Decision, Dynamic Templates, Transactional Standards, Error Handling |
| [gmail.md](gmail.md) | Gmail | OAuth2, SMTP, Rate Limits, When to Use, When Not to Use |

---

## Core Principles

1. **API keys never in code** — always in environment variables, never committed
2. **Secrets server-side only** — Stripe secret key, Razorpay key_secret, SendGrid API key never exposed to the browser
3. **Webhook signatures always verified** — before processing any payload
4. **Idempotency on webhooks** — store event IDs, skip already-processed events
5. **Never 500 for business logic in webhooks** — return 200 with a log entry instead
6. **Test mode before live** — use test credentials locally and in staging, live keys only in production

---

## Related Checklists

- [Stripe Integration](../../checklists/third-parties-implementation/stripe-integration.md)
- [Razorpay Integration](../../checklists/third-parties-implementation/razorpay-integration.md)
- [SendGrid Integration](../../checklists/third-parties-implementation/sendgrid-integration.md)
- [Gmail Integration](../../checklists/third-parties-implementation/gmail-integration.md)
