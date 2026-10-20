# Hermes Agent — catalog entry for `provenance-gate` (0.1.0)

| | |
|---|---|
| Pull request | [#141935](https://github.com/NousResearch/hermes-agent/pull/141935) |
| Submitted | 2026-10-20 15:47 (UTC+08:00) — `07:47:39Z` |
| State | open, ready for review, mergeable; CI held at the first-time-contributor approval gate |
| Labels | none at filing |
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

## Checks

Public-repository CI at `8813469`: `success` (run `39387728408`). The PR's own checks are empty at
filing — GitHub holds workflows for a first-time contributor until a maintainer approves them. That
is the approval gate, not a failure, and it is not something to work around.

`scripts/validate_plugin_catalog.py` reports `OK: 1 file(s) valid` against the entry as committed.
