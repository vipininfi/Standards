# Stripe Integration Standard

Covers every part of a Stripe integration: account setup, API keys, customers, products, payments, subscriptions, webhooks, metadata, error handling, and refunds.

---

## 1. Account Setup and API Keys

### Key Types

| Key | Where Used | Exposure |
|-----|-----------|---------|
| Publishable key (`pk_...`) | Frontend (browser/client) | Safe to expose |
| Secret key (`sk_...`) | Backend only | **NEVER expose to browser** |
| Webhook signing secret (`whsec_...`) | Backend webhook handler only | **NEVER expose** |

### Environment Variables

```
STRIPE_PUBLISHABLE_KEY=pk_test_...   # or pk_live_...
STRIPE_SECRET_KEY=sk_test_...        # or sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

- Use `pk_test_` / `sk_test_` locally and in staging
- Use `pk_live_` / `sk_live_` in production only
- Never commit any Stripe key to version control
- Rotate immediately if a secret key or webhook secret is exposed

### Local Webhook Testing

Use Stripe CLI to forward webhook events to your local server:

```bash
stripe listen --forward-to localhost:8000/webhooks/stripe/
```

The CLI prints a temporary webhook signing secret — use it in local `.env`.

---

## 2. Stripe Customer

### When to Create a Customer

Create a Stripe Customer:
- When a user initiates their first payment, OR
- At user sign-up if your app is payment-centric

Do NOT create a Stripe Customer at sign-up for apps where most users never pay.

### What to Store

Store `stripe_customer_id` in your database against the user record:

```sql
ALTER TABLE users ADD COLUMN stripe_customer_id text UNIQUE;
```

### Customer Metadata

Always set metadata on the Customer object:

```python
stripe.Customer.create(
    email=user.email,
    name=user.get_full_name(),
    metadata={
        'user_id': str(user.id),
        'workspace_id': str(workspace.id),
    }
)
```

### Never Store Card Details

Do not store raw card numbers, CVVs, or full card data. Stripe's tokenization (via Stripe.js / Elements) handles this entirely.

---

## 3. Products and Prices

### Concepts

- **Product**: what you sell (e.g., "Pro Plan", "Enterprise Plan")
- **Price**: how you sell it (e.g., $29/month, $290/year, $0.01/API call)
- One product can have many prices (monthly vs annual, different currencies)

### Where to Create

| Scenario | Create In |
|----------|-----------|
| Fixed subscription tiers | Stripe Dashboard — create once, reference by price_id |
| Dynamic pricing (per-seat, usage-based) | Code — create via API at runtime |
| Marketing changes copy/price often | Dashboard — non-engineers can update |

### Product Metadata

```python
# Set when creating a product in code
stripe.Product.create(
    name='Pro Plan',
    metadata={
        'plan': 'pro',
        'feature_set': 'analytics,api,export',
    }
)
```

### Price Lookup Keys

Use lookup keys for prices so you reference them by name, not hardcoded price_id:

```python
stripe.Price.create(
    product=product.id,
    unit_amount=2900,  # in cents
    currency='usd',
    recurring={'interval': 'month'},
    lookup_key='pro-monthly',
)
```

Then retrieve by lookup key instead of hardcoding:

```python
prices = stripe.Price.list(lookup_keys=['pro-monthly'])
price_id = prices.data[0].id
```

---

## 4. One-Time Payments — Payment Intents

### Flow

```
Client: "Start payment" 
→ Server: create PaymentIntent → returns client_secret
→ Client: stripe.confirmCardPayment(client_secret)
→ Stripe: confirms payment
→ Webhook: payment_intent.succeeded → update DB
```

### Creating a Payment Intent (server-side)

```python
intent = stripe.PaymentIntent.create(
    amount=2900,           # in cents — never floats
    currency='usd',
    customer=user.stripe_customer_id,
    metadata={
        'workspace_id': str(workspace.id),
        'user_id': str(user.id),
        'purpose': 'pro_upgrade',
    },
    idempotency_key=f'{user.id}-{workspace.id}-{purpose}-{timestamp}',
)
return {'client_secret': intent.client_secret}
```

### Rules

- Amount always in **smallest currency unit** (cents for USD, pence for GBP, paise for INR)
- Use integer amounts — never floats
- Return `client_secret` to the frontend — never the secret key
- Use idempotency keys to prevent duplicate charges on retried requests
- Activate after success via **webhook** (`payment_intent.succeeded`), not the client-side callback

### Confirming on the Client (Stripe.js)

```js
const stripe = Stripe(PUBLISHABLE_KEY);  // from env, not hardcoded

