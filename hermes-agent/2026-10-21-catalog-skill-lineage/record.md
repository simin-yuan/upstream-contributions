# Hermes Agent — catalog entry for `skill-lineage` (0.1.0)

| | |
|---|---|
| Pull request | [#142381](https://github.com/NousResearch/hermes-agent/pull/142381) |
| Submitted | 2026-10-21 13:08 (UTC+08:00) — `05:08:14Z` |
| State | open 2026-10-21 — last touched `2026-10-21T09:29:38Z` |
| Labels | `P3`, `type/feature`, `comp/plugins` |
| Mergeability | `blocked` (a review is required), one file, `+22 −0`, two commits |
| Pinned commit | `7cc92a53fb3e6f1a4ff0db5dfe951fcbe048a9da` — the plugin repository at 0.1.0 |
| Repository | <https://github.com/simin-yuan/hermes-skill-lineage> — public since 2026-10-21 |

## What it is, and why it mattered

For every skill write — a foreground `skill_manage` call, a background-review edit, or a curator
consolidation — the plugin stores the skill's content fingerprint before and after, the sources that
were present earlier in the same session (the text the user typed, tool results, skills that were
read), the features that link a source to the added lines, and a `seq → prev_hash → hash` chain over
the entries. `trace` answers where a version came from, `descendants` goes the other way, `report`
lists versions carrying lines of no traceable origin, `verify` re-walks the chain, and `head` prints a
value to write down elsewhere, so a later run can tell truncation from an intact chain.

The submission could not go straight to a pull request. The plugin imported `tools.skills_tool`, a
private core module; catalog rule 9 is a decline rather than a bug-fix request, so an entry filed
with that import would have been refused on sight. The fix landed in the plugin first — `locate()`
now scans `$HERMES_HOME/skills` — and the repository stayed private until the fixed build passed
validation with zero warnings. That order is the reason the entry was never declined.

## Timeline

| Time (UTC) | Event |
|---|---|
| 2026-10-20 20:14:36 | Repository created, private, to hold the timestamp before anything was published. |
| 2026-10-21 04:05:28 | Rule-9 patch pushed (`33631f8`): private core import removed, validation 1 warning → 0. |
| 2026-10-21 05:07:35 | README and the end-to-end smoke log committed (`96ffb13`); the repository was flipped public between this commit and the next one. |
| 2026-10-21 05:08:12 | Entry commit on the branch (`9c20143`), pinned to `96ffb13`. |
| 2026-10-21 05:08:14 | PR opened by `simin-yuan`. |
| 2026-10-21 09:29:20 | Smoke log sanitized in the plugin repository (`7cc92a5`): the absolute path of the isolated data root was replaced with a placeholder; every command and output in the log is unchanged. |
| 2026-10-21 09:29:36 | Branch re-pinned to `7cc92a5` (`281433a`) so the catalog points at the sanitized build. |
| 2026-10-21 09:29:38 | Labels `type/feature`, `comp/plugins`, `P3` present; `mergeable_state` `blocked`. |

## What changed in the text after submission

Two commits, and the second one is a correction of the first: the pin moved because the plugin
repository itself changed after the entry was filed. Nothing in the entry's text changed between the
two commits — the description, capabilities and disclosure are as first posted. `pr-body-original.md`
is the body as filed.

## Contents

- `pr-body-original.md`, `skill-lineage.yaml` — the body as filed, and the entry as it stands on the
  branch at `281433a`
- `snapshot.json` — the PR payload as returned by the API on 2026-10-21 17:29 +08:00
- `pr-files-142381.json` — the files payload at the same moment
- `evidence.md` — validation, tests, the end-to-end run, and the rule-9 fix
- `archive.md` — the Internet Archive capture and its read-back
