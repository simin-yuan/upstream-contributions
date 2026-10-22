# smolagents — `Retrying` ignores `Retry-After`, and the first retry already waits 2–3× `wait_seconds`

| | |
|---|---|
| Issue | [#3037](https://github.com/huggingface/smolagents/issues/3037) |
| Opened | `2026-10-20T15:48:12Z` by `simin-yuan` |
| State at capture | open; unlabelled; 1 comment (ours); no assignee |
| Files in this folder | the body as it stands, our follow-up comment, the raw API payloads, and this record |

## What it is

Two findings, measured on `smolagents==1.26.0` and on `main` at `96f33fa`:

1. `Retry-After` is never read. When a provider answers 429 and says "come back in 3 seconds", the
   retry sleeps for whatever the exponential schedule says instead.
2. The first retry already waits 2–3× the configured `wait_seconds`, because the delay is multiplied
   by the exponential base (plus jitter) before the first sleep rather than after it.

## The correction

Point 2 already had a fix in the open: [#2596](https://github.com/huggingface/smolagents/pull/2596),
opened `2026-08-02` and still open, moves the sleep before the multiplication — exactly the change
this issue proposes.

We found it after opening the issue and said so publicly, as a comment on our own issue, rather than
letting the overlap sit there. The issue stays open for point 1, which is not covered by #2596.
That is the honest shape of it: one original observation, one duplicate, stated as such in the
place where the duplicate would otherwise be discovered by someone else.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-08-02` | #2596 opened, fixing the sleep-before-multiplication point. Still open. |
| `2026-10-20T15:48:12Z` | This issue opened with both findings. |
| `2026-10-20T17:53:55Z` | Our follow-up comment: prior art on point 2 is #2596, which should have been checked before opening. |
| `2026-10-22T13:50:49Z` | Capture. Open, no maintainer response. |

## What would count as success

A maintainer reads `Retry-After` — that is the part that is still unfixed anywhere.

## Files in this folder

- `body-current.md` — the issue body and our follow-up comment as they stand on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