const {error} = await stripe.confirmCardPayment(clientSecret, {
    payment_method: { card: cardElement }
});

if (error) {
    // Show error to user — never log raw error to analytics
} else {
    // Payment pending — wait for webhook to update backend
}
```

---

## 5. Checkout Sessions (Hosted Checkout)

Use Checkout Sessions when you want Stripe to host the payment page (simpler integration, less custom UI).

```python
session = stripe.checkout.Session.create(
    customer=user.stripe_customer_id,
    payment_method_types=['card'],
    line_items=[{
        'price': 'price_id_here',
        'quantity': 1,
    }],
    mode='payment',  # or 'subscription'
    success_url=f'{APP_BASE_URL}/payment/success?session_id={{CHECKOUT_SESSION_ID}}',
    cancel_url=f'{APP_BASE_URL}/payment/cancelled',
    metadata={
        'workspace_id': str(workspace.id),
        'user_id': str(user.id),
    },
)
return {'checkout_url': session.url}
```

### Rules for Checkout Sessions

- Activate via **webhook** (`checkout.session.completed`), not the `success_url` redirect — the redirect is unreliable (user might close browser before it loads)
- Include `{CHECKOUT_SESSION_ID}` in `success_url` so you can verify if needed
- Set metadata on the session for webhook processing

---

## 6. Subscriptions

### Creating a Subscription

```python
subscription = stripe.Subscription.create(
    customer=user.stripe_customer_id,
    items=[{'price': price_id}],
    trial_period_days=14,      # optional
    metadata={
        'workspace_id': str(workspace.id),
        'plan': 'pro',
    },
)
```

### Subscription States

| Status | Meaning | Action |
|--------|---------|--------|
| `active` | Paying and active | Grant access |
| `trialing` | In free trial | Grant access |
| `past_due` | Payment failed, retrying | Warn user, restrict non-critical features |
| `canceled` | Ended | Revoke access |
| `unpaid` | All retries failed | Restrict access |
| `incomplete` | First payment pending | Do not grant access yet |

### Handling Subscription Lifecycle via Webhooks

```python
# Events to handle:
'customer.subscription.created'   → provision access
'customer.subscription.updated'   → update plan/status in DB
'customer.subscription.deleted'   → revoke access
'invoice.payment_succeeded'       → record payment, send receipt
'invoice.payment_failed'          → notify user, update payment status
```

---

## 7. Webhooks

### Why Webhooks, Not Client Callbacks

Client-side callbacks (success page redirect) are **unreliable** — the user might close the browser, lose connection, or the callback might never fire. Always use webhooks as the source of truth for payment state.

### Signature Verification (Required, Always)

```python
import stripe
from django.http import HttpResponse

def stripe_webhook(request):
    payload = request.body
    sig_header = request.META.get('HTTP_STRIPE_SIGNATURE')
    
    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, settings.STRIPE_WEBHOOK_SECRET
        )
    except ValueError:
        return HttpResponse(status=400)  # Invalid payload
    except stripe.error.SignatureVerificationError:
        return HttpResponse(status=400)  # Invalid signature
    
    handle_event(event)
    return HttpResponse(status=200)
