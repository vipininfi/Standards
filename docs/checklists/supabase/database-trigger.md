# Checklist — Supabase Database Trigger

> Run this checklist every time you create a database trigger or trigger function.

---

## 1. Decision — Should This Be a Trigger?

```text
USE A TRIGGER WHEN:
[ ] An action must always happen after a database event — no exceptions
[ ] The action is too low-level to put in application code (audit logging, timestamps)
[ ] The action must apply even if data is modified by a migration script or admin

DO NOT USE A TRIGGER WHEN:
[ ] The action involves external services (email, webhooks) — use an Edge Function
[ ] The logic is complex business logic — put it in an RPC function instead
[ ] The trigger would create circular dependencies
[ ] The trigger would cause N+1 database calls
```

---

## 2. Trigger Function

```text
[ ] Trigger function uses RETURNS TRIGGER
[ ] Named clearly: trigger_fn_[action]: trigger_fn_set_timestamps, trigger_fn_audit_log
[ ] LANGUAGE plpgsql (not SQL for complex logic)
[ ] Function handles both OLD and NEW records correctly for UPDATE triggers
[ ] Returns NEW for BEFORE triggers (modify the row before write)
[ ] Returns NULL to cancel the operation (intentional in BEFORE trigger only)
[ ] Returns NEW for AFTER triggers (return value ignored but convention)
[ ] Error raised with RAISE EXCEPTION if trigger validation fails (BEFORE triggers only)
```

---

## 3. Trigger Definition

```text
[ ] BEFORE or AFTER chosen correctly:
    — BEFORE: to validate or modify the row before it is written
    — AFTER:  to react to a change (create related records, audit log)
[ ] FOR EACH ROW (almost always) vs FOR EACH STATEMENT (bulk operations)
[ ] Trigger event: INSERT, UPDATE, DELETE, or combination
[ ] For UPDATE triggers: OF column_list used when only specific column changes matter
    (avoids trigger firing on every unrelated update)
[ ] Trigger attached to the correct table
[ ] Conditional trigger: WHEN clause used to skip unnecessary invocations
```

---

## 4. Common Trigger Patterns

### Automatic Timestamps

```sql
CREATE OR REPLACE FUNCTION trigger_fn_set_timestamps()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    NEW.created_at := now();
  END IF;
  NEW.updated_at := now();
  RETURN NEW;
END;
$$;

CREATE TRIGGER set_timestamps
BEFORE INSERT OR UPDATE ON projects
FOR EACH ROW EXECUTE FUNCTION trigger_fn_set_timestamps();
```

```text
[ ] created_at only set on INSERT (not overwritten on UPDATE)
[ ] updated_at set on both INSERT and UPDATE
[ ] Uses now() for consistency
```

### Audit Log

```sql
CREATE OR REPLACE FUNCTION trigger_fn_audit_log()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER  -- needs to write audit_logs even if RLS blocks the user
AS $$
BEGIN
  INSERT INTO audit_logs (action, table_name, record_id, old_data, new_data, actor_id)
  VALUES (
    TG_OP,
    TG_TABLE_NAME,
    COALESCE(NEW.id, OLD.id),
    CASE WHEN TG_OP = 'DELETE' THEN row_to_json(OLD) ELSE NULL END,
    CASE WHEN TG_OP != 'DELETE' THEN row_to_json(NEW) ELSE NULL END,
    auth.uid()
  );
  RETURN NEW;
END;
$$;
```

```text
[ ] Stores both old and new values for UPDATE
[ ] Stores actor_id (auth.uid()) — who made the change
[ ] SECURITY DEFINER only for audit_log insert — justified by need to bypass RLS
[ ] Does not log sensitive fields (passwords, tokens, PII unless required by compliance)
```

---

## 5. Performance Rules

```text
[ ] Trigger does not perform N+1 queries (no loop with individual queries)
[ ] Trigger function is fast — no heavy computation or external calls
[ ] For UPDATE triggers: WHEN clause limits when the trigger fires
[ ] Trigger does not call other triggers in a chain
[ ] Indexes exist for any columns the trigger queries
```

---

## 6. Testing

```text
[ ] INSERT: trigger fires correctly
[ ] UPDATE: trigger fires on relevant column changes; does not fire on irrelevant columns
[ ] DELETE: trigger fires correctly; OLD record data is correct
[ ] BEFORE trigger: validation correctly cancels invalid writes
[ ] AFTER trigger: related records created correctly after the write
[ ] Failure in trigger: entire transaction rolled back (including the original write)
[ ] Performance: trigger does not add significant latency to high-volume operations
```

---

## 7. Documentation

```text
[ ] Trigger name and purpose documented
[ ] Which table and which events (INSERT, UPDATE, DELETE) it listens to
[ ] What the trigger does (one paragraph)
[ ] Any SECURITY DEFINER usage justified in a comment above the function
[ ] Listed in the migration file that creates it
```

---

## Done When

```text
[ ] Trigger fires on the correct event(s) and table
[ ] BEFORE vs AFTER chosen correctly for the use case
[ ] Trigger function is correct and fast
[ ] SECURITY DEFINER justified if used
[ ] All test cases pass (INSERT, UPDATE, DELETE)
[ ] Migration file includes both trigger function and trigger creation
[ ] Trigger documented
```
