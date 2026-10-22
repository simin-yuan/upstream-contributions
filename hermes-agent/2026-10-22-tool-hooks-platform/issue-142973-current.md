### Summary

`pre_llm_call` receives `platform`; the tool-path hooks never do. A plugin that gates tool calls
therefore cannot distinguish an attended turn from an unattended one (`cron`, `subagent`,
`curator`) at the exact moment it has to decide whether to veto or only record.

### Where

All links are pinned to `81a55437f3`.

- [`agent/inline_tool_executors.py#L20`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/agent/inline_tool_executors.py#L20)
  — `tool_hook_ids()` builds the shared identity set (`task_id`, `session_id`, `tool_call_id`,
  `turn_id`, `api_request_id`) and never reads `agent.platform`, although `agent` is its first
  argument.
- [`hermes_cli/plugins.py#L1979`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/hermes_cli/plugins.py#L1979)
  and [`#L1995`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/hermes_cli/plugins.py#L1995)
  — the `pre_tool_call` payload is an explicit keyword list with no `platform`.
- [`model_tools.py#L643`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/model_tools.py#L643)
  — `_CallIds`, which backs `post_tool_call` and `transform_tool_result`, carries the same five
  fields.
- [`hermes_cli/hooks.py#L102`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/hermes_cli/hooks.py#L102)
  — the `hermes hooks test` / `doctor` sample payloads mirror the runtime shape and lack it too.
- [`agent/turn_context.py#L819`](https://github.com/NousResearch/hermes-agent/blob/81a55437f31a9f12a30c53b6c5d49d420898aa/agent/turn_context.py#L819)
  — by contrast, `pre_llm_call` passes it: `platform=getattr(agent, "platform", None) or ""`.

The hook catalog in `website/docs/user-guide/features/hooks.md` shows the same split: `pre_llm_call`,
`post_llm_call`, `transform_llm_output`, `pre_verify`, `pre_api_request`, `post_api_request` list
`platform`; `pre_tool_call`, `post_tool_call`, `transform_tool_result` do not.

### Minimal reproduction — no turn needed

```python
from agent.inline_tool_executors import tool_hook_ids

class Agent:
    platform = "cron"
    session_id = "s"
    _current_turn_id = "t"
    _current_api_request_id = "a"

print(tool_hook_ids(Agent(), "task", "call"))
```

Actual:

```
{'task_id': 'task', 'session_id': 's', 'tool_call_id': 'call', 'turn_id': 't', 'api_request_id': 'a'}
```

Expected: the same dict plus `'platform': 'cron'`.

End to end the same thing is visible: register

```python
def register(ctx):
    ctx.register_hook("pre_tool_call", lambda **kw: print(sorted(kw)))
```

and let a cron job call any tool — `platform` is not in `kw`, even though the same run's
`pre_llm_call` gets it.

### Why it matters

I maintain `provenance-gate` (in the plugin catalog via #141935), which refuses a "decision-shaped"
write unless it can point at a quote the user actually typed in this session. In an attended turn
that is the right default. In a cron run there
is no user message to quote, so every such write would be refused while nobody is watching — the
gate has to run in observe mode there. Without `platform` in the hook the plugin cannot make that
distinction, so it stays in observe mode everywhere, i.e. the check cannot be enabled by default
for anyone. The workaround available today is guessing from `session_id` prefixes or lock files,
which is not something a policy hook should do.

### Scope

The whole class, not just `pre_tool_call`: the same identity set backs `post_tool_call` and
`transform_tool_result`, and tool-side middleware receives the same kwargs, while LLM-side
middleware already gets `platform` (`agent/turn_api_request.py`), so the two sides disagree today.
Shell hooks receive identity fields in `extra` (`agent/shell_hooks.py#L87`), so they pick the fix
up without any extra code.

### Environment

`main` @ `81a55437f31a9f12a30c53b6c5d49d420898aa`, Windows 11, Python 3.14.7.
I have a patch ready and will open the PR right after this issue.