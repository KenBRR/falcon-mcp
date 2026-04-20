---
title: ODS
description: Hunting ODS scan results, launching scans, managing schedules, and reviewing malicious files found by ODS
sidebar:
  order: 10
---

Hunting ODS scan results, launching scans, managing schedules, and reviewing malicious files found by ODS

## API Scopes

- `On-demand scans (ODS):read`
- `On-demand scans (ODS):write`

## Tools

### `falcon_search_ods_scans`

**Required scopes:** `On-demand scans (ODS):read`

Search ODS scans and return full scan details.

### `falcon_get_ods_scan_details`

**Required scopes:** `On-demand scans (ODS):read`

Get full details for ODS scan IDs you already know.

### `falcon_search_ods_scan_hosts`

**Required scopes:** `On-demand scans (ODS):read`

Search ODS scan-host records and return full details.

### `falcon_get_ods_scan_host_details`

**Required scopes:** `On-demand scans (ODS):read`

Get full details for ODS scan-host IDs you already know.

### `falcon_launch_ods_scan`

**Required scopes:** `On-demand scans (ODS):write`

Create and start an on-demand scan.

### `falcon_cancel_ods_scans`

**Required scopes:** `On-demand scans (ODS):write`

Cancel one or more running ODS scans by ID.

### `falcon_search_ods_scheduled_scans`

**Required scopes:** `On-demand scans (ODS):read`

Search scheduled ODS scans and return full schedule details.

### `falcon_get_ods_scheduled_scan_details`

**Required scopes:** `On-demand scans (ODS):read`

Get full details for scheduled ODS scan IDs you already know.

### `falcon_schedule_ods_scan`

**Required scopes:** `On-demand scans (ODS):write`

Create or update a scheduled ODS scan definition.

### `falcon_delete_ods_scheduled_scans`

**Required scopes:** `On-demand scans (ODS):write`

Delete scheduled ODS scan definitions by ID or filter.

### `falcon_search_ods_malicious_files`

**Required scopes:** `On-demand scans (ODS):read`

Search malicious files found by ODS and return full details.

### `falcon_get_ods_malicious_file_details`

**Required scopes:** `On-demand scans (ODS):read`

Get full details for ODS malicious file IDs you already know.

## Resources

- **`falcon://ods/scans/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_ods_scans` tool.
- **`falcon://ods/scan-hosts/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_ods_scan_hosts` tool.
- **`falcon://ods/scheduled-scans/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_ods_scheduled_scans` and `falcon_delete_ods_scheduled_scans` tools.
- **`falcon://ods/malicious-files/search/fql-guide`**: Contains the guide for the `filter` param of the `falcon_search_ods_malicious_files` tool.
