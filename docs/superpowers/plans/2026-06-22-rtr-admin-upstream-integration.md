# RTR Admin Upstream Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring current `origin/main` into the saved RTR Admin branch while preserving the working Admin module, prompt plumbing, safety guidance, and local operating notes.

**Architecture:** Keep this as "upstream into ours": the integration branch starts at the saved RTR Admin branch and has `origin/main` merged into it. Accept upstream's new docs layout, dynamic mode, new modules, and FQL helper patterns; keep RTR Admin as an additional module that conforms to those new project conventions. Resolve generated docs by fixing the generator and regenerating, not by hand-editing each generated page.

**Tech Stack:** Python 3.12, FastMCP, FalconPy, `uv`, pytest, ruff, mypy, generated Markdown docs under `docs/modules/`.

---

## Current Saved State

- Main saved branch: `main`
- Integration branch: `codex/rtr-admin-upstream-integration`
- Integration worktree: `C:\Security\falcon-mcp\.worktrees\rtr-admin-upstream-integration`
- Merge currently open: `git merge --no-commit --no-ff origin/main`
- Saved commits before integration:
  - `767421a docs: save RTR admin working notes`
  - `075d747 chore: ignore local worktrees`

## What Can Merge As-Is

These upstream updates should generally be kept with minimal changes:

- `.env.example` and `.env.dev.example` dynamic-mode settings.
- `.github/` governance file moves and docs-check workflow.
- Removal of `docs-site/` and adoption of `docs/` as the source of truth.
- New upstream modules and resources:
  - `falcon_mcp/modules/exclusions.py`
  - `falcon_mcp/modules/host_groups.py`
  - `falcon_mcp/modules/policies.py`
  - matching resources and tests.
- Upstream shared helpers:
  - `falcon_mcp/common/fql.py`
  - `falcon_mcp/filter_hints.py`
  - `falcon_mcp/dynamic.py`
- Upstream changes to non-RTR modules unless a test proves interaction with RTR Admin.
- `uv.lock`, `pyproject.toml`, `server.json`, and `gemini-extension.json` version/dependency updates.

## What Needs Refactor Or Manual Merge

- `README.md`: keep upstream `developer.crowdstrike.com` links and new Exclusions/Host Groups/Policies rows; add the RTR Admin row back.
- `falcon_mcp/server.py`: combine upstream dynamic mode with the RTR Admin branch's prompt registration/counting.
- `scripts/generate_module_docs.py`: keep upstream `docs/` output, HTML metadata, GitHub admonitions, and site-base links; preserve prompt extraction and prompt rendering.
- `docs/modules/*.md`: resolve mechanically by regenerating after the generator conflict is fixed.
- `docs/modules/rtr-admin.md`: move from old `docs-site/src/content/docs/modules/rtr-admin.md` into the new generated `docs/modules/rtr-admin.md` shape.
- Validation tests: update any expectations that assume no prompts, no RTR Admin module, or old docs-site output.

---

### Task 1: Resolve The Narrow Source Conflicts

**Files:**
- Modify: `falcon_mcp/server.py`
- Modify: `scripts/generate_module_docs.py`
- Modify: `README.md`

- [ ] **Step 1: Resolve `falcon_mcp/server.py` by combining dynamic mode and prompt support**

Replace the conflicted logger block with:

```python
        prompt_count = self._register_prompts()
        prompt_word = "prompt" if prompt_count == 1 else "prompts"

        # Count modules and tools with proper grammar
        module_count = len(self.modules)
        module_word = "module" if module_count == 1 else "modules"

        logger.info(
            "Falcon MCP v%s - %d %s, %d %s, %d %s, %d %s%s",
            get_version(),
            module_count,
            module_word,
            tool_count,
            tool_word,
            resource_count,
            resource_word,
            prompt_count,
            prompt_word,
            " (dynamic mode)" if self.dynamic else "",
        )
```

Keep upstream's `dynamic: bool = False`, `self.dynamic = dynamic`, and `_register_tools()` dynamic branch. Keep the RTR Admin branch's `_register_prompts()` method:

```python
    def _register_prompts(self) -> int:
        """Register prompts from all modules.

        Returns:
            int: Number of prompts registered
        """
        for module in self.modules.values():
            if hasattr(module, "register_prompts") and callable(module.register_prompts):
                module.register_prompts(self.server)

        return sum(len(getattr(m, "prompts", [])) for m in self.modules.values())
```

- [ ] **Step 2: Resolve `scripts/generate_module_docs.py` by preserving prompts inside upstream docs format**

