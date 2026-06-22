# Agent Guidance for Falcon MCP

This file is a lightweight operating guide for AI agents working in this repository. It sets minimum habits, not a ceiling. Use current repository docs, GitHub state, maintainer feedback, and engineering judgment when they provide more specific guidance.

## Start With Current State

- Before development or PR review, refresh upstream state with `git fetch --all --prune`.
- Compare the working branch with current `origin/main`; do not rely on stale local `main`, old PR notes, or memory.
- Check the live GitHub PR/branch state when relevant with `gh pr view`, `gh pr checks`, `gh pr diff`, `gh pr status`, or the GitHub UI.
- If a maintainer has pushed commits, review those commits directly before continuing: `git show`, `git log --oneline --decorate --graph`, and targeted diffs.
- Treat current project files as the source of truth. Re-read the relevant docs instead of assuming prior project rules still apply.

## Read The Project Rules That Apply

For MCP module work, inspect these before changing code:

- `docs/CONTRIBUTING.md`
- `docs/development/module_development.md`
- `docs/development/resource_development.md`
- `docs/development/docs_site.md`
- `tests/test_mcp_compliance.py`
- `scripts/generate_module_docs.py`

This `AGENTS.md` should not replace those files. If there is a conflict, follow the current project docs, tests, and maintainer direction.

## MCP Development Checks

- Keep tool surfaces small and consistent with nearby modules. Prefer one clear tool with `ids` or `filter` inputs over several overlapping tools when that matches project style.
- For Falcon API pagination, polling, cursors, sequence IDs, or continuation tokens, use state returned by the API response. Do not infer next state unless the API docs or existing code clearly does that.
- Mutating tools must have explicit `ToolAnnotations`; verify `readOnlyHint`, `destructiveHint`, `idempotentHint`, and `openWorldHint` against the operation semantics.
- Update mutating-tool allowlists and compliance expectations when adding or changing tools.
- Add or update `TOOL_EXAMPLES` and module metadata through `scripts/generate_module_docs.py`; do not hand-edit generated module docs.
- Keep docstrings in the project style: what the tool does, when to use it, what resource to consult, and what it returns.

## Local Validation Gate

Use Python 3.12 for local validation unless the project changes its supported matrix. On Windows, force UTF-8 for docs generation.

```powershell
$env:PYTHONUTF8 = "1"
$env:PYTHONIOENCODING = "utf-8"
$uv = Join-Path $env:APPDATA "Python\Python312\Scripts\uv.exe"

& $uv sync --python 3.12 --extra dev
& $uv run ruff check . --select I
& $uv run ruff check .
& $uv run mypy .
& $uv run pytest
& $uv run python scripts/generate_module_docs.py
git diff --exit-code
```

For docs-site changes, also run:

```powershell
Push-Location docs-site
corepack pnpm install --frozen-lockfile
corepack pnpm run build
Pop-Location
```

If any gate cannot be run, say exactly which command was skipped and why.

## PR Readiness

- Do not mark a PR ready based only on CodeRabbit or one passing test. CodeRabbit is review input, not the project authority.
- Before asking for review, check the diff against current `origin/main`, regenerated docs, compliance tests, and CI status.
- Summarize known risks plainly in the PR instead of hiding uncertainty.
- If a maintainer changes the PR, fold the lesson back into the next review pass.
