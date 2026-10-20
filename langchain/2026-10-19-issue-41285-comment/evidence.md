# Evidence

Every claim in `record.md`, and what it rests on.

| Claim | Rests on |
|---|---|
| Issue #41285 exists, opened by `taguacil` | `api-issue.json` → `created_at`, `user.login`, `html_url` |
| Open, six comments, no assignee at capture | `api-issue.json` → `state: "open"`, `comments: 6`, `assignees: []` |
| Labels `bug`, `core`, `external` | `api-issue.json` → `labels`; `api-timeline.json` → `labeled` events at `10:50:58Z` (by `taguacil`) and `10:51:04Z` / `10:51:08Z` (by `github-actions[bot]`) |
| Our comment is `6273122378`, posted `17:26:02Z` | `api-comment.json` → `id`, `created_at`, `user.login = simin-yuan` |
| We are the only `simin-yuan` comment on the issue | `api-comments.json` → filtered on `user.login`, one entry |
| `Kylinny` claimed it at `11:21:24Z` | `api-comments.json` → comment `6233291856`: "I'd like to take this one" |
| `charan-rathore` reproduced it on 1.6.12 | `api-comments.json` → comment `6233296327` |
| The other two stepped aside | `api-comments.json` → `6233415478` ("happy to leave this one with you"), `6261375077` ("I'll step back") |
| #41286 was cross-referenced from this issue | `api-timeline.json` → `cross-referenced` event at `14:35:22Z`, source `langchain-ai/langchain#41286` |
| Our comment cross-references #11469 | `api-timeline.json` → `cross-referenced` event at `17:26:18Z`, source `langchain-ai/langchainjs#11469` |
| A duplicate was posted and deleted | `api-timeline.json` → `mentioned` / `subscribed` events for `Kylinny` at `21:45:16Z`, the minute the duplicate `6278084871` was posted. The duplicate now returns 404 from the API; the events it generated are still in the timeline. |
| Eight cells ran the truncated call | `results/runs.jsonl` in the bench: rows with `tool_executed: true` **and** `mode` in `length` / `drop` — eight rows, four for each language. The other 16 executed rows are the `complete` control. |
| 866 bytes instead of 2,120 | `results/with-core-fix/` (patched control, writes all 40 lines) against the truncated runs; the bench README table is generated from `runs.jsonl` by `summarize.py` |
| The other four frameworks did not run it | Same `runs.jsonl`, the `tool_executed: false` rows for those frameworks in `length` / `drop` |

## What is not claimed here

- That our comment is a fix, or that we are entitled to one. No patch was proposed in the thread;
  the patch lives in the bench repository, unsubmitted, and `iamdgarcia` had already published one.
- That the `reset` mode was tested for LangGraph. It was not: the connection error stopped every
  client before a call could be built.
- That a maintainer has read the comment. At capture, none had replied.
