# Checklist — Razorpay Integration

Run this checklist every time you add or change a Razorpay payment flow.

**Standard:** [Razorpay Standard](../../standards/third-parties-implementation/razorpay.md)

---

## 1. Account and Keys

[ ] `RAZORPAY_KEY_ID` stored in environment variable
[ ] `RAZORPAY_KEY_SECRET` stored in environment variable — never exposed to browser
[ ] `RAZORPAY_WEBHOOK_SECRET` stored in environment variable
[ ] Test keys (`rzp_test_`) used in local and staging
[ ] Live keys (`rzp_live_`) used in production only
[ ] No Razorpay key committed to version control

## 2. Orders

[ ] Order created server-side before opening Razorpay checkout (never skip this step)
[ ] Amount in paise (integer) — never floats: ₹290 = 29000 paise
[ ] `notes` set with: `workspace_id`, `user_id`, `purpose`
[ ] Order ID (`razorpay_order_id`) stored in your DB before returning to client
[ ] `receipt` field set on order for your own reference tracking

## 3. Payment Verification — Critical

[ ] Server-side verification endpoint exists and is called after every payment
[ ] Signature verified using HMAC SHA256: `order_id + "|" + payment_id` signed with `key_secret`
[ ] `hmac.compare_digest()` used for comparison — NOT `==` (timing-safe)
[ ] Payment never activated client-side — always waits for server verification
[ ] Returns 400 error if signature is invalid
[ ] DB update only happens after successful signature verification

## 4. Checkout.js (Frontend)

[ ] Razorpay checkout.js script loaded from `https://checkout.razorpay.com/v1/checkout.js`
[ ] `key_id` used in Checkout.js options (publishable) — not `key_secret`
[ ] `order_id` passed from server — not omitted
[ ] `handler` callback sends all three values to server: `razorpay_order_id`, `razorpay_payment_id`, `razorpay_signature`
[ ] `handler` does NOT activate anything client-side before server verification
[ ] `payment.failed` event handled — error shown to user

## 5. Subscriptions (if used)

[ ] Plan created in Razorpay (amount, period, interval)
[ ] Subscription created with `plan_id` and `notes`
[ ] All subscription states handled: created, authenticated, active, pending, halted, cancelled, completed, expired
[ ] Subscription events handled via webhook

## 6. Webhooks

[ ] Webhook endpoint registered in Razorpay Dashboard
[ ] Signature verified using HMAC SHA256 of raw request body with `RAZORPAY_WEBHOOK_SECRET`
[ ] Raw request body (bytes) used for signature — NOT parsed JSON/string
[ ] `X-Razorpay-Signature` header used for comparison
[ ] Returns 200 after successful receipt
[ ] Returns 200 even for skipped events — not 400/500
[ ] Idempotency: event ID stored in DB — duplicate events skipped
[ ] Business logic failures return 200 with a log entry — not 500
[ ] Handles: `payment.captured`, `payment.failed`
[ ] Handles: `subscription.activated`, `subscription.charged`, `subscription.cancelled`, `subscription.halted`
[ ] Handles: `refund.created`

## 7. Notes (Metadata)

[ ] Notes set on Order: `workspace_id`, `user_id`, `purpose` (and any other relevant identifiers)
[ ] All note values are strings
[ ] Maximum 15 key-value pairs per object respected
[ ] Note values ≤ 256 characters each

## 8. Error Handling

[ ] Payment failure shown to user with descriptive message (not "Something went wrong")
[ ] `payment.failed` event from Checkout.js handled — shows `error.description` to user
[ ] Server verification failure returns a clear error to frontend
[ ] Network errors during order creation shown to user with retry option

## 9. Refunds

[ ] Refund issued via `client.payment.refund(payment_id, {...})`
[ ] `refund_id` stored in database
[ ] Refund confirmed via webhook `refund.created` before marking DB as fully refunded
[ ] Partial refund uses `amount` field in paise

---

**Standard:** [Razorpay Standard](../../standards/third-parties-implementation/razorpay.md)
