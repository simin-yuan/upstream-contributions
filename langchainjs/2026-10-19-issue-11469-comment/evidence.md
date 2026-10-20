# Evidence

Every claim in `record.md`, and what it rests on.

| Claim | Rests on |
|---|---|
| Issue #11469 exists, opened by `Sparky579` | `api-issue.json` → `created_at`, `user.login`, `html_url` |
| Open, three comments, no labels at capture | `api-issue.json` → `state: "open"`, `comments: 3`, `labels: []` |
| Our comment is `6273127102`, posted `17:26:17Z` | `api-comment.json` → `id`, `created_at`, `user.login = simin-yuan` |
| We are the only `simin-yuan` comment on the issue | `api-comments.json` → filtered on `user.login`, one entry |
| #11494 was cross-referenced before we arrived | `api-timeline.json` → `cross-referenced` event at `2026-09-08T02:59:29Z`, source `langchain-ai/langchainjs#11494` |
| Only one `simin-yuan` comment remains | `api-comments.json` → filtered on `user.login`, one entry. A duplicate (`6278088015`, posted in the `21:45Z` minute) was deleted by its author and now returns 404; `api-timeline.json` has no event from it. |
| The versions named in the comment | `results/runs.jsonl` → the `versions` object on the `langgraph-js` rows: `@langchain/core` 1.2.18, `@langchain/openai` 1.6.2, `@langchain/langgraph` 1.5.1 |
| The JS rows behave like the Python ones | `results/runs.jsonl` in the bench: LangGraph.js rows with `tool_executed: true` and `mode` in `length` / `drop` — four of them — and `tool_executed: false` with `streaming: false` |
| `#11494` keys on `finishReason === "length"` | That PR's changed files: `libs/langchain/src/.../stream.ts` only, adding the `finishReason === "length"` branch. Read through the API at `pulls/11494/files`. |
| `collapseToolCallChunks` builds the call in the agent path | `src/messages/utils.ts`, `parsePartialJson` call at line 634 on current main |

## What is not claimed here

- That #11494 is wrong. It is narrower than the report: it covers the output-limit case and not a
  stream that ends without a finish reason.
- That the JavaScript behaviour differs from the Python one in any way other than the code path.
  Both run the cut-off call.
- That a maintainer has read the comment. At capture, none had replied.
