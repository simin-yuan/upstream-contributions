# Evidence — `platform` on the tool-path hooks

Everything below was run on Windows 11, Python 3.14.7, in a worktree-based checkout of
`NousResearch/hermes-agent` at `81a55437f3`, with the patch applied on top for the "after" runs.
Raw output is kept verbatim next to this file; the numbers here are copied from it.

One redaction: the raw pytest output carried this machine's absolute paths — the checkout directory
and the temporary test directories — so in the four log files those are replaced with `<worktree>`,
`<base-worktree>`, `<tmp>` and `<home>`. Nothing else in the output was touched; the
`rootdir`/`tmp_path` lines are still there, just without the paths.

## 1. The tests fail on the base commit and pass with the patch

Two new files: `tests/agent/test_tool_hook_platform.py` (6 tests) and
`tests/cron/test_cron_tool_hook_platform.py` (1 end-to-end test that ticks a real cron job and reads
what the hook saw).

On `81a55437f3` without the patch — `tests-before.txt`:

```
=== 2 files with test failures (7 tests failed) ===
  tests\agent\test_tool_hook_platform.py  (6 tests failed)
  tests\cron\test_cron_tool_hook_platform.py  (1 test failed)
```

The failures are the shape you want from a negative control, not an import error: `KeyError:
'platform'` where the payload is read, `TypeError: unexpected keyword argument` where the hook is
invoked, and in the cron case

```
>       assert seen["platform"] == "cron"
E       AssertionError: assert None == 'cron'
```

With the patch — `tests-after.txt`:

```
=== Summary: 2 files, 7 tests passed, 0 failed (100% complete) in 16.5s (12 workers) ===
```

One of the six started out passing on the base commit as well, which means it was not testing the
fix. It was rewritten to go through the whole chain (`tool_hook_ids()` → payload → shell-hook field
mapping) and only then did it turn red on the base.

## 2. `scripts/check` clean, including file-length limits

`check-after.txt`:

```
=== health (ok) ===
code health: 8 files vs 81a55437f3f3: 0 blocking, 0 advisory (7.9s)

scripts/check: 12 checks, ok
```

It failed the first time: the repo caps file length, and adding code pushed
`agent/agent_runtime_helpers.py` to 3809 lines against a limit of 3807 and `hermes_cli/plugins.py` to
2236 against 2235. Rather than raise the limits, `_pre_tool_block_message()` was changed to reuse
`tool_hook_ids()` instead of hand-building the same five identity fields, which took three lines out;
the `plugins.py` signature was kept on one line. Net growth is zero or negative in both files.

## 3. Every place that builds a tool-hook payload was checked

`audit_callsites.txt` is the output of a scan over `agent/` and `hermes_cli/` for the three identity
constructors (`tool_hook_ids`, `_CallIds`, `_build_hook_payload`) and for the three hook emissions.
It covers 7 `tool_hook_ids` call sites, 3 `_CallIds` constructions, 3 emission points, and the
tool-request and tool-execution middleware on both sides. Each was confirmed to carry `platform`
after the patch. This is the check that would catch the patch being half done.

## 4. What a real run shows

A stub plugin was loaded into a real Hermes home and a real cron job was ticked with the patched
code; the hook wrote what it received to `seen.json`:

- without the patch: six fields, no `platform`
- with the patch: `"platform": "cron"`

The same run is what `tests/cron/test_cron_tool_hook_platform.py` asserts, end to end, through
`cron/scheduler.py` rather than by calling the hook directly.

## 5. Regression suite

`tests-regression-after.txt`, five files that touch plugin loading, hook dispatch and cron execution:

```
=== Summary: 5 files, 144 tests passed, 1 failed, 12 skipped (100% complete) in 54.0s (12 workers) ===
```

The one failure is `tests/hermes_cli/test_plugins.py::TestPluginDiscovery::test_enabled_portable_plugin_registers_components`,
an existing Windows-only failure: the portable plugin fixture validates a `app.darwin.location` value
that cannot resolve on Windows. It fails the same way on `81a55437f3` without the patch —
`test-plugins-portable-base.txt` in this folder is that run.

## 6. What could not be run

The repository's own entry point, `scripts/run_tests.sh`, does not start here: it pins `uv 0.12.3`
and downloads it, and that download is blocked on this network. The runs above used the repo's
internal `scripts/run_tests_parallel.py` with `TZ=UTC LANG=C.UTF-8 PYTHONHASHSEED=0` instead. This
is stated in the PR body as well, rather than presented as if the official entry point had run.

No CI result exists to report. The three workflows on the head commit are at `action_required`,
waiting for a maintainer to approve the run.