Keep upstream metadata output:

```python
    lines.append(f"<!-- meta:title {title} -->")
    lines.append(f"<!-- meta:description {description} -->")
    lines.append("<!-- meta:section modules -->")
    lines.append("<!-- meta:link-base /falcon-mcp/ -->")
    lines.append("<!-- frontmatter:sidebar order:10 -->")
```

Keep RTR Admin prompt extraction immediately after resource extraction:

```python
    # Extract resources
    resources = extract_resource_info(module_cls)

    # Extract prompts
    prompts = extract_prompt_info(module_cls)
```

Keep upstream GitHub admonitions:

```python
            if destructive:
                lines.append("> [!CAUTION]")
                lines.append("> This tool performs destructive operations.")
                lines.append("")
            elif not read_only:
                lines.append("> [!NOTE]")
                lines.append("> This tool modifies data.")
                lines.append("")
```

Keep the prompt rendering section before `return "\n".join(lines)`:

```python
    if prompts:
        lines.append("## Prompts")
        lines.append("")
        for prompt in prompts:
            lines.append(f"### `{prompt['name']}`")
            lines.append("")
            if prompt["title"]:
                lines.append(f"**Title:** {prompt['title']}")
                lines.append("")
            if prompt["description"]:
                lines.append(prompt["description"])
                lines.append("")
```

- [ ] **Step 3: Resolve `README.md`**

Use the upstream module table as the base, keep all upstream rows, and add this row after Real Time Response:

```markdown
| [Real Time Response Admin](https://developer.crowdstrike.com/falcon-mcp/modules/rtr-admin/) | Inspect RTR Admin assets, classify command risk, preview payloads, and execute approved admin workflows |
```

- [ ] **Step 4: Check no source conflict markers remain**

Run:

```powershell
rg -n "<<<<<<<|=======|>>>>>>>" falcon_mcp/server.py scripts/generate_module_docs.py README.md
```

Expected: no output.

---

### Task 2: Resolve Generated Docs Mechanically

**Files:**
- Modify: `docs/modules/*.md`
- Delete: old `docs-site/**`
- Keep generated: `docs/modules/rtr-admin.md`

- [ ] **Step 1: Accept upstream docs migration for non-RTR generated docs**

Run:

```powershell
git checkout --theirs -- docs/modules/cloud.md docs/modules/correlationrules.md docs/modules/custom-ioa.md docs/modules/data-protection.md docs/modules/detections.md docs/modules/discover.md docs/modules/firewall.md docs/modules/hosts.md docs/modules/idp.md docs/modules/intel.md docs/modules/ioc.md docs/modules/ngsiem.md docs/modules/quarantine.md docs/modules/rtr.md docs/modules/scheduled-reports.md docs/modules/sensor-usage.md docs/modules/serverless.md docs/modules/shield.md docs/modules/spotlight.md
```

Expected: non-RTR generated docs use upstream `<!-- meta:* -->` tags and `developer.crowdstrike.com` compatible links.

- [ ] **Step 2: Remove the old docs-site RTR Admin generated file from the merge**

Run:

```powershell
git rm -- docs-site/src/content/docs/modules/rtr-admin.md
```

Expected: old generated docs-site location is deleted.

- [ ] **Step 3: Regenerate module docs from the merged generator**

Run:

```powershell
$env:PYTHONUTF8 = "1"
$env:PYTHONIOENCODING = "utf-8"
$uv = Join-Path $env:APPDATA "Python\Python312\Scripts\uv.exe"
& $uv run python scripts/generate_module_docs.py
```

Expected:

```text
Generated: modules/rtr-admin.md
Done.
```

- [ ] **Step 4: Stage generated docs after verifying the RTR Admin page exists**

Run:

```powershell
Test-Path docs/modules/rtr-admin.md
git add docs/modules docs-site/src/content/docs/modules/rtr-admin.md
```

Expected: `Test-Path` returns `True`.

---

### Task 3: Verify RTR Admin Fits Upstream Dynamic Mode

**Files:**
- Inspect: `falcon_mcp/dynamic.py`
- Inspect: `falcon_mcp/modules/rtr_admin.py`
- Test: `tests/modules/test_dynamic.py`
- Test: `tests/modules/test_rtr_admin.py`
- Test: `tests/test_mcp_compliance.py`

- [ ] **Step 1: Confirm DynamicMode discovers RTR Admin tools through existing module registration**

Run:

```powershell
rg -n "register_tools|tools|execute_tool|search_tools" falcon_mcp/dynamic.py falcon_mcp/modules/rtr_admin.py
```

