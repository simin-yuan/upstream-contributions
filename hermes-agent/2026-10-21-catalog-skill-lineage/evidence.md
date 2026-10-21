# Evidence

## The plugin builds, validates and runs

All four commands below were run on `7cc92a5`, the pinned commit.

```
$ hermes plugins validate .
Validation passed.

$ python -m pytest -q                  # Python 3.11.16
SKIPPED [1] tests/test_e2e_hermes.py:63: Hermes runs on Python 3.14 only (pyproject.toml); this is 3.11.16
46 passed, 1 skipped in 1.04s

$ python -m pytest -q                  # Python 3.14.7
SKIPPED [1] tests/test_e2e_hermes.py:63: Hermes is not importable (run_agent not found)
46 passed, 1 skipped in 1.24s
```

The skip is `tests/test_e2e_hermes.py`, which needs a Hermes importable by the running interpreter.
It skips with a printed reason rather than passing silently — which is why the end-to-end run below
exists at all.

## End to end, in a running Hermes

Because both test runs skip the end-to-end suite, the plugin was exercised in an isolated
`HERMES_HOME` — plugin adopted and enabled, then one real one-shot session that created a skill
through `skill_manage`, followed by the six commands:

```
status      -> chain: 2 entries, head seq 2 at 2026-10-21T05:05:43Z; 1 skill versions
trace       -> skill smoke-probe @00f513a524 (seq 2) <- src:user:26edf93a… [user] [strong, 5 shingles]
descendants -> LIVE skill smoke-probe @00f513a524 [strong, 5 shingles]
report      -> no skill versions with untraceable operative tokens
verify      -> INTACT: 2 entries, head seq 2 hash c3a26bf9…c0c
head        -> seq 2 hash c3a26bf9…c0c
verify --head -> INTACT; given head found; 0 entries follow it
```

The full log, including the tool call that was refused for a 78-character description before the
38-character one succeeded, is `tests/smoke-2026-10-21.txt` in the plugin repository.

## The rule-9 fix

```
$ hermes plugins validate .            # before the fix
⚠ private core imports (catalog rule 9 …) skill_lineage/skills.py:20
  tools.skills_tool.locate_skill / skill_search_dirs
```

`_hermes_locate()` was deleted and `locate()` now returns `_plain_locate(name)`, which walks
`$HERMES_HOME/skills` (including `skills/<category>/<name>`). All five real skills on this machine
resolve through it. Diff: `33631f8`, `+3 −16`, one file.

## What the record does not claim

The link between a source and an added line is computed from shared content plus session order. It
is not a proof of causation, and `report` exists because some lines have no traceable origin — that
is surfaced rather than smoothed over. Hashes are keyed per install, so a third party cannot
re-verify another machine's links; `verify` checks internal consistency, and `verify --head` checks
against a value recorded elsewhere.
