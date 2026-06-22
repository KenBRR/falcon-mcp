# RTR Admin Follow-Up Note

Captured from the 2026-06-02 live RTR Admin deep test. This note is for later
work that should not be folded into the current hardening pass because it would
expand the module surface, touch adjacent modules, or require a separately
planned live validation pass.

## Later Candidates

- Consider direct get-by-ID helper behavior for custom scripts and put-files if
  Falcon FQL ID filters remain unreliable. Live testing showed broad search
  results can return known IDs that do not match later `id:` filters.
- Consider a metadata-only script inventory mode, or separate content/detail
  retrieval, because custom and Falcon script search responses can include full
  script bodies.
- Put-file inventory `file_type` is not enough to predict retrieval exposure.
  Live full-flow testing found `file_type: binary` inventory that retrieved text
  content. Current hardening warns on retrieved text content; a stricter future
  mode could require explicit content-review intent before returning any
  put-file text.
- Put-file name filtering is narrower than the guide originally implied. Live
  testing matched an exact put-file name, but contains and wildcard variants
  returned no rows.
  Future work should build a small live FQL compatibility matrix for custom
  scripts and put-files.
- Add a separately gated live smoke suite for RTR Admin that verifies one
  known-good inventory sort, safe binary put-file content handling, read-only
  `pwd`, a valid `reg query` shape, command submission, and status polling.
- Re-test adjacent RTR session and audit FQL behavior. The resource guides now
  warn about exact lookup caveats, but a future live semantic test should prove
  which exact filters are reliable across tenants.
- Extend command-shape validation beyond the current `reg query` warning only
  after collecting more live parser evidence for each RTR Admin command.
- Confirm put-file sort direction against a stable test tenant before adding a
  stronger ordering assertion. The module now preserves list ID order after
  detail fetch, but the API's own sort direction should still be validated live.

## Unrelated Review Items Observed

These came up while running scoped CodeRabbit review on the uncommitted diff.
They were not fixed here because they are outside the RTR Admin module pass.

- `falcon_mcp/resources/data_protection.py`: align documented FQL operator lists
  across the data-protection classification, policy, and content-pattern guides.
- `falcon_mcp/resources/intel.py`: replace nonstandard `_partial_` wildcard
  examples with asterisk wildcard examples.
- `falcon_mcp/resources/cloud.py`: replace the duplicate cluster-name example
  for `container_name` with a container-like value.
- `falcon_mcp/resources/cloud.py`: fix the `cps_rating` description acronym.
- `falcon_mcp/modules/idp.py`: replace deprecated `datetime.utcnow()` calls with
  timezone-aware UTC timestamps.
- `falcon_mcp/modules/idp.py`: review GraphQL query builders for entity ID
  sanitization before interpolating IDs into query strings.
- `tests/modules/utils/test_modules.py`: fix a stale `expected_tools` docstring
  reference in `assert_resources_registered`.
- `tests/e2e/utils/base_e2e_test.py`: clean up typing/name nits around
  `Callable`, `**kwargs`, and `DEFAULT_MODLES_TO_TEST`.
- CodeRabbit recommended reverting RTR Admin sort examples to `created_at|desc`.
  That was intentionally not applied because live Falcon testing in this case
  showed `created_at` returning server errors while `created_timestamp` worked.