Expected: dynamic mode indexes registered module tools; RTR Admin should not need a separate dynamic registration path.

- [ ] **Step 2: Add or adjust a focused dynamic-mode test for RTR Admin discovery**

If `tests/modules/test_dynamic.py` does not cover arbitrary module tools, add:

```python
def test_dynamic_search_finds_rtr_admin_tools(self):
    module = RtrAdminModule(self.mock_client)
    module.register_tools(self.mock_server)
    dynamic = DynamicMode({"rtr_admin": module}, self.mock_server)

    results = dynamic.search_tools(query="runscript", limit=10)

    tool_names = {tool["name"] for tool in results["tools"]}
    self.assertIn("falcon_execute_admin_command", tool_names)
```

Use the actual RTR Admin command tool name from `tests/modules/test_rtr_admin.py` if it differs.

- [ ] **Step 3: Keep compliance expectations additive**

In `tests/test_mcp_compliance.py`, keep upstream mutating allowlist additions for host groups, exclusions, policies, and detections, and keep RTR Admin mutating/admin tool expectations from this branch.

Run:

```powershell
& $uv run pytest tests/test_mcp_compliance.py -q
```

Expected: compliance tests pass.

---

### Task 4: Run Focused Validation Before Full Gate

**Files:**
- Test: `tests/modules/test_rtr_admin.py`
- Test: `tests/integration/test_rtr_admin.py`
- Test: `tests/modules/test_dynamic.py`
- Test: `tests/test_registry.py`
- Test: `tests/test_mcp_compliance.py`

- [ ] **Step 1: Run import and formatting checks**

Run:

```powershell
& $uv run ruff check . --select I
& $uv run ruff check .
```

Expected: both pass. If import sorting changed due to merge, run the formatter/linter fix only on affected files and re-run.

- [ ] **Step 2: Run focused tests**

Run:

```powershell
& $uv run pytest tests/modules/test_rtr_admin.py tests/integration/test_rtr_admin.py tests/modules/test_dynamic.py tests/test_registry.py tests/test_mcp_compliance.py -q
```

Expected: all focused tests pass.

- [ ] **Step 3: Check generated docs are clean**

Run:

```powershell
& $uv run python scripts/generate_module_docs.py
git diff --exit-code docs/modules
```

Expected: no diff after regeneration.

---

### Task 5: Finish The Merge Commit

**Files:**
- All conflicted and auto-merged files.

- [ ] **Step 1: Confirm there are no unmerged paths**

Run:

```powershell
git diff --name-only --diff-filter=U
```

Expected: no output.

- [ ] **Step 2: Review the summary diff**

Run:

```powershell
git status --short
git diff --cached --stat
git diff --stat
```

Expected: staged merge contains upstream updates plus RTR Admin preservation; unstaged diff should be empty or only intentionally unresolved follow-up edits.

- [ ] **Step 3: Commit the integration merge**

Run:

```powershell
git add -A
git commit -m "merge upstream main into RTR admin branch"
```

Expected: merge commit succeeds on `codex/rtr-admin-upstream-integration`.

---

### Task 6: Run The Full Local Gate

**Files:**
- Full repository.

- [ ] **Step 1: Sync dependencies under Python 3.12**

Run:

```powershell
$env:PYTHONUTF8 = "1"
$env:PYTHONIOENCODING = "utf-8"
$uv = Join-Path $env:APPDATA "Python\Python312\Scripts\uv.exe"
& $uv sync --python 3.12 --extra dev
```

Expected: dependency sync completes.

- [ ] **Step 2: Run the project validation gate**

Run:

```powershell
& $uv run ruff check . --select I
& $uv run ruff check .
& $uv run mypy .
& $uv run pytest
& $uv run python scripts/generate_module_docs.py
git diff --exit-code
```

Expected: all commands pass and the worktree is clean after docs generation.

---

## Risks To Watch

- Dynamic mode may expose only three tools, so tests must not assume all RTR Admin tools are directly listed when `dynamic=True`.
- Prompt registration is additive and separate from tools; if upstream maintainers intended dynamic mode to hide prompts too, gate prompt registration behind `not self.dynamic` and update tests accordingly.
- Generated docs should not be hand-edited. If `docs/modules/rtr-admin.md` looks wrong, fix `scripts/generate_module_docs.py` or RTR Admin docstrings/examples, then regenerate.
- RTR Admin live-operation semantics must remain approval-gated; do not relax annotations or safety preview behavior to fit upstream tests.
