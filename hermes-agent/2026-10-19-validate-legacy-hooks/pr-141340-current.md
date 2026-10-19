## What does this PR do?

The loader reads declared hooks from `provides_hooks` and falls back to the legacy `hooks:` key (#108371). `hermes plugins validate` read only `provides_hooks`, so an external plugin using `hooks:` loaded fine but failed validation with "undeclared hooks registered (not in provides_hooks)".

This PR moves the resolution rule into `hermes_cli.plugins_manifest.hooks_manifest_key()`, and both the loader and the validator use it. Loader behaviour is unchanged. When only the legacy key is present, validation passes and adds a warning recommending `provides_hooks`. The `provides_tools` and `provides_middleware` checks and the bundled manifests are untouched. The second commit maps my contributor email, which the contributor check requires.

## Related Issue

Fixes #141339

## Type of Change

- [x] 🐛 Bug fix (non-breaking change that fixes an issue)
- [ ] ✨ New feature (non-breaking change that adds functionality)
- [ ] 🔒 Security fix
- [ ] 📝 Documentation update
- [ ] ✅ Tests (adding or improving test coverage)
- [ ] ♻️ Refactor (no behavior change)
- [ ] 🎯 New skill (bundled or hub)

## Changes Made

- `hermes_cli/plugins_manifest.py` — new `hooks_manifest_key(data)`: `provides_hooks`, else the legacy `hooks:` list, else the default.
- `hermes_cli/plugin_validate.py` — the "declared hooks" check resolves the key through that helper instead of reading `provides_hooks` directly; a legacy-only manifest passes and gets a warning.
- `tests/hermes_cli/test_plugin_validate.py` — `test_legacy_hooks_key_declares_hooks_like_the_loader` covers the loader-equal path.
- `contributors/emails/yleven120@gmail.com` — contributor email mapping.

The loader path in `hermes_cli/plugins_manifest.py` now calls the same helper, so the two sides cannot drift again.

## How to Test

- `scripts/run_tests.sh tests/hermes_cli/test_plugin_validate.py`: the new test `test_legacy_hooks_key_declares_hooks_like_the_loader` fails on main and passes with this change.
- Manually: run `hermes plugins validate` on a plugin whose `plugin.yaml` lists its hooks under `hooks:`. The "declared hooks" check passes and a warning recommends `provides_hooks`.
- `python scripts/check` passes.

Platforms tested: Linux and Windows.

Note on the test file: `tests/hermes_cli/test_plugin_validate.py` also contains `test_portable_validation_fails_orphan_and_reports_availability`, which fails identically on unmodified main in my local environment (a missing optional dependency); it is unrelated to this change and was failing before it.

## Checklist

### Code

- [x] I've read the [Contributing Guide](https://github.com/NousResearch/hermes-agent/blob/main/CONTRIBUTING.md)
- [x] My commit messages follow [Conventional Commits](https://www.conventionalcommits.org/) (`fix(plugins):`, `chore:`)
- [x] I searched for [existing PRs](https://github.com/NousResearch/hermes-agent/pulls) to make sure this isn't a duplicate
- [x] My PR contains **only** changes related to this fix/feature (no unrelated commits)
- [ ] I've run `pytest tests/ -q` and all tests pass — run for the touched file instead: 27 passed, plus the one pre-existing failure described above that also fails on main. I am not claiming a green full suite from this machine.
- [x] `python scripts/check` passes (the blocking lint and code-health checks CI runs) — `python scripts/check --commit HEAD --base upstream/main`: 12 checks, ok
- [x] I've added tests for my changes (required for bug fixes, strongly encouraged for features)
- [x] I've tested on my platform: Windows 11

### Documentation & Housekeeping

- [x] I've updated relevant documentation (README, `docs/`, docstrings) — or N/A (the warning message itself tells the author to use `provides_hooks`)
- [x] I've updated `cli-config.yaml.example` if I added/changed config keys — or N/A
- [x] I've updated `CONTRIBUTING.md` or `AGENTS.md` if I changed architecture or workflows — or N/A
- [x] I've considered cross-platform impact (Windows, macOS) per the [compatibility guide](https://github.com/NousResearch/hermes-agent/blob/main/CONTRIBUTING.md#cross-platform-compatibility) — or N/A (no platform-specific code)
- [x] I've updated tool descriptions/schemas if I changed tool behavior — or N/A

## Screenshots / Logs

```
$ pytest tests/hermes_cli/test_plugin_validate.py -q
27 passed, 1 failed        # the failure is test_portable_validation_fails_orphan_and_reports_availability,
                            # which fails the same way on unmodified main here

$ python scripts/check --commit HEAD --base upstream/main
scripts/check: 12 checks, ok
```
