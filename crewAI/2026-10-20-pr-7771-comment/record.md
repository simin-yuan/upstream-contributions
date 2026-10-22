# crewAI — Comment with measurements on someone else's retry-after patch

| | |
|---|---|
| Pull request | [#7771](https://github.com/crewAIInc/crewAI/pull/7771) — opened by `wangtaotaotao95`, not by us |
| Our comment | `6290556192`, posted `2026-10-20T12:12:14Z` |
| State at capture | open; unlabelled; 3 comments; 2 files changed |
| Files in this folder | the PR body, our comment, the raw API payloads, and this record |

## What it is

This is a comment, not a contribution of code. The PR is someone else's work: it teaches CrewAI's
retry helpers to honour a numeric `Retry-After` on SDK response objects, not only on plain
dictionaries.

Our comment adds a measurement taken while reviewing that behaviour on CrewAI 1.15.28 at default
settings: against a local fake server that answers every request with `429` and `Retry-After: 3600`,
the client still makes nine requests within about ten seconds, with the gaps recorded individually.
The point of the comment is that a `Retry-After: 3600` hint is not just ignored — it is replaced by
a fast exponential ladder, which is worse than waiting.

No patch of ours is attached and no claim is made on the PR. Keeping it here is about the number:
it is a public measurement under someone else's patch, and if that patch lands, the number becomes
part of why.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-09-26T04:19:51Z` | PR opened by `wangtaotaotao95`. |
| `2026-09-26T04:20:19Z` | `coderabbitai[bot]` posts an automated summary. |
| `2026-10-02T02:16:41Z` | Author refreshes the branch onto `main` `8078f913` as `e85f588b`, and records that a previously failing CI shard was telemetry, not the retry header logic. |
| `2026-10-20T12:12:14Z` | Our comment, `6290556192`: the nine-requests-in-ten-seconds measurement. |
| `2026-10-22T13:50:49Z` | Capture. Open, no maintainer response. |

## What would count as success

The PR merges with the `Retry-After` hint honoured. Whether our number caused that is not something
this record claims.

## Files in this folder

- `body-current.md` — the PR body and our comment as they stand on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json`, `api-files.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
