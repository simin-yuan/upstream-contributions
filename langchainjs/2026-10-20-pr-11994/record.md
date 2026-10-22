# LangChain.js — Truncated streamed tool-call arguments run as a valid call

| | |
|---|---|
| Pull request | [#11994](https://github.com/langchain-ai/langchainjs/pull/11994) |
| Opened | `2026-10-20T15:08:19Z` by `simin-yuan` |
| Closes | [#11469](https://github.com/langchain-ai/langchainjs/issues/11469) |
| State at capture | open; mergeable; 4 checks green; no review |
| Files in this folder | the body as it stands, the raw API payloads, and this record |

## What it is

A fix for the JavaScript side of the same defect measured on the Python side in
[`langchain/2026-10-19-issue-41285-comment`](../../langchain/2026-10-19-issue-41285-comment/).

When a stream ends partway through a tool call's arguments, `collapseToolCallChunks` repairs the
partial JSON with `parsePartialJson`. The aggregated message then carries a complete-looking entry
in `tool_calls` and nothing in `invalid_tool_calls`, so an agent built on `invoke()` — a
`createReactAgent`, for instance — runs the tool with half its input. This happens on
`finish_reason: "length"` and also when the stream simply ends with no finish reason and no
`[DONE]`, which is what a dropped connection or an intermediary proxy looks like from the client.

The change parses with `JSON.parse` at finalisation, the same rule the non-streaming path already
uses, so a cut-off call lands in `invalid_tool_calls` instead of being healed into a runnable one.

Four files: a changeset, the chat-model aggregation, the message utilities, and a new test file.
The bench that produced the measurement is [`agent-cutoff-bench`](https://github.com/simin-yuan/agent-cutoff-bench).

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-10-19T17:26:02Z` | Our comment on the Python issue #41285, measuring the behaviour across six frameworks. |
| `2026-10-20T15:08:19Z` | This PR opened, `Fixes #11469`. |
| `2026-10-20T15:08:26Z` | `changeset-bot` reports a changeset detected for one package. |
| `2026-10-22T13:50:49Z` | Capture. Still open, mergeable, four checks green, no maintainer review. |

## What would count as success

Either a maintainer takes the strict parse at finalisation, or a maintainer states that healing a
cut-off call into a runnable one is intended. The second outcome is a real answer too, and would be
recorded here as such.

## Files in this folder

- `body-current.md` — the PR body as it stands on the platform.
- `api-item.json`, `api-comments.json`, `api-timeline.json`, `api-files.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
