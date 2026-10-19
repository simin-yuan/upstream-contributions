**What and why**

The loader reads declared hooks from `provides_hooks` and falls back to the legacy `hooks:` key (#108371). `hermes plugins validate` read only `provides_hooks`, so an external plugin using `hooks:` loaded fine but failed validation with "undeclared hooks registered (not in provides_hooks)".

This PR moves the resolution rule into `hermes_cli.plugins_manifest.hooks_manifest_key()`, and both the loader and the validator use it. Loader behaviour is unchanged. When only the legacy key is present, validation passes and adds a warning recommending `provides_hooks`. The `provides_tools` and `provides_middleware` checks and the bundled manifests are untouched.

The second commit maps my contributor email (`contributors/emails/`), which the contributor check requires.

**How to test**

- `scripts/run_tests.sh tests/hermes_cli/test_plugin_validate.py`. The new test `test_legacy_hooks_key_declares_hooks_like_the_loader` fails on main and passes with this change.
- Manually: run `hermes plugins validate` on a plugin whose `plugin.yaml` lists its hooks under `hooks:`. The "declared hooks" check passes and a warning recommends `provides_hooks`.
- `python scripts/check` passes.

**Platforms tested**

Linux.

Fixes #141339
