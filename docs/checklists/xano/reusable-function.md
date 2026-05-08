# Checklist — Xano Reusable Function

> Run this checklist every time you extract logic into a Xano Custom Function.

**Standard:** [Xano Standards](../../standards/xano-standards.md)

---

## 1. Decision — Should This Be a Reusable Function?

```text
EXTRACT TO A REUSABLE FUNCTION WHEN:
[ ] The same logic is used (or will be used) in 2 or more endpoints
[ ] The logic is complex enough to be worth isolating and testing independently
[ ] The logic represents a coherent unit: "check workspace permission", "format user object"

DO NOT EXTRACT WHEN:
[ ] It is only used once and is simple
[ ] It is one or two steps that read clearly inline
[ ] Extracting would require too many inputs, making it harder to understand
```

---

## 2. Naming

```text
[ ] Function name describes exactly what it does: verb_noun pattern
    check_workspace_permission, format_user_response, validate_invite_token
[ ] No vague names: not "helper", not "utility", not "process_data"
[ ] Name matches what it returns or does — not what calls it
```

---

## 3. Inputs

```text
[ ] All inputs explicitly named and typed
[ ] No implicit global access — all required data is passed as an input
[ ] Input names match the convention of the calling endpoint: workspace_id not ws
[ ] Optional inputs have default values documented
[ ] No inputs for data the function can derive internally
```

---

## 4. Function Logic

```text
[ ] Function does ONE thing — single responsibility
[ ] No auth token validation inside a reusable function (auth is the endpoint's first step)
[ ] Validation inside the function only if it is always needed when called
[ ] Error raised with the correct error format if validation fails:
    { "code": "FORBIDDEN", "message": "You are not a member of this workspace." }
[ ] No hardcoded IDs or environment-specific values inside the function
[ ] No external API calls inside (put those in the calling endpoint)
```

---

## 5. Outputs

```text
[ ] Output is typed and documented
[ ] Output shape is consistent — not different shapes depending on code path
[ ] On error: function raises an exception — does not return null or an empty object
[ ] Calling endpoints handle the output consistently (not each endpoint doing its own transformation)
```

---

## 6. Testing

```text
[ ] Function tested directly in Xano with the correct inputs
[ ] Success case: correct input → correct output
[ ] Error case: invalid input → correct error code and message raised
[ ] All calling endpoints re-tested after function is extracted
[ ] No regression in existing endpoints after extraction
```

---

## 7. Documentation

```text
[ ] Function purpose documented (one sentence)
[ ] All inputs listed: name, type, required/optional
[ ] Output documented with example
[ ] Error cases documented: what raises, what error code
[ ] List of endpoints that call this function
```

---

## Done When

```text
[ ] Function extracted and named correctly
[ ] All inputs explicitly typed
[ ] Single responsibility — does one thing
[ ] Error handling is consistent with the rest of the codebase
[ ] All calling endpoints work after extraction
[ ] Function documented
```

---

## Practice Task

**→ [Reusable Function Task](../../tasks/xano/02-reusable-function.md)**
Extract a `check_workspace_permission` reusable function used across multiple endpoints, with role hierarchy logic and correct error raising.
