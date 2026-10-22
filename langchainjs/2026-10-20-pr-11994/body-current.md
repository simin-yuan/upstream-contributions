# fix(core): don't turn truncated streamed tool-call arguments into a tool call

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

Fixes #11469

**Problem.** When a stream ends partway through a tool call's arguments, `collapseToolCallChunks` repairs the partial JSON with `parsePartialJson`. The aggregated message then has a complete-looking entry in `tool_calls` and nothing in `invalid_tool_calls`, so agents built on `invoke()` (e.g. `createReactAgent`) run the tool with half its input. This happens on `finish_reason: "length"`, and also when the stream simply ends (no finish reason, no `[DONE]`), which is what a dropped connection or proxy looks like.

**Change.**
- `collapseToolCallChunks(chunks, { final: true })` parses with `JSON.parse`, the rule the non-streaming path (`parseToolCall`) already applies. The default is unchanged.
- `BaseChatModel` applies it once a stream has ended, in both places a streamed message is assembled: its own aggregation (streaming callbacks, e.g. LangGraph's `messages` mode) and the result of `_generate` for providers that stream inside it (e.g. `ChatOpenAI` with `streaming: true`).
- Chunks yielded while streaming keep the partial parse, so progressive rendering is unchanged.

It does not key on the finish reason, so it also covers the dropped-stream case, which #11494 (closed) could not.

**Relation to #11471.** #11471 addresses the same issue and has been open since August. Two differences worth checking against it, both in `libs/langchain-core/src/messages/utils.ts`:

1. #11471 passes a `truncated` flag gated on `response_metadata.finish_reason === "length"`. A stream that simply ends (no finish reason, no `[DONE]`) never sets it, so that path still produces a repaired call.
2. Its strict check runs before the `if (usesRawInputArgs && id)` branch, so `JSON.parse(argsStr)` also applies to raw-input/custom tools whose arguments are deliberately not JSON. That is the question drew-chen left on #11471 and it is still open. Here the strict parse happens inside the existing `try`, which is reached only after that branch has already `continue`d, so raw-input tools are untouched.

If you would rather land #11471, I am glad to rebase this onto it or carry the dropped-stream and raw-input cases over there instead — either way the fix should not depend on the finish reason.

**Tests.** `truncated_tool_calls.test.ts`: 7 tests, 3 fail before the change. They cover both aggregation paths, a complete call (with an escaped newline) still producing a tool call, mid-stream chunks staying lenient, and empty args still meaning `{}`. Full `@langchain/core` unit suite: 1556 passed, 1 skipped; the 5 failures (`count_tokens`, the tracer snapshot) time out the same way without this change, because they need network access. oxlint, oxfmt and tsc are clean.

**End to end**, `ChatOpenAI({ streaming: true }).invoke()` against a fake endpoint (tool calls / invalid tool calls):

| stream | before | after |
|---|---|---|
| cut mid-arguments, `finish_reason: "length"` | 1 / 0 | 0 / 1 |
| cut mid-arguments, stream just ends | 1 / 0 | 0 / 1 |
| complete call | 1 / 0 | 1 / 0 |

**Not in scope.** Code that iterates `.stream()` and concatenates the chunks itself still gets the repaired call, because the chunks carry no end-of-stream marker. Python's core marks the final chunk with `chunk_position="last"`; a similar marker would be a follow-up, and I'm happy to do it if you want that direction.

The Python side is tracked in langchain-ai/langchain#41285. Agent-level reproduction across frameworks: https://github.com/simin-yuan/agent-cutoff-bench
