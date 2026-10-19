## What does this PR do?

Wires the existing `RunawayStreamWatch` (#130551) into the Bedrock Converse and Codex Responses streams, so a reply that turns into a repetition loop is cut where it starts instead of streaming to the output limit. Both cuts hand off the same way the other two streams do: the response carries `_runaway_repetition`, and the length path ends the turn on it. No new detection logic.

## Related Issue

Fixes #141477

## Type of Change

- [x] 🐛 Bug fix (non-breaking change that fixes an issue)

## Changes Made

- `agent/bedrock_adapter.py`: `stream_converse_with_callbacks(..., watch_runaway=False)`. `_ConverseRunawayWatch` wraps the event iterator, so the consumer loop gains no branches; a cut returns what streamed so far with `stop_reason=max_tokens`. A tool call cut mid-arguments is dropped, including its content block, so no `toolUse` is left without a `toolResult`. The default is unchanged; only the live stream path opts in.
- `agent/chat_completion_helpers.py`: `_BedrockStream` passes `watch_runaway=True` (one line).
- `agent/codex_runtime.py`: text and reasoning deltas feed the watch. Two details: a reasoning-only cut no longer raises "did not emit a terminal response", and a cut skips `_drain_for_finalizer`, which would otherwise keep reading the loop it just stopped.
- `agent/turn_response_check.py`: a marked Codex response routes to the length path (two lines, same shape as #91770).
- `tests/agent/test_stream_runaway_repetition_bedrock_codex.py`: new.

## How to Test

1. `pytest tests/agent/test_stream_runaway_repetition_bedrock_codex.py -q` — 15 passed with this patch. On unmodified `2e83e2a`, the same file gives 12 failed, 3 passed. The 3 that pass there are the two "legitimate repetition on Codex is delivered untouched" cases and the coverage map below.
2. The tests cover text and reasoning loops on both paths, a loop after ~100K of legitimate text, asked-for repetition and distinct rows (both delivered untouched), the live Bedrock dispatch through `interruptible_streaming_api_call` (provider stream closed, response marked), and Codex's routing to the length path.
3. `test_every_continuable_stream_mode_has_a_runaway_cut` fails if a mode is added to `_CONTINUABLE_MODES` without a mid-stream cut and a test.
4. `python scripts/check` — 12 checks, ok.

## Checklist

### Code

- [x] I've read the Contributing Guide
- [x] My commit messages follow Conventional Commits
- [x] I searched for existing PRs to make sure this isn't a duplicate
- [x] My PR contains only changes related to this fix
- [ ] I've run `pytest tests/ -q` and all tests pass — I ran the Bedrock, Codex, stream-runaway, repetition and truncation suites at `2e83e2a` (215 tests: 214 passed, 1 skipped), not the whole `tests/` tree.
- [x] `python scripts/check` passes (12 checks)
- [x] I've added tests for my changes
- [x] I've tested on my platform: Windows 11, Python 3.11.16

### Documentation & Housekeeping

- [x] Documentation: N/A (the providers docs already describe this behavior; this makes Bedrock and Codex match it)
- [x] `cli-config.yaml.example`: N/A
- [x] `CONTRIBUTING.md` / `AGENTS.md`: N/A
