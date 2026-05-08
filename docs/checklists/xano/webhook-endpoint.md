# Checklist — Xano Webhook Endpoint

> Run this checklist every time you build an endpoint that receives a webhook from an external service (Stripe, SendGrid, Calendly, etc.).

**Standard:** [Xano Standards](../../standards/xano-standards.md)

---

## 1. Webhook Signature Verification

```text
[ ] Signature verified BEFORE processing any payload
[ ] Signature secret stored in Xano Environment Variables — not hardcoded
[ ] Request rejected with 401 if signature is invalid
[ ] Signature header name known and documented: Stripe-Signature, Svix-Signature, etc.
[ ] Timing-safe comparison used for signature verification (not string equality)
```

---

## 2. Endpoint Configuration

```text
[ ] Endpoint is unauthenticated (no Xano Auth Token check — webhook has its own auth)
[ ] Method is POST (all webhooks are POST)
[ ] URL is unique and not guessable (include a random path segment if needed)
[ ] Rate limiting considered — Xano rate limit configured on this endpoint
```

---

## 3. Payload Handling

```text
[ ] Payload type validated: expected event type checked before processing
    (e.g., only handle "checkout.session.completed" — ignore other Stripe events)
[ ] Required fields extracted and validated before use
[ ] Unexpected or missing fields handled gracefully — do not crash on unknown event types
[ ] Idempotency key stored: if same webhook fires twice, do not process twice
[ ] Idempotency check: look up event_id in a webhook_events table before processing
```

---

## 4. Processing Logic

```text
[ ] Processing logic separated from signature verification:
    — Step 1: Verify signature → reject if invalid
    — Step 2: Parse payload → extract event type + data
    — Step 3: Idempotency check → skip if already processed
    — Step 4: Business logic → update database
    — Step 5: Mark as processed → insert into webhook_events
[ ] Business logic uses reusable functions where appropriate
[ ] Return 200 quickly — do not do slow blocking operations synchronously
```

---

## 5. Response

```text
[ ] Always return 200 after successful receipt (even if you skipped due to idempotency)
[ ] Return 401 for signature failure
[ ] Return 400 for malformed payload
[ ] Do NOT return 500 for business logic failures — return 200 and log the error internally
    (returning 500 causes the provider to retry and may cause duplicate processing)
[ ] Response body is minimal: { "received": true }
```

---

## 6. Logging

```text
[ ] Every received webhook logged with: timestamp, event type, event ID
[ ] Failed signature verifications logged with: timestamp, source IP
[ ] Processing errors logged with: event ID, error message
[ ] Do NOT log full payloads if they contain payment card data or PII
[ ] webhook_events table: stores event_id, type, status (received/processed/failed), timestamp
```

---

## 7. Error Handling

```text
[ ] If business logic fails: log the error, mark webhook as failed in webhook_events
[ ] Retry logic: if the provider retries, idempotency check prevents double processing
[ ] Failed webhooks: replayable from provider dashboard (Stripe, etc.) if needed
```

---

## 8. Testing

```text
[ ] Tested with a real payload from the provider's test mode
[ ] Signature verification tested: valid signature → processed; invalid → rejected
[ ] Idempotency tested: same event sent twice → processed only once
[ ] Unknown event type tested: ignored gracefully, 200 returned
[ ] Business logic tested: correct database changes made
[ ] Provider dashboard used to verify test events were received and acknowledged
```

---

## Done When

```text
[ ] Signature verification is the first step
[ ] Signature secret stored in environment variable — not hardcoded
[ ] Idempotency implemented (event_id stored and checked)
[ ] Processing logic separated from signature verification
[ ] Returns 200 quickly — no slow blocking operations
[ ] All events logged with status
[ ] All test cases pass
[ ] Endpoint URL registered with the external service
```
