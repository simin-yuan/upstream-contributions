# fix(llms): don't stack the sdk's retries on rate-limit replays

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

## Related issue

Closes #8332

## Summary

`run_with_rate_limit_retry` / `arun_with_rate_limit_retry` now record whether the attempt they are about
to run is a **replay** of a call that already failed (`_llm_retry_replay` contextvar, exposed as
`in_rate_limit_retry_replay()`), and the OpenAI provider uses that to build its client:

```python
def _client_for_attempt(self, client: Any) -> Any:
    if in_rate_limit_retry_replay():
        return client.with_options(max_retries=0)
    return client
```

All nine provider call sites in `lib/crewai/src/crewai/llms/providers/openai/completion.py` go through
`_client_for_attempt(...)`: 4 `responses.create`, 3 `chat.completions.create`, and 2
`chat.completions.with_raw_response.create` (in `_handle_completion` and `_ahandle_completion`, i.e. the
non-streaming path).

Why: the two layers multiply today. `max_retries` defaults to `2` at `completion.py:376`, so the SDK makes
up to 3 HTTP attempts per call; CrewAI then retries the same call up to 3 times (`retry.py:35`, wrapper
installed at `base_llm.py:201`/`:215`). Measured on `crewai==1.15.28` against a server that always answers
`429`: **27 byte-identical requests** for one call
(https://github.com/simin-yuan/agent-double-exec-bench, `results/e1/crewai/429/server.jsonl`; the SDK's
`x-stainless-retry-count` cycles 0→1→2 nine times).

## Measured, before and after

Same harness, same fault server, CrewAI default configuration, `Retry-After: 1` on the 429 cell. "Before"
is base commit `dec075b5`, "after" is this branch head:

| cell | before | after |
|---|---|---|
| 429 (`Retry-After: 1`) | 27 chat requests (`x-stainless-retry-count` 0 ×9, 1 ×9, 2 ×9) | **15** (0 ×9, 1 ×3, 2 ×3) |
| 500-neutral | 9 chat requests (0 ×3, 1 ×3, 2 ×3) | 9 (unchanged) |

The 15 is counted in the server log, not derived: the first attempt still spends the SDK's three attempts
(that is the cheap way to ride out a one-off blip), and the two replays CrewAI makes now spend one request
each — 3 rounds × (3 + 1 + 1). The 500 cell is unchanged because CrewAI never replays a 500 at the model
layer, so there is no replay for the SDK's retries to stack on.

Logs: `results/e3/crewai-before/` and `results/e3/crewai-after/` in
https://github.com/simin-yuan/agent-double-exec-bench (`429` and `500-neutral` cells, `server.jsonl`;
`chat/completions` lines only).

## Verification

- [x] Tests added or updated for the changed behavior
- [x] Relevant tests and quality checks pass locally

`lib/crewai/tests/llms/test_retry.py`, six new tests (eight cases):

| test | before the change | after |
|---|---|---|
| `test_replays_of_a_throttled_call_are_marked_as_replays` | fails — sees `[False, False]` | passes |
| `test_async_replays_of_a_throttled_call_are_marked_as_replays` | fails — sees `[False, False]` | passes |
| `test_the_replay_flag_is_off_outside_a_retry` | fails at import | passes |
| `test_openai_provider_turns_sdk_retries_off_on_replay` | fails at import | passes |
| `test_non_streaming_chat_keeps_sdk_retries_for_the_first_attempt_only[True]` | fails — the replay still carries `max_retries=2` | passes |
| `test_async_non_streaming_chat_keeps_sdk_retries_for_the_first_attempt_only[True]` | fails — the replay still carries `max_retries=2` | passes |

The non-streaming pair also runs with `[False]` (first-attempt case) and passes in both directions — that
half is the control: the patch must not touch the first attempt. With only `completion.py` reverted to its
parent version, the run is `2 failed, 2 passed`: the two replay cases fail, the two controls pass.

The replay tests drive the real code path (`BaseLLM.__init_subclass__` wrapper → retry loop) and assert the
sequence `[False, True]`; the provider test asserts the client is asked for `max_retries=0` on a replay and
left untouched otherwise.

Local runs (Windows, Python 3.12, venv with `uv sync --all-groups` plus the provider extras):

```
pytest lib/crewai/tests/llms/test_retry.py     -> 22 passed
pytest lib/crewai/tests/llms/                  -> 918 passed, 18 failed, 20 skipped
pytest lib/crewai/tests/                       -> 5888 passed, 138 failed, 73 skipped, 11 errors
ruff check lib/                                -> All checks passed
ruff format --check lib/                       -> 966 files already formatted
mypy --no-incremental (the two changed files)  -> Success: no issues found in 2 source files
```

The 18 and the 138 are environmental, and I checked that rather than assuming it: running those exact test
ids with `lib/` checked out at `dec075b5` reproduces them 1:1 (18 failed, 14.95 s / 138 failed). The 11
errors are collection errors for optional dependencies that are not installed locally (`a2a`, `qdrant`).
The whole-tree mypy run also needs the full dev extras; on the two files this PR touches it is clean under
the same strict configuration, unchanged from `dec075b5`.

On Windows the repo's `--block-network` addopts blocks the local socket asyncio needs, which makes *every*
async test — pre-existing ones included — error out at setup; the runs above pass `--allowed-hosts=127.0.0.1`
for that reason only. CI on Linux does not have this problem and is the number to trust.

## Additional context

Deliberately not touched here:

- `Agent._times_executed` never being reset (`agent/core.py:242`, `:886-887`) — real, but it is a
  semantics change for `max_retry_limit`, so it is reported in the linked issue instead;
- the agent-level retry itself (`_handle_execution_error` → `execute_task`), which produces the 0.02 s
  re-sends;
- non-OpenAI providers. The same pattern applies to any provider whose SDK retries by default; the hook is
  generic (`in_rate_limit_retry_replay()`), so they can opt in one at a time.

---

Disclosure: I directed this work. The source reading, the patch, its tests and the measurements were
produced by AI assistants I operate. Every number above comes from the server logs in
agent-double-exec-bench, and I will answer review comments on it.

Per CONTRIBUTING this should carry the `llm-generated` label. Contributors can't set labels, so I asked for
it in a comment below.

Related: #7771 handles Retry-After on SDK response objects; this PR changes a different part of the same
files (the SDK's own retries under CrewAI's rate-limit replays). The two do not touch the same lines.


---

## Comment `6297245220`, posted `2026-10-20T17:41:12Z`

Per CONTRIBUTING, this was created with AI assistance; could a maintainer please add the llm-generated label? (Contributors can't set labels.)