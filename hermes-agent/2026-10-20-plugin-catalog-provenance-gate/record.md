# Hermes Agent — catalog entry for `provenance-gate` (0.1.0)

| | |
|---|---|
| Pull request | [#141935](https://github.com/NousResearch/hermes-agent/pull/141935) |
| Submitted | 2026-10-20 15:47 (UTC+08:00) — `07:47:39Z` |
| State | merged 2026-10-21 03:20 (UTC+08:00) — `2026-10-20T19:20:46Z` |
| Merged by | @teknium1 |
| Commits on `main` | [`d21713a1`](https://github.com/NousResearch/hermes-agent/commit/d21713a1beac195f444749cf36261c9ddc895565) — entry, author `Simin (Simon) Yuan <yleven120@gmail.com>`; [`23f0ed0`](https://github.com/NousResearch/hermes-agent/commit/23f0ed0f60776bcb9e64eb8779d2e3c5aacbd4e4) — maintainer follow-up, author `@teknium1` |
| Labels | none at filing; `comp/plugins`, `P3`, `tool/memory`, `type/feature` added by @alt-glitch, `ci-reviewed` added by @teknium1 |
| Entry commit | `e1fa5ad03832cfe21a6ea41cfa0a1d635e60471b` (`simin-yuan:feat/plugin-catalog-provenance-gate`) |
| Pinned plugin commit | `8813469d45c82fb0f5af6724c3761169ca7c3e86` (plugin version 0.1.0) |
| Diff | 2 files, +31 / −0 |

## What it is, and why it matters

An agent's memory is written by the agent, about the user, and nothing in a normal write path checks
it against what the user actually said. A line like "the user decided to ship on Friday" can be
recorded, reinforced by every later read, and never contradicted — the user never sees it happen.
`provenance-gate` closes that gap at the write: a memory or `skill_manage` write that states a user
decision must carry a quote that appears verbatim in text the user typed this session, shares key
words with the decision, and does not drop a negation. A patch is judged by the text it produces,
not by the shape of the call.

Observe mode is the default — flagged writes are recorded and `hermes provenance-gate report` lists
them. Block mode refuses the write and returns instructions the model can act on. The plugin keeps a
local copy of the user's typed text and an evidence log under `plugin-data/provenance-gate` (30-day
default retention, `hermes provenance-gate purge`), has no network access and no credentials, and
never asks for approval.

The precision/recall pair in the entry and in the PR body is not this repository's own measurement.
It was produced on 133 lines a reviewer wrote and labelled *before* reading the code, and the misses
are published alongside it. That is the number a reader should try to break.

## How the public repository came to exist

The plugin was developed in a private repository. Before publication, a scan of the **entire history**
— not just the current tree — found environment-specific strings in a test corpus file that would
have identified the author's machine and setup. Rebuilding the history was cheaper than leaving them
in place, and publication is not reversible once a clone exists.

The original private repository was therefore renamed `hermes-provenance-gate-private` and kept,
private and unmodified, as an archive; a new public repository was created under the same original
name and given the rebuilt history. Nothing was deleted and no pushed history was rewritten. The
rebuilt tip `8813469` differs from the last private revision only in those corpus lines.

## Timeline

| Time (UTC) | Event |
|---|---|
| 2026-10-20 07:37 | Private repository renamed to `hermes-provenance-gate-private` (PRIVATE, kept as archive). |
| 2026-10-20 07:38 | Public repository created under the original name; rebuilt history `8813469` pushed as `main`. |
| 2026-10-20 07:47:24 | Entry commit `e1fa5ad038` on `feat/plugin-catalog-provenance-gate`, branched from `NousResearch:main`. |
| 2026-10-20 07:47:39 | PR #141935 opened against `main`, two files: the catalog entry and the contributor-email mapping. |
| 2026-10-20 08:28:38 | @whyyagswhy commented: "Verified head SHA e1fa5ad0 matches PR (fetched to refs/review/141935). Catalog validation suite (test_plugin_catalog.py + test_plugin_catalog_presence.py + test_plugin_catalog_ci_confinement.py) via scripts/run_tests.sh: 45 passed, 0 failed." That run covers the branch head as it stood at the time, not the later maintainer commit. |
| 2026-10-20 18:28:22 | @teknium1 pushed `2517de8` — "chore(catalog): review disclosure for provenance-gate" — onto the branch. The commit changes one line: `category: memory` → `category: general`. Author, repository URL, description and disclosure text are unchanged. |
| 2026-10-20 18:51:02 | @teknium1 labeled it `ci-reviewed`. |
| 2026-10-20 19:20:46 | Merged by @teknium1. `main` received `d21713a1` and then `23f0ed0`, each keeping its own author, and `main`'s `plugin-catalog/provenance-gate.yaml` is byte-identical to the branch head `2517de8`. |

## Checks

Public-repository CI at `8813469`: `success` (run `39387728408`). The PR's own checks were empty at
filing — GitHub holds workflows for a first-time contributor until a maintainer approves them. That
was the approval gate, not a failure.

On the merged branch head `2517de8`, GitHub reports 40 check runs: 17 `success`, 23 `skipped`, 0
failures. The merge commit `23f0ed0` carries a single run, `probe`, `success`. `main`'s entry file is
byte-identical to the branch head (`sha256` of the blob matches on both sides).

`scripts/validate_plugin_catalog.py` reports `OK: 1 file(s) valid` against the entry as committed.

## Contents

- `pr-body-original.md`, `pr-141935-current.md`
- `provenance-gate.yaml` — the entry as filed (pinned to `8813469`, `category: memory`)
- `provenance-gate-merged.yaml` — the entry as it stands on `main` after the merge; differs from the
  filed copy by one line, `category: memory` → `category: general`
- `snapshot.json`, `pr-files-141935.json`, `entry-contents-api.json` — raw API payloads
- `patch/0001-add-provenance-gate-catalog-entry.patch` — the entry commit only
- `evidence.md` — reproduction steps and the output produced while the work was done
- `archive.md` — Internet Archive snapshots
