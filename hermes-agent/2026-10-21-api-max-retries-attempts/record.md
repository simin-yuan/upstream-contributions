# Hermes Agent — `api_max_retries` counts three attempts in total, not four

| | |
|---|---|
| Pull request | [#142179](https://github.com/NousResearch/hermes-agent/pull/142179) |
| Submitted | 2026-10-20 17:47:23Z (2026-10-21 01:47 UTC+08:00) |
| State | open, 3 commits, 19 files, +21/−21; no comment, no review |
| Head | `simin-yuan:docs/api-max-retries-attempts` at `88515e120f` |
| Labels | `type/docs`, `area/config`, `area/i18n`, `P3` |
| Files in this folder | the pull request as filed, and the API payloads behind the timeline |

## What it is

Three places stated `agent.api_max_retries` as a count of *retries on top of* the first call, which
makes the default of `3` read as four requests. The value is the total number of attempts, the first
call included, so the default is three. The PR changes wording only: the paragraph and the sample
comment in `website/docs/user-guide/configuration.md`, the same two in the zh-Hans mirror, and the
`tips.t240` startup tip in all 17 catalogs under `locales/`. No behaviour change, no code touched.

The body argues the semantics from four call sites (`agent/agent_init.py:1444`,
`agent/conversation_loop.py:1672` and `:1510`, `agent/turn_api_error.py:139`) and from
`agent/retry_utils.py:173`, whose docstring already states the equality, and reports a measured
fake-provider run where `api_max_retries: 3` produced 3 streaming attempts in one cycle.

## Timeline

All times UTC as returned by the API; Beijing time is +08:00.

| Time | Event |
|---|---|
| 2026-10-20 17:46:52 | Commit `e6d0336` — "docs(config): api_max_retries counts three attempts total, not four". Author `Simin (Simon) Yuan <yleven120@gmail.com>`. |
| 2026-10-20 17:47:23 | PR #142179 opened against `main`. |
| 2026-10-20 18:06:11 | Commit `8f43fd6` — "docs(config): say attempts, not retries, for api_max_retries". Same author. |
| 2026-10-20 18:17:44 | Commit `88515e1` — "docs(i18n): say fallback, not replacement, in the Korean retry tip". Same author; this is the current head. |
| 2026-10-20 18:23:23 | Last update to the PR record. No comment, no review, no label added by a maintainer. |

## Checks as reported by the API

Every push on this branch produced three workflow runs. On the current head `88515e1`:

```
2026-10-20T18:23:26Z  .github/workflows/ci.yaml           completed  failure         jobs=0
2026-10-20T18:23:27Z  Nix flake check                     completed  action_required jobs=0
2026-10-20T18:23:27Z  Docker Build, Test, and Publish     completed  action_required jobs=0
```

The same pattern holds for `8f43fd6` and for the first head `e6d0336`, and for the four `Label
rerun` runs created between 17:59 and 18:00: conclusion `failure`, `jobs_total=0`. A run with zero
jobs never allocated a runner, so no test ran and no step log exists. The API returns the conclusion
but not a cause; this folder records the conclusion and leaves the cause unstated.

No maintainer has commented, requested a change, or applied `ci-reviewed`.

## Contents

- `pr-body-original.md` — the body as filed
- `snapshot.json` — raw API payloads for the pull request, its commits, its files and every workflow
  run on the branch, pulled on 2026-10-21 17:30 UTC+08:00

## Not covered here

`#104465` corrects two other stated defaults (`TERMINAL_DOCKER_IMAGE`, `HERMES_STREAM_RETRIES`) in
other files. The PR body explains why the two are not duplicates and share no changed line; that
argument is in `pr-body-original.md` and is not restated here.
