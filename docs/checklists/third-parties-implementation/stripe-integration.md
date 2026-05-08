# Checklist — Stripe Integration

Run this checklist every time you add or change a Stripe payment flow.

**Standard:** [Stripe Standard](../../standards/third-parties-implementation/stripe.md)

---

## 1. Account and Keys

[ ] `STRIPE_PUBLISHABLE_KEY` stored in environment variable — not hardcoded
[ ] `STRIPE_SECRET_KEY` stored in environment variable — never exposed to browser/client
[ ] `STRIPE_WEBHOOK_SECRET` stored in environment variable
[ ] Test keys (`pk_test_`, `sk_test_`) used in local and staging
[ ] Live keys (`pk_live_`, `sk_live_`) used in production only
[ ] No Stripe key committed to version control
[ ] Stripe CLI set up locally for webhook forwarding (`stripe listen --forward-to ...`)

## 2. Stripe Customer

[ ] `stripe_customer_id` stored in your database users/billing table
[ ] Customer created with `metadata.user_id` and `metadata.workspace_id`
[ ] Customer email matches the authenticated user's email
[ ] Customer created server-side — never via frontend

## 3. Products and Prices

[ ] Products and prices created in Stripe Dashboard (or via API if dynamic)
[ ] Price lookup keys used where possible — not hardcoded price IDs
[ ] Price lookup keys stored in environment variables if used in code

## 4. Payment Intents (One-Time Payments)

[ ] PaymentIntent created server-side with secret key
[ ] Amount in smallest currency unit (cents, paise) — integer, not float
[ ] Metadata set: `workspace_id`, `user_id`, `purpose`
[ ] Idempotency key used to prevent duplicate charges
[ ] Only `client_secret` returned to frontend — not the secret key
[ ] Stripe.js / Elements used on frontend to collect card details
[ ] Payment activated via webhook `payment_intent.succeeded` — NOT the client-side callback

## 5. Checkout Sessions (if used)

[ ] Session created server-side
[ ] `success_url` includes `{CHECKOUT_SESSION_ID}` parameter
[ ] Metadata set on session: `workspace_id`, `user_id`
[ ] Activation done via webhook `checkout.session.completed` — not the success_url redirect

## 6. Subscriptions (if used)

[ ] Subscription created with correct `customer` and `price_id`
[ ] Metadata set: `workspace_id`, `plan`
[ ] Trial period set if applicable (`trial_period_days`)
[ ] All 8 subscription states handled (active, trialing, past_due, canceled, unpaid, incomplete, etc.)

## 7. Webhooks

[ ] Webhook endpoint registered in Stripe Dashboard (or Stripe CLI for local)
[ ] Signature verified using `stripe.Webhook.construct_event()` before processing
[ ] Raw request body used for signature verification — NOT parsed JSON
[ ] Returns 200 on success, 400 on invalid signature
[ ] Returns 200 even for skipped/ignored events (not 404 or 400)
[ ] Idempotency: event ID stored in DB — duplicate events skipped
[ ] Business logic failures logged and return 200 (not 500) to prevent Stripe retries
[ ] Handles: `payment_intent.succeeded`, `payment_intent.payment_failed`
[ ] Handles: `customer.subscription.created`, `updated`, `deleted`
[ ] Handles: `invoice.payment_succeeded`, `invoice.payment_failed`

## 8. Metadata

[ ] Customer metadata: `user_id`, `workspace_id`
[ ] PaymentIntent metadata: `workspace_id`, `user_id`, `purpose`
[ ] Subscription metadata: `workspace_id`, `plan`
[ ] All metadata values are strings (`str()` applied to UUIDs and integers)
[ ] No sensitive data in metadata (passwords, secrets)

## 9. Error Handling

[ ] Decline codes mapped to user-friendly messages (never raw Stripe error shown to user)
[ ] `stripe.error.CardError` caught and user shown card decline message
[ ] `stripe.error.StripeError` caught — generic "payment error" shown to user, error logged
[ ] SCA/3DS handled via `stripe.confirmCardPayment()` in Stripe.js

## 10. Refunds

[ ] Refund created via API — not via Dashboard manually
[ ] `refund_id` stored in database
[ ] Refund confirmed via webhook `charge.refunded`
[ ] Partial refund uses `amount` field in paise/cents

---

**Standard:** [Stripe Standard](../../standards/third-parties-implementation/stripe.md)
