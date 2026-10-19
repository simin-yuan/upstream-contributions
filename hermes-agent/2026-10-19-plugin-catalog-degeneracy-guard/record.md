# Hermes Agent — catalog entry for `hermes-degeneracy-guard` (1.3.0)

| | |
|---|---|
| Pull request | [#141341](https://github.com/NousResearch/hermes-agent/pull/141341) |
| Submitted | 2026-10-19 10:20 (UTC+08:00) — `02:20:48Z` |
| State | open, ready for review, mergeable |
| Labels | `P3`, `type/feature`, `comp/plugins`, `area/streaming` |
| Pinned commit | `ffc28d0ab9ac6b6c0cbb118ed5fb0c06909a3dc1` (plugin version 1.3.0) |

## What it is, and why it mattered

Hermes' core truncates runaway text, and by design lets a provider-confirmed tool call through even
when its arguments are repetition-dominated. That exception exists because a repeated payload can be
legitimate — a table of zeros, a seed script with forty identical rows — so the core cannot decide
on content alone. The plugin covers the same gap from the other side: at `pre_tool_call`, when the
arguments look like a repetition loop rather than data, it returns `approve` and lets Hermes' own
approval gate ask the user. A denial, an error, or an unanswered prompt refuses the call, which
keeps cron and other unattended runs on the safe side; "always" stops the prompts for that tool.

The catalog entry is what makes it installable through the plugin catalog and therefore
discoverable. The paragraph above is also the argument of the PR: the plugin does not override the
core decision, it borrows the mechanism the core already has for "a human should look at this".

## Timeline

| Time (UTC) | Event |
|---|---|
| 2026-10-19 02:20:48 | PR opened, two commits: the entry, and the contributor-email mapping. |
| 2026-10-19 02:27:02 | @alt-glitch labeled it `area/streaming`, `comp/plugins`, `P3`, `type/feature`. |
| 2026-10-19 04:29:18 | Converted to draft while the plugin's detection was being redesigned. |
| 2026-10-19 08:18:17 | Branch rebased onto current `main` and force-pushed; the pin moved 1.1.0 → 1.3.0. |
| 2026-10-19 08:47:19 | Marked ready for review again. |
| 2026-10-19 08:47:39 | Cross-referenced by the listing request in `0xNyk/awesome-hermes-agent` (#524). |

The PR body carries a "Changes since this PR was opened" section, because the substance changed
while it was open: version 1.1.0 refused legitimate repetitive writes, and 1.3.0 asks instead of
refusing. A reviewer who only looks at the diff would see a version bump; the body says why.

CI sits at `completed / action_required` — the first-time-contributor approval gate, not a failure.

## What changed in the text after submission

`pr-body-original.md` is the body as first posted (pinned to 1.1.0, no template sections).
`pr-141341-current.md` is the body after the rewrite into the repository's pull request template:
the same technical argument, plus a type-of-change box, a checklist ticked only where it can be
defended, the measured numbers, and the "changes since opened" paragraph. GitHub does not serve
earlier body revisions over its API, so the original here is the text recorded on this machine when
it was posted.

## Contents

- `pr-body-original.md`, `pr-141341-current.md`
- `snapshot.json` — raw API payloads as returned on 2026-10-19 16:58 +08:00
- `patch/0001-add-degeneracy-guard-catalog-entry.patch` — the entry commit only
- `evidence.md` — validation, doctor, catalog validation, tests, the hold-out numbers, the CI run
- `archive.md` — Internet Archive snapshots
