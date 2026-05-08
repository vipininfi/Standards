# Task — Storage Bucket: Project Attachments

**Platform:** Supabase Storage
**Checklist to Run:** [Storage Bucket Checklist](../../checklists/supabase/storage-bucket.md)
**Standard:** [Supabase Standards — Multi-Tenant & Storage](../../standards/supabase/05-validation-multitenant-storage.md)

---

## Scenario

The app allows users to upload files to tasks within projects. Each file upload is attached to a specific task. Users should only be able to see and download files for tasks in workspaces they are a member of. File uploads are managed by users — they can upload and delete their own files. Workspace admins can delete any file in the workspace.

---

## What to Build

1. A Supabase Storage bucket named `project-attachments` (private)
2. RLS policies on `storage.objects` for the bucket
3. A `task_attachments` table to track file metadata
4. The upload flow, download flow, and delete flow (describe each — implementation optional)

---

## Requirements

### Bucket

- Name: `project-attachments`
- Visibility: **Private** (all access via RLS or signed URLs)
- Allowed MIME types: `image/jpeg`, `image/png`, `application/pdf`, `application/msword`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`
- Max file size: 10 MB (enforce in bucket config)

### File Path Structure

```
{workspace_id}/{project_id}/{task_id}/{uuid}-{sanitized_filename}
```

Example: `a1b2c3d4.../e5f6g7h8.../i9j0k1l2.../550e8400-marketing-brief.pdf`

- UUID prefix prevents filename collisions
- Sanitized filename: lowercase, spaces replaced with hyphens, no special characters

### `task_attachments` Table

| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| task_id | uuid | FK → tasks.id, ON DELETE CASCADE |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE (for RLS) |
| uploaded_by | uuid | FK → auth.users.id |
| storage_path | text | Full path in the bucket |
| filename | text | Original filename for display |
| file_size_bytes | integer | Size in bytes |
| mime_type | text | e.g., 'application/pdf' |
| created_at | timestamptz | default now() |

### RLS Policies on `storage.objects`

**SELECT (download):**
User must be a member of the workspace matching the first segment of the file path.

**INSERT (upload):**
User must be authenticated and a member of the workspace in the first path segment.

**DELETE:**
User uploaded the file (uploaded_by = auth.uid()) OR user is admin/owner of the workspace.

### Frontend Upload Flow

1. User selects a file — validate type and size before upload begins
2. Show upload progress bar
3. On upload success: insert a row into `task_attachments` with the storage path
4. Show the file in the task's attachment list
5. On upload error: show the error, do not insert into `task_attachments`

### Frontend Download Flow

1. User clicks "Download" on an attachment
2. Frontend calls `supabase.storage.from('project-attachments').createSignedUrl(path, 300)` (5-minute expiry)
3. Open the signed URL in a new tab
4. Never store the signed URL in the database

### File Deletion Flow

1. User clicks "Delete" on an attachment
2. Confirmation dialog: "Delete [filename]? This cannot be undone."
3. On confirm: delete the `task_attachments` row AND delete the file from Storage
4. If Storage delete fails: show an error, keep the DB record (no orphaned reference without a file)

---

## What You Should NOT Do

```text
× Make the bucket public (all files contain user data)
× Store signed URLs in the database (they expire and become invalid)
× Store the file as a blob in the database
× Use the service role key in frontend code
× Trust file extension alone for MIME type validation
× Delete the DB record but leave the file in Storage (orphaned files)
× Allow one user's path to be readable by users in a different workspace
```

---

## Checklist to Run

Before marking done, run: [Storage Bucket Checklist](../../checklists/supabase/storage-bucket.md)

---

## Done When

```text
[ ] Bucket created as private with correct MIME type and size limits
[ ] File path structure follows {workspace_id}/{project_id}/{task_id}/{uuid}-{filename}
[ ] task_attachments table created with all columns and constraints
[ ] RLS policies on storage.objects: SELECT, INSERT, DELETE
[ ] RLS tested: member can access their workspace's files; non-member cannot
[ ] RLS tested: uploader can delete their own file; admin can delete any file
[ ] Frontend upload flow: validates type and size before upload; shows progress
[ ] Frontend download flow: uses signed URLs (not public URLs); 5-minute expiry
[ ] File deletion: removes both the DB record and the Storage file
[ ] Documented: bucket name, path structure, RLS policies, signed URL expiry
```
