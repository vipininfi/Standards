# Supabase — Edge Function Checklist

> **Core Rule:** Edge Functions are for server-side logic that cannot or should not run in the browser — webhooks, external API calls, payment processing, email sending, and any logic that requires secrets. They are not a replacement for simple table queries or RPCs.

---

## Contents

| # | Section |
|---|---------|
| [1](#1-when-to-use-an-edge-function) | When to Use an Edge Function |
| [2](#2-edge-function-structure) | Edge Function Structure |
| [3](#3-auth-pattern) | Auth Pattern |
| [4](#4-request-validation) | Request Validation |
| [5](#5-response-standard) | Response Standard |
| [6](#6-secrets--environment-variables) | Secrets & Environment Variables |
| [7](#7-error-handling--logging) | Error Handling & Logging |
| [8](#8-webhook-handling) | Webhook Handling |
| [9](#9-edge-function-checklist--before-marking-done) | Edge Function Checklist — Before Marking Done |

---

# 1. When to Use an Edge Function

Use an Edge Function when:

```text
[ ] Sending email (Resend, SendGrid, etc.)
[ ] Processing payments (Stripe, etc.)
[ ] Calling a third-party API with a secret key
[ ] Handling webhooks from external services
[ ] Running logic that requires secrets not safe to expose
[ ] Complex multi-step operations not suited for RPC
[ ] Background jobs triggered by HTTP
```

Do NOT use an Edge Function when:

```text
× Simple table query — use supabase.from() with RLS
× Business logic without secrets — use RPC / Postgres Function
× Just to "wrap" a table query for no reason
× The logic could be enforced at the database level (constraints, triggers)
```

---

# 2. Edge Function Structure

Every Edge Function must follow this structure:

```ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};

serve(async (req: Request) => {
  // 1. Handle CORS preflight
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders });
  }

  try {
    // 2. Auth check (if protected)
    // 3. Parse and validate input
    // 4. Execute business logic
    // 5. Return success response

    return new Response(
      JSON.stringify({ data: result, message: 'Success.' }),
      { headers: { ...corsHeaders, 'Content-Type': 'application/json' }, status: 200 }
    );

  } catch (error) {
    // 6. Handle errors
    console.error('[function-name] Error:', error.message);

    return new Response(
      JSON.stringify({ error: 'SERVER_ERROR', message: 'An unexpected error occurred.' }),
      { headers: { ...corsHeaders, 'Content-Type': 'application/json' }, status: 500 }
    );
  }
});
```

---

# 3. Auth Pattern

```ts
// For protected Edge Functions — validate auth token first
const authHeader = req.headers.get('Authorization');
if (!authHeader) {
  return new Response(
    JSON.stringify({ error: 'AUTH_REQUIRED', message: 'Authentication required.' }),
    { headers: { ...corsHeaders, 'Content-Type': 'application/json' }, status: 401 }
  );
}

// Create client with user's token (not service role)
const supabase = createClient(
  Deno.env.get('SUPABASE_URL') ?? '',
  Deno.env.get('SUPABASE_ANON_KEY') ?? '',
  { global: { headers: { Authorization: authHeader } } }
);

// Verify token and get user
const { data: { user }, error: authError } = await supabase.auth.getUser();
if (authError || !user) {
  return new Response(
    JSON.stringify({ error: 'AUTH_REQUIRED', message: 'Invalid or expired token.' }),
    { headers: { ...corsHeaders, 'Content-Type': 'application/json' }, status: 401 }
  );
}
```

```text
[ ] Auth token validated at the start of every protected function
[ ] User ID extracted from validated JWT — not from request body
[ ] Role/permissions read from database — never trusted from request body
[ ] Public functions (e.g., contact form) confirmed intentionally public
```

---

# 4. Request Validation

```ts
// Parse body
const body = await req.json().catch(() => null);
if (!body) {
  return new Response(
    JSON.stringify({ error: 'VALIDATION_ERROR', message: 'Invalid request body.' }),
    { status: 400, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
  );
}

// Validate required fields
const errors: Record<string, string> = {};
if (!body.name || typeof body.name !== 'string' || body.name.trim() === '') {
  errors.name = 'Name is required.';
}
if (!body.email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(body.email)) {
  errors.email = 'Enter a valid email address.';
}

if (Object.keys(errors).length > 0) {
  return new Response(
    JSON.stringify({ error: 'VALIDATION_ERROR', message: 'Validation failed.', fields: errors }),
    { status: 400, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
  );
}
```

```text
[ ] req.json() wrapped in try/catch — invalid JSON returns 400
[ ] Every required field validated (not null, not empty)
[ ] String lengths validated (max characters enforced)
[ ] Format validated (email format, UUID format, enum values)
[ ] Numeric ranges validated
[ ] Field-level errors returned with VALIDATION_ERROR code
```

---

# 5. Response Standard

All responses must follow this format:

## Success

```json
{
  "data": { ... },
  "message": "Descriptive success message."
}
```

## Error

```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable description."
}
```

## Validation Error (with field errors)

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Validation failed.",
  "fields": {
    "email": "Enter a valid email address.",
    "name": "Name is required."
  }
}
```

```text
[ ] All responses include Content-Type: application/json
[ ] All responses include CORS headers
[ ] Success: status 200 or 201
[ ] Validation error: status 400
[ ] Auth error: status 401
[ ] Forbidden: status 403
[ ] Not found: status 404
[ ] Server error: status 500
[ ] No stack traces or internal error details in response body
```

---

# 6. Secrets & Environment Variables

```text
[ ] All API keys accessed via Deno.env.get('KEY_NAME')
[ ] No secrets hardcoded in function code
[ ] No secrets committed to git
[ ] Secrets set in Supabase Dashboard → Edge Functions → Secrets
[ ] Service role key used ONLY when needed (and only in server-side functions)
[ ] Supabase anon key used for user-scoped operations (respects RLS)
[ ] Third-party API keys (Stripe, Resend, etc.) stored as secrets
[ ] Environment variable names follow: UPPERCASE_SNAKE_CASE
[ ] Local development uses .env.local — not committed to git
```

---

# 7. Error Handling & Logging

## What to Log

```text
[ ] Function name included in every log: console.log('[function-name] ...')
[ ] Request received: log action type, NOT sensitive data (no passwords, tokens, card numbers)
[ ] Success: log what was done (e.g., "Email sent to user-id: xxx")
[ ] Failure: log error message + relevant non-sensitive context
[ ] External API failures: log status code and error message from provider
```

## What NOT to Log

```text
× Passwords or tokens
× Full credit card numbers
× Personal data beyond what is needed for debugging (e.g., log user ID, not full name)
× Full request body if it contains sensitive fields
× Supabase service role key or other secrets
```

## Error Pattern

```ts
try {
  // business logic
} catch (error) {
  // Log for debugging
  console.error(`[function-name] Unexpected error: ${error.message}`);

  // Never expose internal details to client
  return new Response(
    JSON.stringify({ error: 'SERVER_ERROR', message: 'An unexpected error occurred.' }),
    { status: 500, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
  );
}
```

---

# 8. Webhook Handling

For Edge Functions that receive webhooks from external services:

```text
[ ] Webhook signature verified before processing payload
[ ] Signature secret stored as an environment variable — never hardcoded
[ ] Idempotency check: if the same event arrives twice, only process it once
  — Store event_id in a processed_webhooks table
  — Check for duplicate before processing
[ ] Respond with 200 immediately if only acknowledgement is needed (queue async work)
[ ] Never trust payload claims without signature verification
[ ] Webhook secret rotated if ever exposed
```

## Signature Verification Example (Stripe)

```ts
import Stripe from 'https://esm.sh/stripe@12';

const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY') ?? '', {
  apiVersion: '2023-10-16',
});

const signature = req.headers.get('stripe-signature') ?? '';
const body = await req.text();

let event: Stripe.Event;
try {
  event = stripe.webhooks.constructEvent(
    body,
    signature,
    Deno.env.get('STRIPE_WEBHOOK_SECRET') ?? ''
  );
} catch (err) {
  return new Response(
    JSON.stringify({ error: 'INVALID_SIGNATURE', message: 'Webhook signature verification failed.' }),
    { status: 400 }
  );
}
```

---

# 9. Edge Function Checklist — Before Marking Done

```text
JUSTIFICATION
[ ] Edge Function is needed — not a simpler table query or RPC
[ ] Reason documented (requires secrets / webhook / third-party API)

STRUCTURE
[ ] CORS OPTIONS handler at the top
[ ] Try/catch wraps entire logic
[ ] All responses include CORS headers + Content-Type: application/json

AUTH
[ ] Protected functions validate auth token at the start
[ ] Public functions confirmed intentionally public
[ ] User ID from validated JWT — not from request body
[ ] Role from database — not from request body

INPUT VALIDATION
[ ] req.json() wrapped in try/catch
[ ] Required fields validated
[ ] Field-level errors returned for validation failures
[ ] String lengths validated
[ ] Format/enum values validated

RESPONSE FORMAT
[ ] Success: { data, message } with 200/201
[ ] Error: { error, message } with correct status
[ ] Validation: { error, message, fields } with 400
[ ] No stack traces or secrets in response

SECRETS
[ ] All secrets via Deno.env.get()
[ ] No secrets hardcoded
[ ] Secrets added to Supabase Dashboard (not just .env.local)

ERROR HANDLING
[ ] All errors caught and returned with correct status
[ ] Logging includes function name, action, and error details
[ ] No sensitive data in logs

WEBHOOKS (if applicable)
[ ] Signature verified before processing
[ ] Idempotency check implemented
[ ] Returns 200 quickly (async processing if needed)

TESTING
[ ] Success case tested
[ ] Invalid input tested
[ ] Auth failure tested
[ ] Webhook signature rejection tested (if webhook)
[ ] Staged on staging before production
[ ] Secrets confirmed set in the correct Supabase project/environment
```

---

## Practice Task

Apply what you learned by building a real Edge Function that calls a third-party API and handles every error case.

**→ [Task 04: Build a Project Invite Edge Function](../../tasks/supabase/04-invite-edge-function.md)**

Covers: full Edge Function structure (CORS/auth/validation/logic/response), Deno.env secrets, JWT validation, permission check, duplicate check, DB insert via service role, email via Resend, structured logging without sensitive data.