```

### Idempotency — Process Each Event Once

```python
def handle_event(event):
    # Check if already processed
    if StripeEvent.objects.filter(event_id=event['id']).exists():
        return  # Skip duplicate
    
    StripeEvent.objects.create(event_id=event['id'], type=event['type'])
    
    if event['type'] == 'payment_intent.succeeded':
        handle_payment_succeeded(event['data']['object'])
    elif event['type'] == 'customer.subscription.deleted':
        handle_subscription_cancelled(event['data']['object'])
    # ...
```

### Webhook Response Rules

- Always return **200** after receiving the event (even if you skip processing it)
- Never return 500 for business logic failures — log the error and return 200
- Stripe retries on 4xx/5xx responses — returning 500 causes duplicate processing

### Events to Handle

| Event | Action |
|-------|--------|
| `payment_intent.succeeded` | Activate order/access, send receipt |
| `payment_intent.payment_failed` | Notify user, do not activate |
| `checkout.session.completed` | Activate purchase |
| `customer.subscription.created` | Provision plan access |
| `customer.subscription.updated` | Sync plan changes to DB |
| `customer.subscription.deleted` | Revoke access after period ends |
| `invoice.payment_succeeded` | Record payment in billing history |
| `invoice.payment_failed` | Email user, mark payment as failed |

---

## 8. Metadata Standards

Consistent metadata keys across all Stripe objects:

| Object | Required Metadata Keys |
|--------|----------------------|
| Customer | `user_id`, `workspace_id` |
| PaymentIntent | `workspace_id`, `user_id`, `purpose` |
| Checkout Session | `workspace_id`, `user_id`, `plan` (if subscription) |
| Subscription | `workspace_id`, `plan` |
| Invoice | auto-inherited from subscription |

### Rules

- Use `snake_case` for all metadata keys
- Values must be strings (convert UUIDs and integers to `str()`)
- Metadata values max 500 characters, max 50 keys
- Do NOT store sensitive data (passwords, secrets, PII beyond what Stripe already has) in metadata

---

## 9. Error Handling

### Decline Codes

| Code | User Message |
|------|-------------|
| `card_declined` | "Your card was declined. Please try a different card." |
| `insufficient_funds` | "Your card has insufficient funds." |
| `incorrect_cvc` | "Your card's security code is incorrect." |
| `expired_card` | "Your card has expired." |
| `processing_error` | "An error occurred. Please try again." |

Never show raw Stripe error messages to users. Map decline codes to user-friendly messages.

### 3D Secure / SCA (Strong Customer Authentication)

```js
const {paymentIntent, error} = await stripe.confirmCardPayment(clientSecret);

if (error?.payment_intent?.status === 'requires_action') {
    // Stripe.js handles the 3DS challenge automatically with confirmCardPayment
    // Show user: "Your bank requires additional verification."
}
```

### Network / API Errors

```python
try:
    intent = stripe.PaymentIntent.create(...)
except stripe.error.CardError as e:
    # Decline — show decline message
    return {'error': e.user_message}
except stripe.error.RateLimitError:
    # Too many requests — retry later
    raise
except stripe.error.InvalidRequestError:
    # Invalid parameters — log, fix code
    raise
except stripe.error.StripeError:
    # General error — log, show generic error
    raise
```

---

## 10. Refunds

```python
refund = stripe.Refund.create(
    payment_intent=payment_intent_id,
    amount=500,   # partial refund in cents; omit for full refund
    reason='requested_by_customer',  # or 'fraudulent', 'duplicate'
    metadata={
        'refunded_by': str(admin_user.id),
        'reason_detail': 'Customer reported non-delivery',
    },
)
```

### Rules

- Always refund via API — never manually in the Stripe Dashboard (for consistency with your DB records)
- Store `refund_id` in your database
- Update order/payment status in your DB after successful refund
- Confirm via webhook `charge.refunded` before marking as fully refunded in DB
- Partial refunds: specify `amount` in cents; omit for full refund
