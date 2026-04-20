---
title: Sample Uploads
description: Uploading suspicious files and archives, inspecting archive contents, and extracting files for analysis
sidebar:
  order: 10
---

Uploading suspicious files and archives, inspecting archive contents, and extracting files for analysis

## API Scopes

- `Sample uploads:read`
- `Sample uploads:write`

## Tools

### `falcon_upload_sample_for_cloud_analysis`

**Required scopes:** `Sample uploads:write`

Upload a file or base64 payload to the Sample Uploads service.

### `falcon_delete_uploaded_samples`

**Required scopes:** `Sample uploads:write`

Delete previously uploaded samples from the Sample Uploads service.

### `falcon_list_uploaded_archives`

**Required scopes:** `Sample uploads:read`

List files discovered inside an uploaded archive by SHA256.

### `falcon_get_archive_upload_status`

**Required scopes:** `Sample uploads:read`

Get processing status for an uploaded archive by SHA256.

### `falcon_upload_archive_for_extraction`

**Required scopes:** `Sample uploads:write`

Upload an archive and queue it for extraction-aware analysis workflows.

### `falcon_delete_uploaded_archive`

**Required scopes:** `Sample uploads:write`

Delete an uploaded archive from the Sample Uploads service.

### `falcon_list_archive_extractions`

**Required scopes:** `Sample uploads:read`

List files associated with an archive extraction operation ID.

### `falcon_get_archive_extraction_status`

**Required scopes:** `Sample uploads:read`

Get status for an archive extraction job by extraction ID.

### `falcon_create_archive_extraction`

**Required scopes:** `Sample uploads:write`

Extract files from an uploaded archive into Falcon internal storage.
