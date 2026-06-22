# RTR Admin MCP Surface Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand the existing RTR Admin module from tool-only API coverage into a fuller MCP surface with prompts, richer resources, docs, and tests.

**Architecture:** Keep RTR Admin ownership inside `falcon_mcp/modules/rtr_admin.py`. Add shared prompt registration plumbing to `BaseModule` and `FalconMCPServer`, then add RTR Admin-specific prompts/resources that guide safe planning, approval packets, runscript review, and status interpretation. Update generated docs so prompts appear alongside tools and resources.

**Tech Stack:** Python 3.11+, FastMCP prompt support via `mcp.server.fastmcp.prompts.Prompt`, pytest, ruff, docs generator.

---

### Task 1: Add Prompt Registration Plumbing

**Files:**
- Modify: `falcon_mcp/modules/base.py`
- Modify: `falcon_mcp/server.py`
- Modify: `tests/modules/utils/test_modules.py`
- Test: `tests/test_registry.py`

- [x] **Step 1: Extend `BaseModule` for prompts**

Add `self.prompts`, a no-op `register_prompts()`, and `_add_prompt()` using `Prompt.from_function`.

```python
from mcp.server.fastmcp.prompts import Prompt

class BaseModule(ABC):
    def __init__(self, client: FalconClient):
        self.client = client
        self.tools: list[str] = []
        self.resources: list[str] = []
        self.prompts: list[str] = []

    def register_prompts(self, server: FastMCP) -> None:
        """Register prompts with the MCP Server."""

    def _add_prompt(
        self,
        server: FastMCP,
        method: Callable[..., Any],
        name: str,
        title: str | None = None,
        description: str | None = None,
    ) -> None:
        prefixed_name = f"falcon_{name}"
        prompt = Prompt.from_function(
            method,
            name=prefixed_name,
            title=title,
            description=description,
        )
        server.add_prompt(prompt)
        self.prompts.append(prefixed_name)
```

- [x] **Step 2: Register prompts in the main server**

In `FalconMCPServer.__init__`, call `_register_prompts()` after resources and include the prompt count in the startup log.

```python
prompt_count = self._register_prompts()
prompt_word = "prompt" if prompt_count == 1 else "prompts"

logger.info(
    "Falcon MCP v%s — %d %s, %d %s, %d %s, %d %s",
    get_version(),
    module_count,
    module_word,
    tool_count,
    tool_word,
    resource_count,
    resource_word,
    prompt_count,
    prompt_word,
)
```

Add:

```python
def _register_prompts(self) -> int:
    """Register prompts from all modules."""
    for module in self.modules.values():
        if hasattr(module, "register_prompts") and callable(module.register_prompts):
            module.register_prompts(self.server)
    return sum(len(getattr(m, "prompts", [])) for m in self.modules.values())
```

- [x] **Step 3: Add prompt assertion helper**

In `tests/modules/utils/test_modules.py`, add:

```python
def assert_prompts_registered(self, expected_prompts):
    self.module.register_prompts(self.mock_server)
    self.assertEqual(self.mock_server.add_prompt.call_count, len(expected_prompts))
    registered_prompts = [
        call.args[0].name if call.args else call.kwargs["prompt"].name
        for call in self.mock_server.add_prompt.call_args_list
    ]
    for prompt in expected_prompts:
        self.assertIn(prompt, registered_prompts)
```

- [x] **Step 4: Verify with existing tests**

Run:

```powershell
.\.venv\Scripts\python.exe -m pytest tests/test_registry.py -q
```

Expected: tests pass with no prompt registration errors.

### Task 2: Add RTR Admin Prompts

**Files:**
- Modify: `falcon_mcp/modules/rtr_admin.py`
- Modify: `tests/modules/test_rtr_admin.py`

- [x] **Step 1: Register RTR Admin prompts**

Add `register_prompts()` with four prompts:

