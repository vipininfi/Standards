# Checklist — Supabase RPC Function

> Run this checklist every time you write a new Postgres/RPC function exposed via `supabase.rpc()`.

**Standard:** [RPC & Postgres Function Standards](../../standards/supabase/07-rpc-functions.md)

---

## 1. Decision

```text
[ ] Confirmed RPC is the right choice (multi-step, business logic, or atomic operation)
[ ] Not using RPC for simple single-table CRUD (use direct table query instead)
[ ] Not using RPC when an Edge Function is needed (secret key, third-party API, email)
```

---

## 2. SECURITY INVOKER vs SECURITY DEFINER

```text
[ ] Default is SECURITY INVOKER (RLS applies) — used unless documented reason to bypass
[ ] SECURITY DEFINER only if explicitly justified in a comment above the function
[ ] If SECURITY DEFINER:
    [ ] auth.uid() captured as the FIRST line of the DECLARE block
    [ ] auth.uid() validated: IF v_user_id IS NULL THEN RAISE EXCEPTION 'AUTH_REQUIRED...'
    [ ] Manual permission checks replace RLS checks
    [ ] Reviewed by a second developer before merging
```

---

## 3. Function Signature

```text
[ ] Function name: snake_case verb_noun pattern (create_project, delete_member)
[ ] All parameters prefixed with p_: p_workspace_id, p_name
[ ] Optional parameters use DEFAULT NULL
[ ] Parameter types are correct:
    [ ] UUIDs typed as uuid (not text)
    [ ] Timestamps typed as timestamptz (not timestamp)
    [ ] Dates typed as date (not text)
    [ ] Booleans typed as boolean (not text or integer)
    [ ] Arrays typed correctly: uuid[], text[]
[ ] Internal variables prefixed with v_: v_project, v_count
[ ] No single-letter variable names
```

---

## 4. Return Type

```text
[ ] Return type chosen and documented:
    [ ] RETURNS json — single record or mixed data
    [ ] RETURNS TABLE — multiple rows with known columns
    [ ] RETURNS void — side effects only
[ ] Success response shape follows standard:
    json_build_object('data', row_to_json(v_record), 'message', '...')
[ ] Never returning error codes inside the data — errors use RAISE EXCEPTION
```

---

## 5. Validation Order

All validation done in this order — stop at first failure:

```text
[ ] 1. Auth check: v_user_id IS NULL → RAISE EXCEPTION 'AUTH_REQUIRED: ...'
[ ] 2. Required field checks: p_name IS NULL OR trim(p_name) = ''
[ ] 3. Length checks: length(trim(p_name)) > 100
[ ] 4. Format / enum checks: p_status NOT IN ('active', 'on-hold', 'archived')
[ ] 5. Cross-field checks: p_end_date <= p_start_date
[ ] 6. Existence checks (FK): workspace not found → NOT_FOUND
[ ] 7. Permission checks: not a member → FORBIDDEN; wrong role → FORBIDDEN
[ ] 8. Business rule checks: duplicate name, limit exceeded → DUPLICATE / BUSINESS_RULE_VIOLATION
```

---

## 6. Error Raising

```text
[ ] All RAISE EXCEPTION follow the format: 'ERROR_CODE: description'
[ ] Error codes used:
    [ ] AUTH_REQUIRED — user not authenticated
    [ ] VALIDATION_ERROR — field-level validation failure
    [ ] NOT_FOUND — FK target or entity does not exist
    [ ] FORBIDDEN — not a member or wrong role
    [ ] DUPLICATE — uniqueness violation
    [ ] BUSINESS_RULE_VIOLATION — business constraint violated
    [ ] SERVER_ERROR — unexpected/catch-all error
[ ] No raw Postgres error messages exposed (wrap in EXCEPTION WHEN OTHERS)
```

---

## 7. Transaction Safety

```text
[ ] Multi-step operations (inserts, updates, audit log) all inside one function
[ ] No COMMIT or ROLLBACK called manually — Postgres handles this
[ ] EXCEPTION WHEN OTHERS block at the end to catch and re-raise as SERVER_ERROR
[ ] If one step fails, earlier steps are automatically rolled back
```

---

## 8. Performance

```text
[ ] SELECT * not used inside function — select only needed columns
[ ] FOUND checked after a SELECT INTO (not checking the variable itself)
[ ] No N+1 queries inside a loop — use JOIN or IN clause
[ ] LIMIT used on any query that could return unbounded rows
[ ] Indexes exist for all WHERE columns used in the function
[ ] Read-only functions marked as STABLE
```

---

## 9. Documentation

```text
[ ] Function documented using the API Documentation Template
[ ] All parameters listed with type, required/optional, constraints
[ ] Success response JSON example included
[ ] All RAISE EXCEPTION codes listed with frontend handling example
[ ] Frontend call example: supabase.rpc('function_name', { p_param: value })
```

---

## 10. Testing

```text
[ ] Success case tested: correct inputs → correct data returned
[ ] Missing required field: VALIDATION_ERROR raised
[ ] Invalid format: VALIDATION_ERROR raised
[ ] Non-existent FK: NOT_FOUND raised
[ ] Non-member caller: FORBIDDEN raised
[ ] Wrong role: FORBIDDEN raised
[ ] Duplicate record: DUPLICATE raised
[ ] All side effects verified (related records, audit log)
[ ] Transaction atomicity: simulate mid-function failure → earlier steps rolled back
[ ] RLS still enforced (SECURITY INVOKER): cross-workspace access blocked
```

---

## Done When

```text
[ ] Function uses correct SECURITY mode with justification documented
[ ] All parameters correctly typed and prefixed
[ ] Validation in correct order, all failure cases handled
[ ] Errors follow the ERROR_CODE: description format
[ ] Multi-step operations are atomic (single function)
[ ] Performance rules followed (no SELECT *, no N+1)
[ ] Fully documented for frontend handoff
[ ] All test cases pass
```

---

## Practice Task

**→ [Archive Project RPC Function](../../tasks/supabase/05-rpc-function-task.md)**
Build an atomic RPC that archives a project, sets all open tasks to on-hold, and writes an audit log — in one transaction, with full validation.
