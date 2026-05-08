# Project — Document Library

**Stack:** React (TypeScript) + Supabase
**Estimated Time:** 8 hours
**Difficulty:** Intermediate–Advanced

---

## What You Are Building

A workspace document library with folder organisation. Members can upload files and create folders. Files are stored in a private Supabase Storage bucket and accessed via signed URLs. Uploaders and admins can delete their own files. The library shows a breadcrumb-navigated folder tree.

---

## Features to Build

| # | Feature | Est. Time |
|---|---------|-----------|
| 1 | Database tables (`folders`, `documents`) + RLS | 1h |
| 2 | Storage bucket (`workspace-docs`) + RLS | 30m |
| 3 | RPC: `create_folder` | 45m |
| 4 | React: Document Library page (folder + file list) | 1h 30m |
| 5 | React: Upload file modal (drag-and-drop + progress) | 1h |
| 6 | React: Create folder modal | 30m |
| 7 | React: Download file (signed URL) | 30m |
| 8 | React: Delete file / folder | 45m |
| 9 | Breadcrumb navigation | 30m |

---

## 1. Database Tables

### `folders` Table

| Column | Type | Rules |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE |
| name | text | NOT NULL, max 100 chars |
| parent_id | uuid | FK → folders.id, ON DELETE CASCADE, nullable (null = root) |
| created_by | uuid | FK → auth.users.id |
| created_at | timestamptz | default now() |

```sql
-- Unique folder name within same parent and workspace
UNIQUE (workspace_id, parent_id, name)
```

### `documents` Table

| Column | Type | Rules |
|--------|------|-------|
| id | uuid | PK, default gen_random_uuid() |
| workspace_id | uuid | FK → workspaces.id, ON DELETE CASCADE |
| folder_id | uuid | FK → folders.id, ON DELETE CASCADE, nullable (null = root) |
| name | text | NOT NULL — display filename (original name) |
| storage_path | text | NOT NULL — full path in the bucket |
| file_size_bytes | integer | NOT NULL |
| mime_type | text | NOT NULL |
| uploaded_by | uuid | FK → auth.users.id |
| created_at | timestamptz | default now() |

```sql
-- Unique filename within same folder and workspace
UNIQUE (workspace_id, folder_id, name)
```

### RLS Policies (both tables)

```text
folders:
  SELECT: workspace members can view all folders in their workspace
  INSERT: workspace members can create folders
  DELETE: uploader or workspace admin/owner can delete

documents:
  SELECT: workspace members can view all documents in their workspace
  INSERT: workspace members can insert document records
  DELETE: uploader (uploaded_by = auth.uid()) OR workspace admin/owner can delete
```

**Run checklist:** [New Table](../../../../checklists/supabase/new-table.md) · [RLS Policies](../../../../checklists/supabase/rls-policies.md)

---

## 2. Storage Bucket — `workspace-docs`

- Visibility: **Private**
- Allowed MIME types: `image/jpeg`, `image/png`, `image/gif`, `application/pdf`, `application/msword`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`, `text/plain`
- Max file size: 25 MB

### File Path Structure

```
{workspace_id}/{folder_id_or_'root'}/{document_id}/{sanitized_filename}
```

### RLS on `storage.objects`

```text
SELECT: workspace member (check via workspace_members JOIN on workspace_id from path)
INSERT: workspace member — path must start with a workspace_id they belong to
DELETE: uploader (path contains their user folder) OR workspace admin
```

**Run checklist:** [Storage Bucket](../../../../checklists/supabase/storage-bucket.md)

---

## 3. RPC: `create_folder`

```sql
CREATE OR REPLACE FUNCTION create_folder(
  p_workspace_id  uuid,
  p_name          text,
  p_parent_id     uuid DEFAULT NULL
)
RETURNS json
LANGUAGE plpgsql
SECURITY INVOKER
AS $$
DECLARE
  v_user_id uuid := auth.uid();
  v_folder  folders%ROWTYPE;
