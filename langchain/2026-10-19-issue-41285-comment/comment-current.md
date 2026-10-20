Independent reproduction, end to end, plus two notes on the patch in #41286.

**End to end.** `create_react_agent` with `ChatOpenAI(streaming=True)`, against a local server that cuts a `write_file` call in the middle of line 17 of a 40-line file. Both `finish_reason: "length"` and a stream that simply ends (no finish reason, no `[DONE]`) give the same result: the tool writes the first 16 lines plus part of line 17, returns success, and the agent answers "Done.". Same with `use_responses_api=True` (`response.incomplete`). With `streaming=False` the tool is not called. On the same server, the OpenAI Agents SDK, smolagents, Codex CLI and Hermes Agent don't run the cut-off call. Harness and per-run server logs: https://github.com/simin-yuan/agent-cutoff-bench

**Two notes for whoever lands the fix**, checked against #41286 at `5e75fa5`:

1. The non-streaming parser uses `json.loads(..., strict=False)`. A strict `json.loads` at finalization marks a complete call whose string contains a raw newline as invalid, so a call that runs without streaming would stop running with streaming.
2. Fixing it in `message_chunk_to_message` covers `invoke()`, but code that sums the chunks itself (`full = full + chunk`, then `full.tool_calls`) still gets the healed arguments. Checking in `init_tool_calls` once `chunk_position == "last"` covers both.

`fix/langchain-core/edge_cases.py` in that repo prints both cases for any build, and `fix/langchain-core/` has a patch that does the check that way, with tests for both. @Kylinny, feel free to use any of it.
