---
title: Sandbox
description: Uploading samples, submitting detonations, and retrieving Sandbox submission and report details
sidebar:
  order: 10
---

Uploading samples, submitting detonations, and retrieving Sandbox submission and report details

## API Scopes

- `Sandbox (Falcon Intelligence):read`
- `Sandbox (Falcon Intelligence):write`

## Tools

### `falcon_upload_sandbox_sample`

**Required scopes:** `Sandbox (Falcon Intelligence):write`

Upload a local file or base64 payload directly to Falcon Sandbox.

### `falcon_check_sandbox_samples`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Check which SHA256 hashes already exist in Falcon Sandbox storage.

### `falcon_submit_sandbox_analysis`

**Required scopes:** `Sandbox (Falcon Intelligence):write`

Submit a URL or uploaded SHA256 for Falcon Sandbox analysis.

This performs a live detonation, consumes Falcon Sandbox quota, and cannot be undone once submitted.

### `falcon_search_sandbox_submissions`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Search Falcon Sandbox submissions and return full submission details.

### `falcon_get_sandbox_submission_details`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Retrieve submission details for IDs you already know.

### `falcon_search_sandbox_reports`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Search Falcon Sandbox reports and return summary report data.

### `falcon_get_sandbox_report_summaries`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Get summary report data for report IDs you already know.

### `falcon_get_sandbox_report_details`

**Required scopes:** `Sandbox (Falcon Intelligence):read`

Get full report details for report IDs you already know.

## Resources

- **`falcon://sandbox/submissions/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_sandbox_submissions` tool.
- **`falcon://sandbox/reports/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_sandbox_reports` tool.
