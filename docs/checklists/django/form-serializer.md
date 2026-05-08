# Checklist — Django Form / DRF Serializer

> Run this checklist for every Django Form (HTML forms) or DRF Serializer (API endpoints).

**Standard:** [Backend-First Logic Standard](../../standards/backend-first-logic.md)

---

## 1. Form vs Serializer Decision

```text
USE A DJANGO FORM WHEN:
[ ] The endpoint renders an HTML template (not a JSON API)
[ ] The form processes HTML form submissions (multipart/form-data or application/x-www-form-urlencoded)

USE A DRF SERIALIZER WHEN:
[ ] The endpoint returns JSON (REST API)
[ ] The request body is JSON
[ ] The response must include nested related data
[ ] You need read/write control at the field level (read_only_fields)
```

---

## 2. Fields

```text
[ ] All required fields are listed explicitly — do not rely on model defaults for validation
[ ] Field types match the data: CharField, IntegerField, EmailField, DateField, etc.
[ ] Optional fields marked with required=False (or allow_null=True, allow_blank=True for DRF)
[ ] Choices: validated against a choices list — not accepting free text for enum fields
[ ] Email fields: use EmailField (validates format automatically)
[ ] URL fields: use URLField (validates format automatically)
[ ] Integer fields: use IntegerField with min_value/max_value where appropriate
[ ] File fields: validate MIME type — not just extension
```

---

## 3. Validation

```text
FIELD-LEVEL VALIDATION
[ ] Specific validate_<field> methods for custom field rules
[ ] E.g., def validate_name(self, value): check length, format, allowed characters
[ ] Returns cleaned value — not the raw input
[ ] Raises ValidationError with a specific message (not a generic "Invalid.")

FORM/OBJECT-LEVEL VALIDATION
[ ] Cross-field rules in clean() (Django Form) or validate() (DRF Serializer)
[ ] E.g., end_date must be after start_date
[ ] Uniqueness checks that involve multiple fields done here or in the service
[ ] Raises ValidationError with a non-field error key

WHAT NOT TO DO
× Do not put business logic in the form/serializer (e.g., creating related records)
× Do not query the database inside validate_<field> unless necessary for uniqueness
× Do not call external APIs inside validation methods
× Validation purpose: ensure data is structurally correct — business logic goes in services.py
```

---

## 4. Error Handling

```text
[ ] ValidationError raised with a specific, user-readable message
[ ] No generic "Invalid value." messages — always say what is wrong
[ ] Field errors attached to the correct field key (not all in non_field_errors)
[ ] DRF: serializer.errors returns a dict matching field names — used directly in the response
[ ] Django Form: form.errors rendered in the template next to each field
```

---

## 5. DRF Serializer Specific

```text
[ ] read_only_fields set for fields that should never be written from the API:
    id, created_at, updated_at, created_by
[ ] write_only_fields (or write_only=True per field) set for fields never returned:
    password, token, secret
[ ] Nested serializers: source field set correctly
[ ] SerializerMethodField used for computed fields that are not model fields
[ ] Meta.model and Meta.fields defined explicitly
[ ] NEVER use fields = '__all__' in production (exposes all model fields — security risk)
```

---

## 6. Django Form Meta

```text
[ ] ModelForm: class Meta.model and class Meta.fields defined
[ ] Never use fields = '__all__'
[ ] widgets defined for fields that need a custom widget (date picker, etc.)
[ ] labels overridden if Django's auto-generated label is not user-friendly
[ ] help_texts added for fields that need explanation
```

---

## 7. Testing

```text
[ ] Valid data: form/serializer is_valid() → True, cleaned data is correct
[ ] Missing required field: is_valid() → False, correct field error
[ ] Invalid format (bad email, wrong type): is_valid() → False, correct field error
[ ] Cross-field error: is_valid() → False, error on the correct non-field key
[ ] Boundary values: max_length - 1 (valid), max_length + 1 (invalid)
[ ] DRF: response data shape tested — correct fields included/excluded
```

---

## Done When

```text
[ ] All required fields defined with correct types
[ ] All optional fields marked as optional
[ ] Custom validate_<field> methods for each field requiring custom rules
[ ] Cross-field validation in clean() / validate()
[ ] Specific error messages for all failure cases
[ ] DRF: read_only and write_only fields set correctly
[ ] Never uses fields = '__all__'
[ ] All validation cases tested
```
