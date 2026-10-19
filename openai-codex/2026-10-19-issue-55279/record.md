# Codex — Stop a streamed reply that turns into a repetition loop

| | |
|---|---|
| Issue | [#55279](https://github.com/openai/codex/issues/55279) |
| Submitted | `2026-10-19T13:15:07Z` (API `created_at`) |
| State at capture | open; no human had triaged it |
| Labels | `enhancement`, `CLI`, `exec`, `model-behavior` — all four applied by `github-actions[bot]` at `2026-10-19T13:15:48Z` |
| Files in this folder | the text as filed, our reply, the raw API payloads, an Internet Archive snapshot |

## What it is

A feature request: when a streamed reply degenerates into a repetition loop, the client should end the
turn instead of reading the loop to the model's output limit. `codex exec` runs unattended, so in CI or a
scheduled job the loop goes straight into a log or the next step.

Two facts the request rests on: the main turn request carries no output cap (`ResponsesApiRequest` in
`codex-rs` has no `max_output_tokens` field), and there is no mid-stream repetition check.

## Timeline

All times UTC, as returned by the API. Relative times name the event they refer to, and none is carried
over from one reference point to another.

| Time (UTC) | Event |
|---|---|
| `2026-10-19T13:15:07Z` | Issue #55279 opened by `simin-yuan`. |
| `2026-10-19T13:15:48Z` | `github-actions[bot]` applied four labels: `enhancement`, `CLI`, `exec`, `model-behavior`. 41 s after the issue was created, and 43 s before the automated duplicate check below. |
| `2026-10-19T13:16:31Z` | `github-actions[bot]` posted a possible-duplicate notice pointing at #54285 (comment `6268132352`). An automated check, not a person. |
| `2026-10-19T13:26:14Z` | Our reply, comment `6268332394`: this is a separate request, not a duplicate. |
| `2026-10-19T14:54:29Z` | Internet Archive snapshot requested and verified. |

As of the capture at `2026-10-19T14:53Z` the issue's whole timeline is six events: the four label
events, the automated duplicate notice, and our reply. Nobody else has touched it — there is no
`assigned`, `milestoned`, `closed` or `referenced` event, and no person besides us has commented.

## Why it is not a duplicate of #54285

#54285 is about `codex exec --output-schema` with `minLength`/`maxLength` constraints failing on some
models. This request is about a client-side cut when any streamed reply turns into a loop, whatever the
schema or model. They meet at one point: the run described in a comment on #54285 (`wesleycheung`,
2026-10-16) produced 216,579 characters of repetitive JSON and was still streaming when that harness's
about-600-second diagnostic limit stopped it. Those two numbers come from that comment, not from the
issue body.

## What was measured

`codex exec` was pointed at a local Responses endpoint that streams one sentence on repeat, roughly
300,000 characters. Codex read all of it and printed 300,049 characters to stdout; the last byte is the
trailing newline from the runner's `wc -c`. Endpoint and runner:
https://github.com/simin-yuan/agent-runaway-bench. Ubuntu 24.04, `openai/codex` main `89461ea` and the
0.163.0 npm release.

## What would count as success

Codex does not accept external pull requests, so this thread is not waiting on a merge. The outcome worth
recording is the issue being kept open, triaged by a person, and — ideally — implemented upstream.

## Files in this folder

- `issue-body.md` — the text as filed.
- `reply-2026-10-19.md` — our comment, exactly as posted.
- `api-issue.json`, `api-comments.json`, `api-timeline.json` — raw API payloads.
- `snapshot.json` — the same three payloads in one file, with the capture time.
- `evidence.md` — each claim, and where it can be checked.
- `archive.md` — the Internet Archive snapshot.