BEGIN
  -- 1. Auth
  IF v_user_id IS NULL THEN
    RAISE EXCEPTION 'AUTH_REQUIRED: authentication required';
  END IF;

  -- 2. Validate name
  IF p_name IS NULL OR trim(p_name) = '' THEN
    RAISE EXCEPTION 'VALIDATION_ERROR: folder name is required';
  END IF;
  IF length(trim(p_name)) > 100 THEN
    RAISE EXCEPTION 'VALIDATION_ERROR: folder name must be 100 characters or fewer';
  END IF;

  -- 3. Workspace membership check
  IF NOT EXISTS (SELECT 1 FROM workspace_members WHERE workspace_id = p_workspace_id AND user_id = v_user_id) THEN
    RAISE EXCEPTION 'FORBIDDEN: you are not a member of this workspace';
  END IF;

  -- 4. Parent folder exists and belongs to same workspace (if provided)
  IF p_parent_id IS NOT NULL THEN
    IF NOT EXISTS (SELECT 1 FROM folders WHERE id = p_parent_id AND workspace_id = p_workspace_id) THEN
      RAISE EXCEPTION 'NOT_FOUND: parent folder not found';
    END IF;
  END IF;

  -- 5. Uniqueness check within same parent
  IF EXISTS (SELECT 1 FROM folders WHERE workspace_id = p_workspace_id AND parent_id IS NOT DISTINCT FROM p_parent_id AND name = trim(p_name)) THEN
    RAISE EXCEPTION 'DUPLICATE: a folder with this name already exists here';
  END IF;

  INSERT INTO folders (workspace_id, name, parent_id, created_by)
  VALUES (p_workspace_id, trim(p_name), p_parent_id, v_user_id)
  RETURNING * INTO v_folder;

  RETURN json_build_object('data', row_to_json(v_folder));
END;
$$;
```

**Run checklist:** [RPC Function](../../../../checklists/supabase/rpc-function.md)

---

## 4. React: Document Library Page

**URL:** `/workspaces/{id}/docs`

### Layout

```text
LEFT/TOP: Breadcrumb navigation
  Home / Marketing / 2024 Q1

CONTENT AREA: Two sections (in order)
  1. Folders list — each folder shows: icon, name, created by, created date, actions (delete)
  2. Files list — each file shows: file type icon, name, size, uploaded by, date, actions

HEADER ACTIONS:
  "Create Folder" button (all members)
  "Upload File" button (all members)

FOLDER ROW ACTIONS:
  Delete (visible to uploader + admins only)

FILE ROW ACTIONS:
  Download · Delete (visible to uploader + admins only)
```

### States

```text
[ ] Loading: skeleton rows for both folders and files sections
[ ] Empty folder (no subfolders, no files): "This folder is empty." + Upload/Create buttons
[ ] Root empty: "No documents yet." + Upload/Create buttons
[ ] Error: "Could not load documents." + retry
```

### TypeScript Interfaces

```ts
interface Folder {
  id: string;
  name: string;
  parent_id: string | null;
  created_by: string;
  created_at: string;
}

interface Document {
  id: string;
  folder_id: string | null;
  name: string;
  storage_path: string;
  file_size_bytes: number;
  mime_type: string;
  uploaded_by: string;
  created_at: string;
}
```

**Run checklist:** [React Page Build](../../../../checklists/website/react-page.md) · [Loading States & Skeletons](../../../../checklists/frontend/loading-states-skeletons.md)

---

## 5. Upload File Modal

```text
FIELDS:
[ ] Drag-and-drop zone (or click to browse)
[ ] Shows: accepted types, max 25 MB

VALIDATION (before upload):
[ ] MIME type checked (not just extension)
[ ] File size checked: > 25 MB shows "File too large" immediately
[ ] Multiple files selectable (optional) — upload sequentially

UPLOAD FLOW:
[ ] Show progress bar per file during upload
[ ] On storage upload success: insert row into documents table
[ ] On success: "File uploaded." toast, file appears in list
[ ] On storage error: show error, do NOT insert document record
[ ] On DB insert error: delete the uploaded file from storage (cleanup)
[ ] Cancel button during upload: cancels the upload

DUPLICATE FILENAME:
[ ] If a file with the same name exists in this folder:
    "A file named '[name]' already exists here. Replace it?" → confirm → overwrite
```

**Run checklist:** [Modals & Dialogs](../../../../checklists/frontend/modals-dialogs.md)

---

## 6. Create Folder Modal

```text
Fields: Folder Name (text, required, max 100 chars)

