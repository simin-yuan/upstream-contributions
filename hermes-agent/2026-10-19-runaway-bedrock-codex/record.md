# Hermes Agent — Bedrock and Codex streams don't cut a repetition loop mid-stream

| | |
|---|---|
| Issue | [#141477](https://github.com/NousResearch/hermes-agent/issues/141477) |
| Pull request | [#141479](https://github.com/NousResearch/hermes-agent/pull/141479) |
| Submitted | 2026-10-19 17:46 (UTC+08:00) — issue `09:46:37Z`, PR `09:46:52Z` |
| State | issue open; PR open, mergeable, CI gated on maintainer approval for a first-time contributor |
| Labels | issue: `type/bug`, `comp/agent`, `provider/openai`, `provider/bedrock`, `P3`, `area/streaming`; PR adds the same set with `P2` |
| Files in this folder | the issue and the pull request, side by side, because the PR exists only to close the issue |

## What it is, and why it mattered

#130551 wired `RunawayStreamWatch` into two of the four streaming consumers: the chat_completions
stream (`agent/chat_completion_helpers.py:3054`) and the Anthropic Messages stream (`:3396`). The
Bedrock Converse stream (`agent/bedrock_adapter.py`, `stream_converse_with_callbacks`) and the Codex
Responses stream (`agent/codex_runtime.py`, `_consume_codex_event_stream`) read a looping stream to
its end, and only the post-stream check catches the loop afterwards.

That matters because the providers documentation promises the opposite: a reply that degenerates into
a repetition loop "is still stopped ... within about 130,000 characters of the loop starting", so an
uncapped endpoint cannot keep a looping model running. The same page notes that Converse omits
`inferenceConfig.maxTokens` by default, which AWS documents as the model maximum. So the one path the
documentation singles out as uncapped is also a path without the mid-stream guard.

Measured on unmodified `main` (`2e83e2a`), feeding ~141,000 characters of one repeated line:
`RunawayStreamWatch` trips at 16,074 characters, while `stream_converse_with_callbacks` and
`_consume_codex_event_stream` each read all 141,000. The answer the user finally sees is still right
— the post-stream check rejects the turn — but the whole output budget is spent first, and on
Converse that budget is the model's maximum.

## The fix

The patch wires the existing watch into both consumers; no new detection logic. In
`stream_converse_with_callbacks` a `_ConverseRunawayWatch` wraps the event iterator, so the consumer
loop gains no branches (the upstream code-health ratchet is one-way); a cut returns what streamed so
far with `stop_reason=max_tokens`, and a tool call cut mid-arguments is dropped together with its
content block so no `toolUse` is left without a `toolResult`. In `_consume_codex_event_stream` the
text and reasoning deltas feed two watches; a cut sets `_runaway_repetition`, and
`_derive_finish_reason` routes the turn to the length path. The new test file covers both paths,
including the live Bedrock dispatch through `interruptible_streaming_api_call`.

## Timeline

All times UTC as returned by the API; Beijing time is +08:00.

| Time | Event |
|---|---|
| 2026-10-19 09:46:37 | Issue #141477 opened. |
| 2026-10-19 09:46:52 | PR #141479 opened against `main`, one commit, five files. |
| 2026-10-19 09:51:24 | @alt-glitch labeled the PR `comp/agent`, `provider/bedrock`, `provider/openai`, `type/bug`, `area/streaming`, `P2`. |
| 2026-10-19 09:51:31 | @alt-glitch labeled the issue with the same six labels at `P3`. |
| 2026-10-19 09:51:35 | @alt-glitch commented on the issue, pointing at the pull request (quoted below). |
| 2026-10-19 09:56:49 | Internet Archive snapshot of the issue requested and verified. |
| 2026-10-19 09:57:41 | Internet Archive snapshot of the PR requested and verified. |
| 2026-10-19 10:27:46 | Second Internet Archive snapshot of the issue, after the comment above was added to this record. |
| 2026-10-19 15:26:28 | Second commit, `b6c366b`: adds `contributors/emails/yleven120@gmail.com`, one line reading `simin-yuan`. The upstream `Contributor Attribution Check` fails any PR whose commit author email has no file under `contributors/emails/`, and `yleven120@gmail.com` was mapped only inside #141340. The file's blob is `e10ecda90b30dbd4dd7ac5467d9d46f5024aab4b`, byte-identical to the copy in #141340, so whichever of the two merges first the other still applies cleanly. The PR now carries six files; the other five are unchanged. |

> #141479 aims to fix this: it adds the mid-stream repetition-loop guard to the Bedrock Converse and
> Codex Responses stream consumers, so a looping reply stops early instead of using up the whole
> output budget.
>
> — `alt-glitch`, 2026-10-19 09:51:35Z

The visible text is quoted exactly as served. The comment body also ends with an invisible marker,
`<!-- argus:v1 item=141477 -->`, which is part of the body returned by the API and is kept here so a
reader can reconcile this quote with `snapshot.json` byte for byte.

CI on the head commit is gated: first-time contributors need a maintainer to approve the run before
the workflows execute, so `gh pr checks` reports no checks. That is not a failure.

## What changed in the text after submission

- `issue-body-original.md` → `issue-141477-current.md`: the PR cross-reference placeholder `#(PR)`
  was replaced with `#141479` once the pull request existed. Nothing else was altered.
- `pr-body-original.md` → `pr-141479-current.md`: identical; the body has not been edited since it
  was opened.
