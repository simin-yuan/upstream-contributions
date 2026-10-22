# smolagents — Disable client retries on `ApiModel.retryer` replays

| | |
|---|---|
| Pull request | [#3036](https://github.com/huggingface/smolagents/pull/3036) |
| Opened | `2026-10-20T15:21:24Z` by `simin-yuan` |
| Closes | [#3034](https://github.com/huggingface/smolagents/issues/3034) |
| State at capture | open; unlabelled; no review; mergeable |
| Files in this folder | the body as it stands, the raw API payloads, and this record |

## What it is

The patch for the nine-requests-per-rate-limit measurement in
[`2026-10-20-issue-3034`](../2026-10-20-issue-3034/).

It keeps the client's own retries on the first attempt and applies
`self.client.with_options(max_retries=0)` only on the retryer's replays. A 429 then costs
3 + 1 + 1 = 5 requests instead of 9. 5xx errors, dropped connections and timeouts are unchanged,
because the retryer does not retry those and the client's retries are their only retry.

Setting `max_retries=0` on the client outright would have dropped those failure modes to a single
request — measured, and recorded in the linked issue before the PR existed. That is the reason the
fix is scoped to replays rather than to the client.

Two files: `src/smolagents/models.py` and `tests/test_models.py`.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-10-20T12:12:12Z` | Issue #3034 opened. |
| `2026-10-20T14:16:46Z` | Our comment on the issue: patch and tests ready, PR today. |
| `2026-10-20T15:21:24Z` | This PR opened. |
| `2026-10-22T13:50:49Z` | Capture. Open, mergeable, no review, no CI status reported. |

## What would count as success

Merged, or a maintainer argues the replay-only scope is wrong and says what to change. A review that
prefers simply setting `max_retries=0` would be a different trade-off than the one measured here,
and that would be worth recording rather than arguing with.

## Files in this folder

- `body-current.md` — the PR body as it stands on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json`, `api-files.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