```python
def register_prompts(self, server: FastMCP) -> None:
    self._add_prompt(
        server,
        self.plan_rtr_admin_action,
        "plan_rtr_admin_action",
        title="Plan RTR Admin Action",
        description="Plan a safe RTR Admin workflow before using execution tools.",
    )
    self._add_prompt(
        server,
        self.build_rtr_admin_approval_packet,
        "build_rtr_admin_approval_packet",
        title="Build RTR Admin Approval Packet",
        description="Create an operator approval packet for a high-impact RTR Admin command.",
    )
    self._add_prompt(
        server,
        self.review_rtr_admin_runscript,
        "review_rtr_admin_runscript",
        title="Review RTR Admin Runscript",
        description="Review an RTR Admin runscript command string for safety and quoting risks.",
    )
    self._add_prompt(
        server,
        self.interpret_rtr_admin_status,
        "interpret_rtr_admin_status",
        title="Interpret RTR Admin Status",
        description="Interpret RTR Admin command status output and suggest next safe steps.",
    )
```

- [x] **Step 2: Add prompt methods**

Each method returns a prompt string and must not call Falcon.

```python
def plan_rtr_admin_action(
    self,
    objective: str,
    target_hostname: str | None = None,
    session_id: str | None = None,
    device_id: str | None = None,
    ticket: str | None = None,
) -> str:
    return self._prompt_text(
        "Plan an RTR Admin action.",
        objective=objective,
        target_hostname=target_hostname,
        session_id=session_id,
        device_id=device_id,
        ticket=ticket,
    )
```

Implement the four methods with concrete text that tells the model to use inventory, classify, preview, get explicit approval for high-impact commands, execute only after review, and poll status.

- [x] **Step 3: Add tests**

In `tests/modules/test_rtr_admin.py`, assert the four prompt names register and render text includes key safety terms:

```python
def test_register_prompts(self):
    expected_prompts = [
        "falcon_plan_rtr_admin_action",
        "falcon_build_rtr_admin_approval_packet",
        "falcon_review_rtr_admin_runscript",
        "falcon_interpret_rtr_admin_status",
    ]
    self.assert_prompts_registered(expected_prompts)
```

Also assert direct calls do not call Falcon:

```python
text = self.module.plan_rtr_admin_action(objective="collect triage evidence")
self.assertIn("falcon_classify_rtr_admin_command", text)
self.mock_client.command.assert_not_called()
```

- [x] **Step 4: Run focused tests**

Run:

```powershell
.\.venv\Scripts\python.exe -m pytest tests/modules/test_rtr_admin.py -q
```

Expected: all RTR Admin tests pass.

### Task 3: Add RTR Admin Context Resources

**Files:**
- Modify: `falcon_mcp/modules/rtr_admin.py`
- Modify: `falcon_mcp/resources/rtr_admin.py`
- Modify: `tests/modules/test_rtr_admin.py`

- [x] **Step 1: Add approval template resource text**

In `falcon_mcp/resources/rtr_admin.py`, add `RTR_ADMIN_APPROVAL_PACKET_TEMPLATE` with fields for target, command, classification, approval hash, reason, ticket, expected effect, and status polling next step.

- [x] **Step 2: Generate policy guide from module constants**

In `RTRAdminModule`, add `_command_policy_guide()` that formats the command sets already defined in the module:

```python
def _command_policy_guide(self) -> str:
    return "\n".join([
        "RTR Admin Command Policy Guide",
        "",
        f"Read-only commands: {', '.join(sorted(READ_ONLY_ADMIN_COMMANDS))}",
        f"Evidence collection commands: {', '.join(sorted(EVIDENCE_COLLECTION_COMMANDS))}",
        f"Sensitive collection commands: {', '.join(sorted(SENSITIVE_COLLECTION_COMMANDS))}",
        f"High-impact commands: {', '.join(sorted(BLOCKED_ADMIN_COMMANDS))}",
        "Unknown commands are blocked until reviewed and allowlisted.",
    ])
```

- [x] **Step 3: Register both resources**

Add:

```python
TextResource(
    uri=AnyUrl("falcon://rtr-admin/policy/command-guide"),
    name="falcon_rtr_admin_command_policy_guide",
    description="Contains RTR Admin command classification policy and command categories.",
    text=self._command_policy_guide(),
)
TextResource(
    uri=AnyUrl("falcon://rtr-admin/approval/packet-template"),
    name="falcon_rtr_admin_approval_packet_template",
    description="Contains the approval packet template for high-impact RTR Admin commands.",
    text=RTR_ADMIN_APPROVAL_PACKET_TEMPLATE,
)
```

