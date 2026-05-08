# Checklist — Supabase Storage Bucket

> Run this checklist every time you create a new Storage bucket or add file upload functionality.

**Standard:** [Supabase Standards — Multi-Tenant & Storage](../../standards/supabase/05-validation-multitenant-storage.md)

---

## 1. Bucket Configuration

```text
[ ] Bucket created with a clear kebab-case name: avatars, project-files, invoices
[ ] Bucket visibility decided:
    [ ] Public bucket: files accessible via URL without auth (only for truly public assets)
    [ ] Private bucket: all access goes through RLS policies or signed URLs
[ ] Allowed MIME types restricted in bucket config where possible
[ ] Max file size limit set in bucket config (or enforced in RLS / frontend)
```

---

## 2. RLS Policies on Storage

Supabase Storage uses RLS on the `storage.objects` table.

```text
[ ] RLS policies defined for storage.objects
[ ] Separate policies for: SELECT (read), INSERT (upload), UPDATE (replace), DELETE (delete)

COMMON PATTERNS:
[ ] User can upload to their own folder: storage.foldername(name) = auth.uid()::text
[ ] Workspace members can access workspace files: ownership verified via workspace_members lookup
[ ] Public read: allow SELECT with no auth check (for public buckets only)
[ ] Owner-only delete: auth.uid() matches uploader's user_id stored in metadata or path
```

---

## 3. File Path Convention

```text
[ ] Paths follow a consistent structure:
    — User files:      {user_id}/{filename}
    — Workspace files: {workspace_id}/{entity_type}/{entity_id}/{filename}
    — Public assets:   public/{filename}
[ ] Filenames sanitized: no spaces, no special characters (use UUID or slugified name)
[ ] Original filename stored separately (in a database column) for display purposes
[ ] Never use predictable sequential IDs in paths (use UUIDs to prevent enumeration)
```

---

## 4. Upload Validation (Frontend)

```text
[ ] Accepted file types shown to user: "JPG, PNG, or PDF up to 5 MB"
[ ] File type validated by MIME type — not just file extension
[ ] File size validated before upload begins
[ ] Progress bar shown for uploads > 1 second
[ ] Upload can be cancelled
[ ] On success: URL stored in the relevant database record
[ ] On failure: error shown to user, no partial record saved
```

---

## 5. Upload Validation (Backend / RLS)

```text
[ ] RLS INSERT policy checks: authenticated, correct workspace/user context
[ ] File size enforced by Supabase bucket config or by an Edge Function
[ ] MIME type enforced by Supabase bucket config (allowedMimeTypes)
[ ] Virus scan considered for user-uploaded documents (Edge Function with external scanner)
[ ] EXIF metadata stripped from images where user privacy matters
```

---

## 6. Serving Files

```text
[ ] Private files: served via signed URL with expiry — not stored as a public URL
[ ] Signed URL expiry appropriate for use case:
    — Temporary download: 60 seconds
    — Short-lived preview: 5 minutes
    — Long-lived link (e.g., invoice): 7 days
[ ] Signed URL generated at request time — not stored in the database
[ ] Public files: permanent public URL stored in DB is acceptable only if truly public
[ ] CDN caching considered for frequently accessed public assets
```

---

## 7. File Deletion

```text
[ ] File deleted from Storage when the owning record is deleted
[ ] Not relying on CASCADE alone — Storage does not cascade with DB deletes
[ ] Delete handled in a transaction or Edge Function: delete DB record → delete Storage file
[ ] Orphaned files: plan for how they are discovered and cleaned up
[ ] User "remove" action: deletes both the record reference and the Storage file
```

---

## 8. Security Rules

```text
[ ] Service role key is NEVER in frontend code
[ ] All uploads from frontend use the anon key + RLS policies
[ ] Storage bucket not set to public unless the files are genuinely public
[ ] Path structure does not allow one user to access another user's files
[ ] Predictable file paths avoided (no /user/1/avatar.jpg — use /user/{uuid}/avatar.jpg)
[ ] No sensitive data in the file path (no SSNs, no email addresses in path)
```

---

## 9. Documentation

```text
[ ] Bucket name and purpose documented
[ ] Allowed file types and max size documented
[ ] Path structure documented with examples
[ ] RLS policies documented (who can read/write/delete)
[ ] Signed URL expiry values documented
```

---

## Done When

```text
[ ] Bucket created with correct visibility (public/private)
[ ] RLS policies cover all operations (SELECT, INSERT, DELETE)
[ ] File paths follow the defined structure
[ ] Frontend validates type + size before upload
[ ] Signed URLs used for private files (not stored public URLs)
[ ] File deletion cleans up Storage too (not just the DB record)
[ ] Security rules verified — no cross-user access possible
[ ] Bucket and policies documented
```

---

## Practice Task

**→ [Storage Setup Task](../../tasks/supabase/06-storage-setup-task.md)**
Create a storage bucket for project attachments, set RLS policies, and design the upload/download/delete flow with signed URLs.
