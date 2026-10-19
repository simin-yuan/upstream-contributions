In #108371, resolved by #118841 (merge commit 74f726c17675642c2eff3033632e812f5f0e6614), the parser was described as one that "also reads the legacy `hooks:` spelling for external plugins". The loader does this, but the validator does not.

**Reproduction** (at 0952996a37274bdf298246d6473d10fb3e0bd88c; still reproduces at 67481b6f37615e58f1434ba6a1528e4c6e9b754c):

1. Create a directory with `plugin.yaml`:

```yaml
name: legacy-fixture
version: 1.0.0
description: Legacy hooks key fixture.
hooks:
  - pre_tool_call
```

and `__init__.py`:

```python
def register(ctx):
    ctx.register_hook('pre_tool_call', lambda **kw: None)
```

2. Run `hermes plugins validate <dir>`.

**Expected:** validation passes, since the loader treats `hooks:` as the declared hook list.

**Actual:**

```
✗ declared hooks — undeclared hooks registered (not in provides_hooks): pre_tool_call
Validation failed.
```

**Root cause:** `hermes_cli/plugins_manifest.py:551` resolves declared hooks as `data.get("provides_hooks", data.get("hooks", []))`. `hermes_cli/plugin_validate.py:418` passes only `"provides_hooks"` to `_declared_list` (`hermes_cli/plugin_validate.py:381`), so hooks declared under `hooks:` are counted as undeclared. A plugin that loads and runs correctly fails admission, including the catalog gate.

I have a small fix ready and will open a PR referencing this issue.



**Environment**
- Reproduced on Linux (Python 3.14) and Windows (Python 3.14).
- Hermes: `0952996a37274bdf298246d6473d10fb3e0bd88c`, and still reproducing at `67481b6f37615e58f1434ba6a1528e4c6e9b754c`.

Reproduction on Windows, unmodified main:

```
$ hermes plugins validate <plugin whose plugin.yaml lists its hooks under `hooks:`>
✗ declared hooks — undeclared hooks registered (not in provides_hooks):
Validation failed.
```

With the patch in #141340:

```
✓ declared hooks — matches registrations
⚠ plugin.yaml declares hooks under the legacy `hooks:` key; rename it to provides_hooks
Validation passed.
```
