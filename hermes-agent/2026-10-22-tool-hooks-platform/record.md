# Hermes Agent — `platform` is missing from the tool-path hooks

| | |
|---|---|
| Issue | [#142973](https://github.com/NousResearch/hermes-agent/issues/142973) |
| Pull request | [#142974](https://github.com/NousResearch/hermes-agent/pull/142974) |
| Submitted | 2026-10-22 09:10:46Z (issue), 09:10:51Z (PR) — 17:10 UTC+08:00 |
| State | open; workflows created but `action_required` (first-time contributor, waiting for a maintainer to approve the run); no review, no comment from a maintainer on the PR |
| Head | `simin-yuan:fix/tool-hooks-platform` at `d08a850f92` (one commit, based on `81a55437f3`) |
| Size | 10 files, +189 / −20 |
| Labels | issue `type/bug` `comp/agent` `P2`; PR `type/feature` `comp/agent` `comp/cli` `comp/plugins` `P3` `area/i18n` |
| Files in this folder | the issue and PR text as filed and as they stand, raw API payloads, the patch, and the local run output |

## What it is

`pre_llm_call` has received the platform name (`cli`, `cron`, `subagent`, `curator`, `gateway`) for
some time — `agent/turn_context.py` builds it as `getattr(agent, "platform", None) or ""`. The three
hooks on the tool path do not: `pre_tool_call`, `post_tool_call` and `transform_tool_result` are all
built from `tool_hook_ids()` / `_CallIds`, which read `session_id`, `turn_id` and `api_request_id`
and nothing else. So a plugin that asks "am I running attended or in a cron job?" can answer it for a
turn-level hook and cannot answer it for a tool-level one.

The consequence, measured on a plugin that does ask (`hermes-provenance-gate`): it refuses a
decision-shaped memory write when no verbatim quote backs it. A cron run has no user message of its
own, so in block mode every such write in that session is refused — not one in three, all of them —
with nobody watching. The plugin's own answer is to run in observe mode during unattended runs, and
it cannot tell when that is.

The patch carries `platform` from `agent.platform` into all three tool-path hooks, the same way the
turn-level hook already gets it, and adds it to a fourth place that documents the payload
(`hermes_cli/hooks.py::_DEFAULT_PAYLOADS`, the fixture behind `hermes hooks test` / `doctor`). Docs
in `website/docs/.../hooks.md` and its zh-Hans mirror list the field. Signature growth is
append-only: `platform` is the last parameter of `handle_function_call` and `_CallIds`.

## Timeline

All times UTC as returned by the API; Beijing time is +08:00.

| Time | Event |
|---|---|
| 2026-10-22 09:10:46 | Issue #142973 opened. |
| 2026-10-22 09:10:51 | PR #142974 opened against `main`, body says `Fixes #142973`. |
| 2026-10-22 09:10:52 | Issue auto-cross-references the PR. |
| 2026-10-22 09:10:56 | Comment on the issue: "Patch is up: #142974". |
| 2026-10-22 09:16:59–09:17:00 | `alt-glitch` labels the PR `area/i18n`, `comp/agent`, `comp/cli`, `comp/plugins`, `P3`, `type/feature`. |
| 2026-10-22 09:17:03 | `alt-glitch` labels the issue `comp/agent`, `P2`, `type/bug`. |
| 2026-10-22 09:17:23 | `alt-glitch` comments on the issue: a one-line summary of what the PR does, tagged `argus:v1`. |
| 2026-10-22 09:10:58 onward | Three workflow runs (CI, Docker Build/Test/Publish, Nix flake check) are created for the head and sit at `action_required`. No checks have executed. |

Nothing else followed: no comment on the PR, no request for review, no further push. The one later
action is the body edit described below.

## What changed in the text after submission

One edit, to the PR body, about twenty minutes after it was filed: "could not bootstrap **in this
sandbox**" became "could not bootstrap **on my machine**". The sentence describes a local run — the
repository's `scripts/run_tests.sh` pins a `uv` version and downloads it, and that download is
blocked from here — so "sandbox" was the wrong word. It is a body-only edit: no commit, head stays
`d08a850f92`, and `pr-142974-original.md` keeps the first version.

The issue body was not edited.

## Answers the record should not invent

- Nobody has reviewed this. The labels and the one comment come from a maintainer triaging, not
  from someone reading the diff.
- No CI result exists. `action_required` means the workflows are waiting for approval, not that they
  passed; saying "checks pass" here would be a claim about runs that never started.
- The local numbers (7 tests red on the base commit, green with the patch; `scripts/check` 12/12;
  regression suite 151 passed / 1 failed / 12 skipped) are in `evidence.md`, with the one failure
  explained there.
