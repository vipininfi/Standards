# Backend Developer Checklist

Run this before marking any API, table, or backend feature as complete.

---

## Database & Schema

```text
[ ] Migration created (not dashboard-only change)
[ ] Table has UUID primary key
[ ] Table has created_at and updated_at timestamps
[ ] Table has created_by (user UUID) where ownership matters
[ ] Table has workspace_id / organization_id for multi-tenant apps
[ ] Table has deleted_at for soft delete if needed
[ ] All foreign keys defined with correct on delete behavior
[ ] Constraints added (not null, unique, check)
[ ] Indexes added (workspace_id, created_by, status, created_at, FK columns)
[ ] Schema reviewed and naming follows snake_case plural convention
```

## RLS

```text
[ ] RLS enabled on table
[ ] SELECT policy added
[ ] INSERT policy added
[ ] UPDATE policy added
[ ] DELETE policy added (or intentionally blocked)
[ ] No policy uses "using (true)" without intent
[ ] No policy uses "with check (true)" without intent
[ ] Auth.uid() used for ownership checks
[ ] Workspace membership table used for team/workspace checks
[ ] Policies tested as: owner / admin / manager / member / viewer / non-member / anonymous
```

## API / Function / Edge Function

```text
[ ] API type chosen correctly (direct query / RPC / Edge Function)
[ ] Auth validation implemented
[ ] Role/permission check implemented
[ ] Input payload validation implemented (all fields)
[ ] Validation at three layers: frontend hint + API check + database constraint
[ ] Error responses are structured and consistent
[ ] No service_role key used in frontend-facing code
[ ] Logs added for important actions
[ ] Audit logs added for sensitive actions (if required)
```

## For Edge Functions

```text
[ ] CORS handling added
[ ] Method validation added (OPTIONS handled)
[ ] JWT / auth header validated
[ ] Payload parsed and validated
[ ] Permission checked
[ ] Structured error response returned
[ ] Structured logs added
[ ] Webhook signature verified (if webhook)
[ ] Idempotency handled (if webhook — no duplicate processing)
```

## Documentation

```text
[ ] API name and purpose documented
[ ] Auth requirement documented
[ ] Required role documented
[ ] Request payload documented (all fields, types, rules)
[ ] Success response documented with example
[ ] All error responses documented (401, 403, 422, 409, 500, etc.)
[ ] RLS policies involved documented
[ ] Frontend implementation notes provided
[ ] Frontend sample code provided
[ ] Test cases listed
```

## Testing

```text
[ ] Authenticated success tested
[ ] Anonymous request tested
[ ] Expired token tested
[ ] Wrong role tested
[ ] Wrong workspace tested
[ ] Missing required field tested
[ ] Invalid field type tested
[ ] Duplicate record tested (if unique constraint exists)
[ ] RLS denial tested
[ ] External service failure tested (if applicable)
[ ] Staging verified before marking done
```

## Storage (if files are involved)

```text
[ ] Bucket created (public vs private separated)
[ ] Storage RLS policy added
[ ] File path follows workspace/{workspace_id}/... pattern
[ ] File size validated
[ ] MIME type validated
[ ] Signed URLs used for private files
```

---

## The Core Question

Before marking any API done, you must be able to answer clearly:

```text
Who is calling this?
What are they allowed to access?
How does the database enforce that?
What exact payload is allowed?
What exact error is returned when something fails?
What should frontend do in every case?
```

If any answer is unclear, the API is not production-ready.
