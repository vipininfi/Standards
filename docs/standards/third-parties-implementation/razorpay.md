# Razorpay Integration Standard

Covers Razorpay account setup, Orders, payment verification, Checkout.js, Subscriptions, Webhooks, Notes (metadata), and Refunds.

---

## 1. Account Setup and API Keys

### Key Types

| Key | Where Used | Exposure |
|-----|-----------|---------|
| `key_id` (`rzp_test_...` / `rzp_live_...`) | Frontend (Checkout.js) + Backend | Safe in frontend for Checkout only |
| `key_secret` | Backend only | **NEVER expose to browser** |
| Webhook secret | Backend webhook handler only | **NEVER expose** |

### Environment Variables

```
RAZORPAY_KEY_ID=rzp_test_...       # or rzp_live_...
RAZORPAY_KEY_SECRET=...            # NEVER in frontend
RAZORPAY_WEBHOOK_SECRET=...
```

- Use `rzp_test_` locally and in staging
- Use `rzp_live_` in production only
- Never commit keys to version control
- Rotate key_secret immediately if exposed

### Client Initialization (Backend)

```python
import razorpay

client = razorpay.Client(auth=(settings.RAZORPAY_KEY_ID, settings.RAZORPAY_KEY_SECRET))
```

---

## 2. Orders — Always Create Server-Side

**Never skip order creation.** Every Razorpay payment must begin with a server-side Order. The Order ID ties the payment back to your system.

### Creating an Order

```python
order = client.order.create({
    'amount': 29000,           # in paise — ₹290 = 29000 paise
    'currency': 'INR',
    'receipt': f'order_{workspace.id}_{user.id}',
    'notes': {
        'workspace_id': str(workspace.id),
        'user_id': str(user.id),
        'purpose': 'pro_upgrade',
    },
})
# Store order['id'] in DB before returning to client
order_record = Order.objects.create(
    razorpay_order_id=order['id'],
    workspace=workspace,
    amount_paise=29000,
    status='created',
)
return {'order_id': order['id'], 'amount': 29000, 'currency': 'INR'}
```

### Amount Rules

- Amount is always in **paise** (smallest Indian currency unit): ₹1 = 100 paise
- Never use floats — use integers only
- ₹290 → 29000 paise
- Validate: minimum ₹1 (100 paise), maximum varies by Razorpay plan

### Notes (Metadata)

`notes` is Razorpay's equivalent of Stripe metadata. Add workspace and user identifiers:

```python
'notes': {
    'workspace_id': str(workspace.id),
    'user_id': str(user.id),
    'plan': 'pro',
    'purpose': 'subscription_initial',
}
```

Rules:
- Maximum 15 key-value pairs
- Maximum 256 characters per value
- Use `snake_case` keys consistently
- Convert all values to `str()` (no integers or UUIDs directly)

---

## 3. Payment Verification — Critical Server-Side Step

### Why This Is Non-Negotiable

Razorpay's client-side `handler` callback fires when payment is complete, but **it can be faked**. Always verify server-side before activating anything.

### Verification Logic

After the client sends the payment details to your server:

```python
import hmac
import hashlib

def verify_razorpay_signature(order_id, payment_id, signature, key_secret):
    """
    Verify the Razorpay payment signature.
    Returns True if valid, False if tampered.
    """
    message = f'{order_id}|{payment_id}'.encode('utf-8')
    expected = hmac.new(
        key_secret.encode('utf-8'),
        message,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

### Server Endpoint (after client callback)

```python
def verify_payment(request):
    data = request.data  # from POST body
    order_id = data['razorpay_order_id']
    payment_id = data['razorpay_payment_id']
    signature = data['razorpay_signature']

    if not verify_razorpay_signature(order_id, payment_id, signature, settings.RAZORPAY_KEY_SECRET):
        return Response({'error': 'INVALID_SIGNATURE', 'message': 'Payment verification failed.'}, status=400)

    # Signature valid — update order status
    order = Order.objects.get(razorpay_order_id=order_id)
    order.razorpay_payment_id = payment_id
    order.status = 'paid'
    order.save()

    activate_subscription(order)
    return Response({'message': 'Payment verified.'})
