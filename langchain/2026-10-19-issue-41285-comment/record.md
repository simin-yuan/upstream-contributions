# LangChain — Truncated tool-call arguments survive the end of the stream

| | |
|---|---|
| Issue | [#41285](https://github.com/langchain-ai/langchain/issues/41285) |
| Comment | `6273122378`, posted `2026-10-19T17:26:02Z` (API `created_at`) |
| State at capture | open; six comments; no assignee |
| Labels | `bug`, `core`, `external` |
| Files in this folder | the comment as it stands, the raw API payloads, an Internet Archive snapshot, and `evidence.md` |

## What it is

A comment under a bug report that `taguacil` opened on 2026-10-17: once a streamed tool call's
arguments are cut off, they are still parsed leniently after the stream ends, so a half-written call
arrives as a valid `tool_call` and `invalid_tool_calls` stays empty.

The comment reports an end-to-end measurement on six agent frameworks, then raises two points about
the patch in #41286 (`5e75fa5`, closed by a bot 31 s after it was opened):

1. The non-streaming parser calls `json.loads(..., strict=False)`. A strict `json.loads` at
   finalization marks a complete call whose string contains a raw newline as invalid, so a call that
   runs without streaming would stop running with streaming.
2. Fixing it inside `message_chunk_to_message` covers `invoke()`, but code that sums the chunks
   itself (`full = full + chunk`, then `full.tool_calls`) still gets the healed arguments. Checking
   in `init_tool_calls` once `chunk_position == "last"` covers both paths.

It does not ask for an assignment. `Kylinny` said they were taking it about 30 minutes after the issue
was opened, and the other two people who reproduced it stepped aside for that reason.

## Timeline

All times UTC, as returned by the API.

| Time (UTC) | Event |
|---|---|
| `2026-10-17T10:50:56Z` | Issue #41285 opened by `taguacil`. |
| `2026-10-17T10:50:58Z` | `taguacil` applied `bug`. |
| `2026-10-17T10:51:04Z` | `github-actions[bot]` applied `core`. |
| `2026-10-17T10:51:08Z` | `github-actions[bot]` applied `external`. |
| `2026-10-17T11:21:24Z` | `Kylinny`, comment `6233291856`: would like to take this one, proposing a strict re-parse in `generate_from_stream` / `init_tool_calls`. |
| `2026-10-17T11:21:55Z` | `charan-rathore`, comment `6233296327`: reproduces on `langchain-core` 1.6.12 — args land in `tool_calls` as `{'type': 'hea'}`, `invalid_tool_calls` stays empty. |
| `2026-10-17T11:35:44Z` | `charan-rathore`, comment `6233415478`: leaves it with `Kylinny`. |
| `2026-10-17T14:35:22Z` | `iamdgarcia` cross-references #41286, a patch for the same issue. |
| `2026-10-17T14:40:42Z` | `iamdgarcia`, comment `6235212007`: fix is up in #41286, asks to be assigned so the PR passes the issue-link gate. |
| `2026-10-19T06:44:49Z` | `iamdgarcia`, comment `6261375077`: sees the earlier claim, steps back, offers the patch. |
| `2026-10-19T17:26:02Z` | Our comment, `6273122378`. |
| `2026-10-19T17:26:18Z` | Our comment cross-references [langchain-ai/langchainjs#11469](https://github.com/langchain-ai/langchainjs/issues/11469), the same behaviour on the JavaScript side. |
| `2026-10-19T21:45:16Z` | A `mentioned` and a `subscribed` event on `Kylinny`, left behind by a duplicate of our comment. The duplicate (`6278084871`) was posted in the `21:45Z` minute and deleted by its author a few minutes later; the comment now returns 404, the two events it generated do not. |

As of the capture at `2026-10-20T06:30Z` nobody has been assigned, and no maintainer has replied
after our comment.

## What was measured

A fake model server cuts the same `write_file` call for every client: a 40-line file, the arguments
stop in the middle of line 17. The tool writes to disk and the bench reads the file back.

Across 64 runs — six frameworks × API shape × streaming on/off × four cut-off modes — eight cells
ran the truncated call: LangGraph for Python and for JavaScript, through Chat Completions and through
Responses, streaming, in both `length` (the server sends `finish_reason: "length"`) and `drop` (the
body ends with no finish reason and no `[DONE]`). Each of those wrote 866 bytes instead of 2,120,
reported `Wrote 866 characters`, and the agent answered `Done.`. Nothing raised.

With `streaming=False`, on the same server, the tool is not called. On the same server, OpenAI Agents
SDK, smolagents, Codex CLI and Hermes Agent do not run the cut-off call.

Bench: https://github.com/simin-yuan/agent-cutoff-bench, commit `e624cdb`. The patch in
`fix/langchain-core/` applies cleanly to `langchain` `master`; its file hash at the time of writing
is pinned in the bench README.

## What would count as success

One of two outcomes: the finalization boundary is fixed so a cut-off call lands in
`invalid_tool_calls`, or a maintainer explains why the lenient parse is intended. #41286 was closed
unmerged without a review, so this comment may also be the only place where the two gaps above are
written down.

## Files in this folder

- `comment-current.md` — the comment as it stands on the platform.
- `api-comment.json`, `api-issue.json`, `api-comments.json`, `api-timeline.json` — raw API payloads.
- `snapshot.json` — the same payloads in one file, with the capture time.
- `evidence.md` — each claim, and where it can be checked.
- `archive.md` — the Internet Archive snapshot.
