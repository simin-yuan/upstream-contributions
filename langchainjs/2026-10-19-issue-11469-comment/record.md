# LangChain.js — The same cut-off call runs, and #11494 cannot catch a stream that stops

| | |
|---|---|
| Issue | [#11469](https://github.com/langchain-ai/langchainjs/issues/11469) |
| Comment | `6273127102`, posted `2026-10-19T17:26:17Z` (API `created_at`) |
| State at capture | open; three comments |
| Labels | none |
| Files in this folder | the comment as it stands, the raw API payloads, an Internet Archive snapshot, and `evidence.md` |

## What it is

A comment under a bug report that `Sparky579` opened on 2026-08-25 for the same behaviour on the
JavaScript side. The comment adds two data points for #11494, the patch that was already proposed:

1. On `@langchain/core` 1.2.18, `@langchain/openai` 1.6.2 and `@langchain/langgraph` 1.5.1,
   `createReactAgent` with streaming still runs the cut-off call end to end — 16 lines of 40, the
   tool returns success, the agent answers `Done.`. Same through `useResponsesApi`, and not run with
   `streaming: false`.
2. The same happens when the stream ends with no finish reason at all (body closes cleanly, no
   `[DONE]`), which is what a proxy dropping the stream looks like. #11494 keys on
   `finishReason === "length"` in `ChatModelStream`, so it cannot catch that case. In those runs the
   call is built by `collapseToolCallChunks` (`src/messages/utils.ts`, the `parsePartialJson` call at
   line 634 on current main), which #11494 does not change.

It points back at langchain-ai/langchain#41285 for the Python side.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-08-25T11:47:01Z` | Issue #11469 opened by `Sparky579`. |
| `2026-08-25T14:30:17Z` | `haoku123` cross-references #11471. |
| `2026-08-27T11:54:13Z` | `mikemikimike`, comment `5438627624`. |
| `2026-09-08T02:59:29Z` | `mikemikimike` cross-references #11494, a patch for the same behaviour. |
| `2026-10-14T10:26:16Z` | `canblmz1`, comment `6166247064`. |
| `2026-10-14T10:34:07Z` | `mikemikimike` unsubscribes. |
| `2026-10-19T17:26:17Z` | Our comment, `6273127102`. |

No maintainer has replied and nobody is assigned, as of capture. A duplicate of our comment was posted
here too (`6278088015`, posted in the `21:45Z` minute of the same hour) and deleted by its author; unlike #41285 it left no timeline
event, because that comment mentioned nobody.

## What was measured

The same bench as on the Python side, built for the JS clients: a fake server cuts one `write_file`
call — 40 lines, arguments stopping in the middle of line 17 — and the tool writes to disk. The
JavaScript rows behave the same way as the Python ones: streaming on, the cut-off call runs in both
the `length` and `drop` modes and under both API shapes; streaming off, it is not run. Writes came
out at 866 bytes instead of 2,120, with the tool reporting success.

Bench: https://github.com/simin-yuan/agent-cutoff-bench, commit `e624cdb`.

## What would count as success

#11494 is closed and unmerged, with no review, so the near-term outcome worth recording is a
maintainer saying which of the two cases they intend to cover — the output-limit case only, or also a
stream that ends without a finish reason.

## Files in this folder

- `comment-current.md` — the comment as it stands on the platform.
- `api-comment.json`, `api-issue.json`, `api-comments.json`, `api-timeline.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
- `evidence.md` — each claim, and where it can be checked.
- `archive.md` — the Internet Archive snapshot.
