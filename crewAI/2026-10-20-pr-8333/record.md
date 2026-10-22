# crewAI — Don't stack the SDK's retries on rate-limit replays

| | |
|---|---|
| Pull request | [#8333](https://github.com/crewAIInc/crewAI/pull/8333) |
| Opened | `2026-10-20T17:17:57Z` by `simin-yuan` |
| Closes | [#8332](https://github.com/crewAIInc/crewAI/issues/8332) |
| State at capture | open; unlabelled; 2 comments; 3 files changed; mergeable |
| Files in this folder | the body as it stands, our comment, the raw API payloads, and this record |

## What it is

CrewAI's `run_with_rate_limit_retry` / `arun_with_rate_limit_retry` replay a call that hit a rate
limit, but the OpenAI client inside each replay keeps its own retries — the same multiplication
measured on smolagents in [`smolagents/2026-10-20-issue-3034`](../../smolagents/2026-10-20-issue-3034/).

The patch records whether the attempt about to run is a replay (a contextvar exposed as
`in_rate_limit_retry_replay()`), and the OpenAI provider builds its client accordingly: replays get
`max_retries=0`, first attempts keep the client's own retries. All nine provider call sites in the
completion module go through that one helper.

Three files: the provider completion module, the retry helpers, and the retry tests.

## The disclosure comment

`2026-10-20T17:41:12Z`, our second action on the PR: a request that a maintainer apply the
`llm-generated` label, noting that contributors cannot set labels themselves.

This is on the record deliberately. CrewAI's contributing rules require AI involvement to be
disclosed, and disclosure is not the same as giving up the authorship — the work is ours, the tool
use is stated plainly, and the label is applied by a maintainer because the platform does not let us.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-10-20T17:17:57Z` | PR opened. |
| `2026-10-20T17:18:03Z` | `strix-security[bot]` posts a review note: one commit (`ec93a00`) arrived after its last review and has not been reviewed by it. |
| `2026-10-20T17:41:12Z` | Our comment asking for the `llm-generated` label, per CONTRIBUTING. |
| `2026-10-22T13:50:49Z` | Capture. Open, mergeable, no maintainer response, label still absent. |

## What would count as success

Merged, with the `llm-generated` label applied. If a maintainer says the replay scope is wrong, the
record keeps the reasoning publicly rather than quietly editing it.

## Files in this folder

- `body-current.md` — the PR body and our comment as they stand on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json`, `api-files.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
