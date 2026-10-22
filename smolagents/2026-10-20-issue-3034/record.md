# smolagents — Client retries and `ApiModel` retries multiply on a rate limit

| | |
|---|---|
| Issue | [#3034](https://github.com/huggingface/smolagents/issues/3034) |
| Opened | `2026-10-20T12:12:12Z` by `simin-yuan` |
| State at capture | open; unlabelled; 3 comments; no assignee |
| Patch | [#3036](https://github.com/huggingface/smolagents/pull/3036), opened the same day |
| Files in this folder | the body as it stands, our comment, the raw API payloads, and this record |

## What it is

`OpenAIModel` and `AzureOpenAIModel` wrap `client.chat.completions.create` in `ApiModel.retryer`
(3 attempts). The `openai` client inside keeps its default `max_retries=2`. On a 429 each outer
attempt therefore makes three requests, and one rate limit costs nine requests.

The report includes a minimal repro against a local server that always answers `429` with
`Retry-After: 1`, plus the measured request count and the timings.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-10-20T12:12:12Z` | Issue opened. |
| `2026-10-20T13:34:54Z` | `ShivanshSinghaL06` says they would like to take it, restating the mechanism. |
| `2026-10-20T14:16:46Z` | Our reply: a patch with tests is ready and the PR will be opened today; also records a trade-off measured while writing it — constructing the client with `max_retries=0` also removes the only retries smolagents has for 5xx and dropped connections, so the fix has to keep those. |
| `2026-10-20T15:21:24Z` | PR #3036 opened, closing this issue. |
| `2026-10-20T22:43:56Z` | `ShivanshSinghaL06`: understands the trade-off, will follow the PR through review or testing. |
| `2026-10-22T13:50:49Z` | Capture. Issue open, PR open, no maintainer response. |

## Why this one is kept

Two things happened here that matter for the record. Someone else said they wanted the work, and
the patch went in the same day anyway — a claim is not a contribution, the public timestamp is. And
the reply did not simply say "I have it": it handed over a measurement that changes how the obvious
fix should be written. Whoever reviews the PR gets that for free.

## What would count as success

The PR is merged, or a maintainer explains why nine requests on a single 429 is the behaviour they
want. Either way the number is now written down somewhere public.

## Files in this folder

- `body-current.md` — the issue body and our comment as they stand on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
