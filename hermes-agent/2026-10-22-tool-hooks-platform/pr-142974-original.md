### What

`pre_llm_call` has always carried `platform`; `pre_tool_call`, `post_tool_call` and
`transform_tool_result` never did. This threads `agent.platform` through the shared identity set so
a policy hook can tell an attended turn from an unattended one (`cron` / `subagent` / `curator`).

Fixes #142973.

### Why the shared set, not three call sites

`tool_hook_ids()` is the single place that builds the identity kwargs for the tool path, and
`_CallIds` is the single carrier for the two `model_tools` paths. Adding the field there means the
three hooks, tool-side middleware and the inline executors all agree, and no future hook call site
can forget it.

| Where | Change |
|---|---|
| `agent/inline_tool_executors.py` | `tool_hook_ids()` adds `platform` (and its docstring says why) |
| `model_tools.py` | `_CallIds.platform`; `_emit_post_tool_call_hook(platform=...)`; `handle_function_call(platform=...)` |
| `hermes_cli/plugins.py` | `_get_pre_tool_call_directive_details(platform=...)` forwards it to the `pre_tool_call` payload |
| `agent/agent_runtime_helpers.py` | `_pre_tool_block_message()` now reuses `tool_hook_ids()` instead of re-listing the ids; the registry dispatch passes `platform` |
| `agent/tool_executor.py` | registry dispatch passes `platform` |
| `hermes_cli/hooks.py` | `_DEFAULT_PAYLOADS` sample payloads gain `"platform": "cli"` (used by `hermes hooks test` / `doctor`) |
| `website/docs/.../hooks.md` + `zh-Hans` mirror | catalog rows and the `pre_tool_call` parameter table document `platform` |
| `tests/agent/test_tool_hook_platform.py` | 6 unit invariants (all red on the base) |
| `tests/cron/test_cron_tool_hook_platform.py` | end-to-end: real scheduler tick, real plugin load, `platform == "cron"` |

### Coverage — every construction site accounted for

| Identity source | Carries `platform` |
|---|---|
| `tool_hook_ids()` (all 7 call sites: `agent_runtime_helpers` ×2, `inline_tool_executors` ×2, `tool_executor` ×3) | yes, single source |
| `_CallIds` (3 constructions: `model_tools` ×2, +1 via `tool_hook_ids`) | yes |
| `post_tool_call` emit (`_emit` → `asdict(ids)`) | yes |
| `transform_tool_result` emit (`ids.hook_kwargs()`) | yes |
| `pre_tool_call` emit (`**ids.hook_kwargs()` / `**tool_hook_ids(...)`) | yes |
| tool request/execution middleware (`**hook_kwargs`) | yes |

### Notes reviewers may want

- **Middleware alignment, not a side effect.** Tool-side middleware signatures are
  `**context`, and LLM-side middleware already receives `platform`
  (`agent/turn_api_request.py`). Both sides now see the same field.
- **Shell hooks get it for free.** `_payload_fields()` promotes only `tool_name` / `args` /
  `session_id` / `parent_session_id`; everything else lands in `extra`, so `platform` shows up in a
  shell hook's JSON without any shell-specific change. Covered by a test.
- **Callers with no agent keep today's behaviour.** The code kernel, MCP bridge, TUI/connector
  dispatch call `handle_function_call()` without `platform`; the field defaults to `None` and the
  wire value is `""`, exactly as the other identity fields behave outside a turn.
- **Additive keyword field**, which is the pattern `plugins/AGENTS.md` prescribes: callbacks are
  signature-inspected, so narrow signatures are unaffected and `**kwargs` callbacks get the field.
- **Append-only signature growth.** `platform` is the *last* parameter of `handle_function_call`
  and of `_CallIds`, so no existing positional call changes meaning — the repo convention of adding
  keyword fields without reshaping the existing ones holds even for the semi-public entry point.

### Tests

`scripts/run_tests.sh` could not bootstrap in this sandbox (its pinned `uv` download is blocked), so
I ran its own per-file runner directly with the same env (`TZ=UTC LANG=C.UTF-8 PYTHONHASHSEED=0`):

```
before (base 81a55437f3):  2 files, 0 passed, 7 failed
after  (this branch):      2 files, 7 passed, 0 failed
```

Related suites (re-run on this branch): `tests/hermes_cli/test_plugins.py`,
`tests/hermes_cli/test_hooks_cli.py`, `tests/agent/test_shell_hooks.py`,
`tests/cron/test_cron_execution_identity.py`, `tests/hermes_cli/test_plugin_hook_failure_reporting.py`
→ `144 passed, 1 failed, 12 skipped` (`151 passed / 1 failed / 12 skipped` including the two new
files); the one failure
(`TestPluginDiscovery::test_enabled_portable_plugin_registers_components`) also fails on
`81a55437f3` on Windows (portable-plugin `app.darwin.location` validation), i.e. pre-existing and
unrelated.

`python scripts/check` → `12 checks, ok`.

### Docs

Catalog table (three rows) + the `pre_tool_call` parameter table, in `website/docs` and the
`zh-Hans` mirror.

### Disclosure

Drafted with an AI assistant under my direction; I reviewed the diff and ran the checks above.