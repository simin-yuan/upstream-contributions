Two data points in case they help with #11494.

1. On the latest release (`@langchain/core` 1.2.18, `@langchain/openai` 1.6.2, `@langchain/langgraph` 1.5.1), `createReactAgent` with `new ChatOpenAI({ streaming: true })` still runs the cut-off call end to end. A `write_file` call cut in the middle of line 17 of a 40-line file writes the first 16 lines plus part of line 17, the tool returns success, and the agent answers "Done.". Same with `useResponsesApi: true`. With `streaming: false` the call is not run.

2. The same happens when the stream ends with no finish reason at all (the body closes cleanly, no `[DONE]`), which is what a proxy or server dropping the stream looks like. The approach in #11494 keys on `finishReason === "length"` in `ChatModelStream`, so it can't catch that case. Also, in the agent runs above the call is built by `collapseToolCallChunks` (`src/messages/utils.ts`, the `parsePartialJson` call at line 634 on current main), which #11494 doesn't change.

Harness and per-run server logs: https://github.com/simin-yuan/agent-cutoff-bench

Python has the same behaviour; it is tracked in langchain-ai/langchain#41285.