Behavior:
[ ] Calls create_folder RPC
[ ] Duplicate name: "A folder with this name already exists here." as field error
[ ] On success: folder appears in list, "Folder created." toast
[ ] Loading state on Create button
```

---

## 7. Download File

```text
[ ] "Download" button on each file row
[ ] Calls supabase.storage.from('workspace-docs').createSignedUrl(path, 60)
[ ] Opens signed URL in new tab (or triggers browser download)
[ ] Loading indicator on download button during URL generation
[ ] 60-second expiry on signed URL (short — for immediate download only)
[ ] Never store the signed URL — generate at click time
[ ] On error: "Could not generate download link. Please try again."
```

---

## 8. Delete File / Delete Folder

### Delete File

```text
[ ] Confirmation: "Delete '[filename]'? This cannot be undone."
[ ] On confirm:
    1. Delete document record from database
    2. Delete file from Supabase Storage
    If storage delete fails: show error, restore DB record (or show warning about orphan)
[ ] On success: file removed from list, "File deleted." toast
```

### Delete Folder

```text
[ ] Only allow deleting EMPTY folders (no subfolders, no files)
[ ] If folder has contents: "This folder is not empty. Delete all files inside first."
[ ] Confirmation: "Delete folder '[name]'? This cannot be undone."
[ ] On success: folder removed from list, "Folder deleted." toast
```

**Run checklist:** [Delete & Destructive Actions](../../../../checklists/frontend/delete-destructive-actions.md)

---

## 9. Breadcrumb Navigation

```text
[ ] Shows full path from root: Home / Marketing / 2024 Q1
[ ] Each segment is a clickable link that navigates to that folder
[ ] "Home" always links to the root
[ ] Current folder (last segment) not a link — just text
[ ] URL updated on folder navigation: ?folder_id={uuid}
[ ] Back button works (browser history)
[ ] Folder ID from URL used on page load to show correct content
```

---

## What You Should NOT Do

```text
× Make the storage bucket public — documents contain workspace data
× Store signed download URLs in the database — generate at request time
× Delete the DB record without deleting the Storage file (orphaned file)
× Upload to Storage without inserting the document record (orphaned storage file)
× Validate file type by extension only — validate the actual MIME type
× Allow deleting a folder that still has content — prevent with a check
× Use sequential IDs in the storage path — use document UUID
× Skip the breadcrumb URL param — folder navigation must be shareable/bookmarkable
```

---

## Checklists to Run (in order)

```text
[ ] New Table — folders table
[ ] New Table — documents table
[ ] RLS Policies — folders + documents tables
[ ] Storage Bucket — workspace-docs bucket
[ ] RPC Function — create_folder
[ ] React Page Build — Document Library page
[ ] Modals & Dialogs — Upload modal, Create Folder modal
[ ] Loading States & Skeletons — library page
[ ] Error Handling — download errors, upload errors, delete errors
[ ] Delete & Destructive Actions — delete file, delete folder
[ ] Notifications & Toasts — all async actions
[ ] Permissions & Role-Based UI — delete visibility (uploader + admin only)
```

---

## Done When

```text
[ ] folders table: all columns, unique constraint per parent+workspace, RLS
[ ] documents table: all columns, unique constraint per folder+workspace, RLS
[ ] workspace-docs storage bucket: private, MIME restricted, RLS on storage.objects
[ ] create_folder RPC: 5-step validation, duplicate check, returns folder data
[ ] Library page: shows folders first, then files
[ ] Library page: all 3 states (loading/empty/error)
[ ] Upload modal: MIME type validated before upload, 25 MB limit enforced
[ ] Upload modal: progress bar shown during upload
[ ] Upload modal: if storage upload fails, no document record inserted
[ ] Upload modal: if DB insert fails, storage file is cleaned up
[ ] Download: signed URL generated at click time (60s expiry), not stored
[ ] Breadcrumb: full path shown, each segment clickable, URL has ?folder_id param
[ ] Folder navigation: URL updates on click, back button restores previous folder
[ ] Delete file: removes both DB record and Storage file
[ ] Delete folder: blocked if folder contains files or subfolders
[ ] Delete actions: visible only to uploader or admin/owner
[ ] Duplicate folder name: shown as field error in modal (not a toast)
[ ] Mobile tested at 375px and 768px
[ ] No silent failures — all errors shown in UI
```
