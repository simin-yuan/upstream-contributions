# Evidence — `hermes-degeneracy-guard` catalog entry

Produced on 2026-10-19, Windows 11, Python 3.14.7. Every command ran against isolated
`HERMES_HOME` directories; nothing was installed into a Hermes that was in use.

## 1. Validation of the pinned commit

Plugin at `ffc28d0ab9ac6b6c0cbb118ed5fb0c06909a3dc1`, version 1.3.0, declared `requires_hermes: ">=0.22.2"`.

```console
$ hermes plugins validate --install-deps <plugin>
→ Installing Python dependencies…
  ✓ Installing Python dependencies
✓ manifest — plugin.yaml parses
✓ manifest fields — name, version, description present
✓ requires_hermes — spec '>=0.22.2' parses
✓ config schema — not declared
✓ requires_env — all entries UPPER_SNAKE
✓ loadable — entry: __init__.py
✓ python dependencies — none declared
✓ capability probe — register() ran in isolation
✓ declared tools — matches registrations
✓ declared hooks — matches registrations
✓ declared middleware — matches registrations
✓ built-in tool collisions — no tools to check
✓ security scan — safe
✓ no core override — no runtime rebinds of Hermes core
◆ Isolation — runs in the plugin host (plugins.isolation: host)

Validation passed.
```

The same command was run a second time against a virtual environment installed from the `v0.22.2`
tag, which is the version floor the entry declares: `Validation passed.`

```console
$ hermes plugins doctor degeneracy-guard --ci
manifest: degeneracy-guard 1.3.0 (standalone)
OK: runtime discovery, manifest parsing, import, and registration passed
registrations: 0 tool(s), 5 hook(s)
```

Same output on the `v0.22.2` environment.

## 2. The catalog itself

```console
$ python scripts/validate_plugin_catalog.py plugin-catalog/
plugin-catalog\agent_memory_es.yaml: warning: unknown top-level key 'homepage_url' (ignored by this validator)
plugin-catalog\skill-integrity.yaml: warning: unknown capabilities key 'provides_commands'
plugin-catalog\stack.yaml: warning: unknown top-level key 'homepage_url' (ignored by this validator)
plugin-catalog\vornik-companion.yaml: warning: unknown top-level key 'requires_env' (ignored by this validator)
OK: 705 file(s) valid
```

The four warnings belong to pre-existing entries, not to this one.

Catalog-related test files (the six that touch the catalog and the plugin validator):

```console
$ pytest tests/hermes_cli/test_plugin_catalog.py tests/hermes_cli/test_plugin_catalog_presence.py \
         tests/hermes_cli/test_plugin_catalog_known_issues.py tests/hermes_cli/test_plugins_cmd_catalog.py \
         tests/ci/test_plugin_catalog_ci_confinement.py tests/hermes_cli/test_plugin_validate.py -q
25 failed, 44 passed, 29 skipped
```

The same six files run against unmodified `main` (`eea83bb6`) in this environment give the same
counts — `25 failed, 44 passed, 29 skipped`. The failures are missing optional dependencies here,
not a consequence of the entry.

```console
$ python scripts/check --commit HEAD --base upstream/main
scripts/check: 12 checks, ok
```

## 3. The plugin's own test suite, at the pinned commit

```console
$ pytest tests -q          # no Hermes importable
36 passed, 1 skipped in 0.46s

$ pytest tests -q          # Hermes at main (22063fc0), editable install
37 passed in 3.33s

$ pytest tests -q          # Hermes at v0.22.2
37 passed in 3.99s
```

## 4. What happens at the approval gate

Driven through Hermes' real plugin manager with a temporary `HERMES_HOME`, a three-sentence cycle
repeated thirty times as the argument of a `write_file` call:

```console
action      : approve
rule_key    : degeneracy-guard:write_file
message     : DEGENERACY_GUARD_APPROVAL: these arguments look like a repetition loop rather than
              data (periodic_repeat: 3-line block repeated 30 times (100% of a 90-line argument):
              'The answer is below.'). No data shape was found in the payload, so this guard cannot
              tell an intentional repeat from a stuck generation — that call is yours to make.
```

The four answers, with the gate's own output:

| Answer | Result |
|---|---|
| deny | `BLOCKED: User denied this potentially dangerous action (matched 'DEGENERACY_GUARD_APPROVAL: …'). Do NOT retry — the user has explicitly rejected it.` |
| allow once | call runs |
| always | call runs, and the same tool is not asked again |
| no human present (separate process, no interactive context) | `BLOCKED: Tool 'edit_file' requires approval (…) but no interactive user or gateway is present to approve it. A plugin flagged this action for human confirmation.` |

An ordinary write with normal content produced no prompt: `(None, None)`.

The incident file written for each of these:

```json
{
  "action": "approve_requested",
  "approval_outcome": "not observable from this hook",
  ...
}
```

The outcome is recorded as unobservable because `pre_tool_call` returns the request and does not
see the answer. That is written as it is rather than guessed.

## 5. Measured detection numbers

Two hold-out sets, both scored once, neither used to tune a rule afterwards.

| Set | Written by | Legitimate payloads | Asked | Degeneracy samples | Caught |
|---|---|---|---|---|---|
| 1 | someone who had not seen the detection rules | 40 | 0 | 13 | 6 |
| 2 | the reviewer, after set 1 | 12 | 6 | 5 | 3 |

Set 1 legit covers SQL, CSV, TSV, JSON, JSONL, YAML, TOML, INI, env, properties, proto, Terraform,
HTML, XML, SVG, CSS, JS, Python, diff, hexdump, base64, licence text, changelog, runbook,
fixed-width table, ASCII art, box drawing. Set 2 legit covers Dockerfile RUN lines, shell echo
lines, INI sections, reStructuredText and LaTeX tables, a hex dump and others.

The seven misses in set 1: a markdown list of 68 identical items, a JSON record cycle, a YAML
document cycle, a counter-only line, a numbered list repeating the same content, a paragraph
repeated with small in-between variations, and a short "let me check / wait / actually" cycle.
They are named in the plugin's README as well; the short version is that near-repetition and
data-shaped repetition sit under the thresholds on purpose, because asking on every
near-repetition is what version 1.1.0 did wrong.

## 6. CI on the pinned commit

`.github/workflows/tests.yml` on `ffc28d0` — run [39220257775](https://github.com/simin-yuan/hermes-degeneracy-guard/actions/runs/39220257775):
`completed / success` across the Python matrix. `hermes-latest.yml` against Hermes `main` was
green on the previous commit, before the README-only change.
