# Evidence — Bedrock and Codex runaway cut

Everything below is output produced on this machine on 2026-10-19, in the order shown. Nothing is
reconstructed from memory; where a run was limited, the limit is stated.

Environment: Windows 11, Python 3.11.16 (`uv` cpython build), git 2.53.0. A shallow clone of
`NousResearch/hermes-agent` at `2e83e2a935ee04f606aab58e498d34ca58d84c07` (2026-10-19 01:54 -0700),
plus a `git worktree` of the same commit that stays unmodified for the before/after comparison.

## 1. Reproduction, unmodified upstream `main`

Two scripts, ~141,000 characters of one repeated line (`"I will now check the configuration file
again.\n"` × 2 × 1500), fed to the guard itself and to each consumer.

```console
$ python hermes_bedrock_unit_repro.py
guard stops after chars: 16074
bedrock consumed chars: 141000 of 141000

$ python hermes_codex_unit_repro.py
codex consumed chars: 141000 of 141000
```

The guard, given the same text directly, stops at 16,074 characters. Neither consumer asks it.

## 2. The new test file, with the patch applied

```console
$ python -m pytest tests/agent/test_stream_runaway_repetition_bedrock_codex.py -q
...............                                                          [100%]
15 passed in 3.96s
```

## 3. The same test file on unmodified `main`

The file is kept, the four source files are reverted.

```console
$ git checkout -- agent
$ python -m pytest tests/agent/test_stream_runaway_repetition_bedrock_codex.py -q
...
12 failed, 3 passed in 2.76s
```

The three that pass there are the two "legitimate repetition on Codex is delivered untouched" cases
and the coverage-map test. The other twelve fail because the watch is not wired: the Bedrock calls
raise on the unknown `watch_runaway` keyword, the Codex loop tests read all 141,000 characters, and
the marked-response routing does not exist.

## 4. The upstream gate

```console
$ python scripts/check
=== health (ok) ===
code health: 5 files vs 2e83e2a935ee: 0 blocking, 0 advisory (8.9s)

scripts/check: 12 checks, ok
```

## 5. The neighbouring suites, with the patch applied

```console
$ python -m pytest tests/agent/test_bedrock*.py tests/agent/test_codex*.py \
    tests/agent/test_stream_runaway_repetition*.py tests/agent/test_repetition_guard.py \
    tests/agent/test_continuation_repetition_guard.py \
    tests/agent/test_anthropic_truncation_continuation.py \
    tests/agent/test_run_agent_codex_responses.py -q --tb=no
214 passed, 1 skipped in 94.06s (0:01:34)
```

`pytest tests/ -q` in full was not run here; the suites above are what was exercised.

## 6. The benchmark artifact, executed rather than read

`agent-runaway-bench` was run on this machine before publishing, and the run exposed a defect in the
published server: on Windows a client hang-up raises `ConnectionAbortedError` (WinError 10053), which
`loop_server.py` did not catch, so the per-request log line — the whole measurement — was never
written. Before the fix, the request logged a traceback and no row; after it:

```console
$ curl -s -N http://127.0.0.1:8767/v1/chat/completions -d '{"model":"m","messages":[]}' | head -c 300
$ cat log.jsonl
{"path": "/v1/chat/completions", "client_cap": null, "chars_streamed": 564, "client_hung_up_early": true, "secs": 0.02}
```

The published `loop_server.py` carries that fix; the catch is `(BrokenPipeError, ConnectionResetError,
ConnectionAbortedError)`.