```

### Rules

- Never trust `razorpay_payment_id` or `razorpay_order_id` from the client without verifying the signature
- Use `hmac.compare_digest()` for timing-safe comparison — never `==`
- Activate subscriptions/orders ONLY after successful server-side verification

---

## 4. Checkout.js Integration (Frontend)

### Loading Razorpay

```html
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
```

### Opening the Checkout

```js
async function initiatePayment() {
    // Step 1: create order server-side
    const res = await fetch('/api/create-order/', { method: 'POST', ... });
    const { order_id, amount, currency } = await res.json();

    // Step 2: open checkout
    const options = {
        key: RAZORPAY_KEY_ID,         // from env — publishable, safe for frontend
        amount: amount,               // in paise
        currency: currency,
        order_id: order_id,
        name: 'Your App Name',
        description: 'Pro Plan Subscription',
        handler: async function(response) {
            // Step 3: send to server for verification — do NOT activate here
            await fetch('/api/verify-payment/', {
                method: 'POST',
                body: JSON.stringify({
                    razorpay_order_id: response.razorpay_order_id,
                    razorpay_payment_id: response.razorpay_payment_id,
                    razorpay_signature: response.razorpay_signature,
                }),
            });
        },
        prefill: {
            name: user.name,
            email: user.email,
        },
        theme: { color: '#3b82f6' },
    };

    const rzp = new Razorpay(options);
    rzp.on('payment.failed', function(response) {
        showError('Payment failed: ' + response.error.description);
    });
    rzp.open();
}
```

### Rules

- Always pass `order_id` (from your server's Order creation) — never open checkout without it
- `handler` callback: send all three fields (`order_id`, `payment_id`, `signature`) to your server for verification — do NOT activate anything client-side
- Handle `payment.failed` event — show error to user

---

## 5. Subscriptions

### Plan Creation (one-time setup)

```python
plan = client.plan.create({
    'period': 'monthly',
    'interval': 1,
    'item': {
        'name': 'Pro Plan',
        'amount': 29000,   # in paise
        'unit_amount': 29000,
        'currency': 'INR',
    },
    'notes': {
        'plan': 'pro',
    },
})
```

Store `plan.id` in your config or DB — reference it when creating subscriptions.

### Creating a Subscription

```python
subscription = client.subscription.create({
    'plan_id': plan_id,
    'customer_notify': 1,
    'total_count': 12,        # number of billing cycles (12 = 1 year)
    'quantity': 1,
    'notes': {
        'workspace_id': str(workspace.id),
        'user_id': str(user.id),
    },
})
```

### Subscription States

| Status | Meaning | Action |
|--------|---------|--------|
| `created` | Not yet authorized | Do not grant access |
| `authenticated` | Authorized, first payment pending | Do not grant access yet |
| `active` | Paying | Grant access |
| `pending` | Payment retrying | Warn user |
| `halted` | Retries exhausted | Restrict access |
| `cancelled` | Cancelled | Revoke access |
| `completed` | All billing cycles done | Revoke or renew |
| `expired` | Past end date | Revoke access |

### Subscription Events to Handle via Webhook

```text
subscription.activated     → grant access
subscription.charged       → record billing
subscription.cancelled     → revoke access
subscription.halted        → restrict access, notify user
subscription.completed     → handle renewal or expiry
```

---

## 6. Webhooks

### Webhook Signature Verification

```python
import hmac
import hashlib

def verify_razorpay_webhook(payload_body, razorpay_signature, webhook_secret):
    expected = hmac.new(
        webhook_secret.encode('utf-8'),
        payload_body,        # raw bytes — do NOT decode or parse first
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, razorpay_signature)


def razorpay_webhook(request):
    payload = request.body       # raw bytes
    signature = request.META.get('HTTP_X_RAZORPAY_SIGNATURE', '')

    if not verify_razorpay_webhook(payload, signature, settings.RAZORPAY_WEBHOOK_SECRET):
        return HttpResponse(status=400)

    event = json.loads(payload)
    handle_event(event)
    return HttpResponse(status=200)
```

### Idempotency

```python
def handle_event(event):
    event_id = event.get('id') or event['payload']['payment']['entity']['id']
    if RazorpayEvent.objects.filter(event_id=event_id).exists():
        return  # Already processed
    RazorpayEvent.objects.create(event_id=event_id, event_type=event['event'])
    # process...
```

### Webhook Response Rules

- Always return **200** after receiving (even if skipping the event)
- Never return 500 for business logic failures — log and return 200
- Razorpay retries on non-200 responses — returning 500 causes duplicates

### Events to Handle

| Event | Action |
|-------|--------|
| `payment.authorized` | Payment authorized — capture if auto-capture disabled |
| `payment.captured` | Payment captured — activate order |
| `payment.failed` | Notify user, mark order as failed |
| `subscription.activated` | Grant subscription access |
| `subscription.charged` | Record payment in billing history |
| `subscription.cancelled` | Mark as cancelled, revoke at period end |
| `subscription.halted` | Restrict access, send payment failure email |
| `refund.created` | Record refund in DB, update payment status |

---

## 7. Refunds

```python
refund = client.payment.refund(payment_id, {
    'amount': 15000,         # partial refund in paise; omit for full refund
    'speed': 'normal',       # or 'optimum' for faster (higher cost)
    'notes': {
        'reason': 'Customer request',
        'refunded_by': str(admin_user.id),
    },
    'receipt': f'refund_{payment_id}',
})
```

### Rules

- Store `refund.id` in your database
- Confirm via webhook `refund.created` before marking fully refunded
- Partial refund: provide `amount` in paise; omit for full refund
- Speed options: `normal` (5-7 days), `optimum` (immediate if supported, higher cost)
- Reason is required by Razorpay for some refund types

---

## 8. Test Credentials

Use Razorpay's test mode with these test cards:

| Card | Result |
|------|--------|
| 4111 1111 1111 1111 | Success |
| 5104 0600 0000 0008 | Success (Mastercard) |
| 4111 1111 1111 1111 + CVV 123 | 3DS challenge |
| Any card with expiry in past | Decline |

UPI test IDs: `success@razorpay` (success), `failure@razorpay` (failure).

Net Banking test: select any bank in test mode — all succeed.
