# Evidence — `hooks:` manifest key

Everything below is output that was produced on this machine on 2026-10-19, in the order shown.
Nothing is reconstructed from memory; where a command failed, the failure is kept.

## 1. Reproduction, unmodified upstream `main`

Environment: Windows 11, Python 3.14.7, a virtual environment with Hermes installed in editable
mode from an upstream checkout at `22063fc099ec9ad8357d7f8f6e6248a0400680d3`, an isolated
`HERMES_HOME` under a temporary directory.

Plugin under test — `plugin.yaml` declaring its hook under the legacy key:

```yaml
name: legacyhooks
version: "0.1.0"
description: repro plugin using the legacy hooks key
hooks:
  - pre_llm_call
```

```console
$ hermes plugins validate <path to that plugin>
✓ manifest — plugin.yaml parses
✓ manifest fields — name, version, description present
...
✓ declared tools — matches registrations
✗ declared hooks — undeclared hooks registered (not in provides_hooks):
✓ built-in tool collisions — no tools to check
✓ security scan — safe
✓ no core override — no runtime rebinds of Hermes core
◆ Isolation — runs in the plugin host (plugins.isolation: host)

Validation failed.
```

The manifest parses, the plugin imports, the hook registers — and admission fails on the
declaration check alone. That is the defect: it is not a broken plugin, it is two reads of the
same manifest that disagree.

## 2. Same plugin, same machine, patched source

A second virtual environment, installed from the branch in this folder:

```console
$ hermes plugins validate <same path>
✓ declared hooks — matches registrations
✓ security scan — safe
✓ no core override — no runtime rebinds of Hermes core
⚠ plugin.yaml declares hooks under the legacy `hooks:` key; rename it to
provides_hooks
◆ Isolation — runs in the plugin host (plugins.isolation: host)

Validation passed.
```

Validation passes, and the author is told which spelling to move to.

## 3. Tests

```console
$ pytest tests/hermes_cli/test_plugin_validate.py -q -k legacy_hooks
1 passed, 27 deselected in 4.21s

$ pytest tests/hermes_cli/test_plugin_validate.py -q
FAILED tests/hermes_cli/test_plugin_validate.py::test_portable_validation_fails_orphan_and_reports_availability
1 failed, 27 passed, 3 warnings in 29.12s
```

The failure in the second run is not caused by this change: running that single test in a checkout
of unmodified `main` (`eea83bb6`) fails the same way in this environment.

```console
$ pytest tests/hermes_cli/test_plugin_validate.py::test_portable_validation_fails_orphan_and_reports_availability -q   # on main, no patch
1 failed in 5.71s
```

The repository's own wrapper, `scripts/run_tests.sh`, could not be used here — it stages a pinned
`uv` from its own mirror and that download failed on this machine:

```console
→ Staging pinned uv 0.12.3 (win32-x64) into the pm store...
curl: (23) client returned ERROR on write of 15158 bytes
✗ failed to download pinned uv ...
setup failed
```

So the numbers above come from `pytest` run directly against the same virtual environment, not
from the wrapper.

## 4. Lint and code-health lane

```console
$ python scripts/check --commit HEAD --base upstream/main
...
scripts/check: 12 checks, ok
```

Run after the branch was rebased onto the current `main`. Before the rebase the code-health
ratchet reported three files as grown; that was an artefact of comparing an older base against a
`main` that has since shrunk those files, not something this change touched.

## 5. Upstream references

| What | Commit / reference |
|---|---|
| The legacy key was added, still supported by the loader | merge `74f726c17675642c2eff3033632e812f5f0e6614` (#118841) |
| Loader fallback line | `hermes_cli/plugins_manifest.py` |
| Validator read only `provides_hooks` | `hermes_cli/plugin_validate.py` (`_declared_list`) |
| Checkout used for the before/after | `22063fc099ec9ad8357d7f8f6e6248a0400680d3` |
| Tag used for version-gate checks elsewhere | `v0.22.2` |

## 6. CI

The workflows on the pull request sit at `completed / action_required` with zero check-runs: GitHub
requires a maintainer to approve a first-time contributor's workflow run. This is the approval
gate, not a red build.