- [x] **Step 4: Test resource registration and content**

Update expected resources and assert the policy guide includes `rm`, `runscript`, `ps`, and “Unknown commands”.

### Task 4: Document Prompts in Generated Docs

**Files:**
- Modify: `scripts/generate_module_docs.py`
- Modify: `docs-site/src/content/docs/modules/rtr-admin.md`

- [x] **Step 1: Extract registered prompt metadata**

Add `extract_prompt_info(module_cls)` that inspects `register_prompts()` and returns prompt name, title, and description from `_add_prompt(...)` blocks.

- [x] **Step 2: Add a Prompts section to module docs**

In `generate_module_page()`, add:

```python
prompts = extract_prompt_info(module_cls)
...
if prompts:
    lines.append("## Prompts")
    lines.append("")
    for p in prompts:
        lines.append(f"### `{p['name']}`")
        lines.append("")
        if p["title"]:
            lines.append(f"**Title:** {p['title']}")
            lines.append("")
        if p["description"]:
            lines.append(p["description"])
            lines.append("")
```

- [x] **Step 3: Regenerate docs**

Run:

```powershell
.\.venv\Scripts\python.exe scripts/generate_module_docs.py
```

Expected: `docs-site/src/content/docs/modules/rtr-admin.md` includes Tools, Resources, and Prompts.

### Task 5: Verify and Review

**Files:**
- All touched files

- [x] **Step 1: Run focused unit tests**

```powershell
.\.venv\Scripts\python.exe -m pytest tests/modules/test_rtr_admin.py tests/modules/test_rtr.py tests/test_registry.py tests/test_mcp_compliance.py -q
```

Expected: pass.

- [x] **Step 2: Run integration smoke**

```powershell
.\.venv\Scripts\python.exe -m pytest tests/integration/test_rtr_admin.py -q
```

Expected: pass or scope-based skips, no missing method errors.

- [x] **Step 3: Run ruff**

```powershell
.\.venv\Scripts\python.exe -m ruff check falcon_mcp/modules/base.py falcon_mcp/server.py falcon_mcp/modules/rtr_admin.py falcon_mcp/resources/rtr_admin.py scripts/generate_module_docs.py tests/modules/test_rtr_admin.py tests/integration/test_rtr_admin.py tests/modules/utils/test_modules.py
```

Expected: all checks pass.

- [x] **Step 4: Run CodeRabbit from WSL**

```bash
cd /mnt/c/Security/falcon-mcp
coderabbit review --agent -t uncommitted
```

Expected: CodeRabbit reports issues or confirms no issues. Fix critical/major issues before completion.

Observed:
- Full uncommitted review exceeded CodeRabbit's 150-file limit.
- `--dir falcon_mcp` completed with one existing-code IOC finding outside this branch's diff.
- `--dir tests` completed with 0 findings.
- `--dir scripts` found one generator resilience issue; it was fixed and the rerun completed with 0 findings.
- `--dir docs-site/src/content/docs/modules` was not completed because CodeRabbit returned a recoverable rate limit.

Follow-up A-grade hardening added:
- `falcon_get_rtr_put_file_contents` for explicit-ID read-only put-file content retrieval.
- `falcon_run_rtr_admin_command_and_wait` for single-host admin execution plus status polling using the same classification and approval gate as `falcon_execute_rtr_admin_command`.
- Prompt-render smoke coverage so registered MCP prompts are exercised through `Prompt.render(...)`.
- RTR Admin command policy now recognizes documented `unmount` as high-impact approval-gated behavior.

---

## Self-Review

Spec coverage:
- Prompts: Task 1 and Task 2.
- Resources/context: Task 3.
- Tools retained and not expanded unnecessarily: Tasks 2 and 3 only add prompt/resource surface.
- Docs: Task 4.
- Verification and CodeRabbit review: Task 5.

Placeholder scan:
- No TODO/TBD placeholders.
- Every task names files, code shape, commands, and expected result.

Type consistency:
- Prompt names use the repo’s existing `falcon_` prefix convention.
- Prompt registration mirrors existing tool/resource registration ownership.
