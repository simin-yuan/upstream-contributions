# Hermes Agent — `hermes plugins validate` rejects the legacy `hooks:` key that the loader reads

| | |
|---|---|
| Issue | [#141339](https://github.com/NousResearch/hermes-agent/issues/141339) |
| Pull request | [#141340](https://github.com/NousResearch/hermes-agent/pull/141340) |
| Submitted | 2026-10-19 10:20 (UTC+08:00) — issue `02:20:22Z`, PR `02:20:33Z` |
| State | issue open; PR open, mergeable, one commit on current `main`; workflows awaiting maintainer approval |
| Labels | `P2`, `type/bug`, `comp/cli`, `comp/plugins` (both items) |
| Files in this folder | the issue and the pull request, side by side, because the PR exists only to close the issue |

## What it is, and why it mattered

`hermes plugins validate` and the plugin loader read the same manifest differently. The loader
resolves declared hooks as `provides_hooks`, falling back to the legacy `hooks:` key added in
#108371; the validator read only `provides_hooks`. A third-party plugin that declared its hooks
under `hooks:` therefore loaded and ran correctly, and failed admission with "undeclared hooks
registered (not in provides_hooks)" — including in the catalog gate, where validation is the
precondition for listing. The fix moves the resolution rule into one helper,
`hermes_cli.plugins_manifest.hooks_manifest_key()`, and has both sides call it, so the two cannot
drift apart again. When only the legacy key is present, validation now passes and adds a warning
that recommends `provides_hooks`. The change does not touch loader behaviour, the
`provides_tools` / `provides_middleware` checks, or any bundled manifest.

## Timeline

All times UTC as returned by the API; Beijing time is +08:00.

| Time | Event |
|---|---|
| 2026-10-19 02:20:22 | Issue #141339 opened. |
| 2026-10-19 02:20:33 | PR #141340 opened against `main`, two commits. |
| 2026-10-19 02:20:34 | Issue cross-referenced by #141340. |
| 2026-10-19 02:32:32 | @alt-glitch labeled the PR `comp/cli`, `comp/plugins`, `P2`, `type/bug`. |
| 2026-10-19 02:32:34 | @alt-glitch labeled the issue with the same four labels. |
| 2026-10-19 02:37:29 | Issue and PR both cross-referenced by [volcengine/OpenViking#6257](https://github.com/volcengine/OpenViking/pull/6257) ("fix(hermes): remove legacy empty hooks field"), a third-party project hitting the same validator. |
| 2026-10-19 07:51:57 | @alt-glitch on the issue: |

> #141340 aims to fix this: it makes the plugin validator accept hooks declared under the legacy `hooks` manifest key as the declared hook list, the same way the loader already reads them.
>
> — `alt-glitch`, 2026-10-19 07:51:57Z

| Time | Event |
|---|---|
| 2026-10-19 08:43:50 | Branch rebased onto the current `main` and force-pushed (`head_ref_force_pushed`); the two commits and their authorship are unchanged. |
| 2026-10-19 08:47 | PR body rewritten in the repository's pull request template; issue body extended with an Environment section. |
| 2026-10-21 05:06:29 | Rebased onto `main` at `cf4f3a7` ("feat(ci): pin docker buildx version to v0.31.8") and force-pushed with `--force-with-lease`, head `c5e60648` → `341b152d`. The contributor-email commit was dropped automatically ("patch contents already upstream"): that file had already reached `main` through the catalog PR. Author `Simin (Simon) Yuan <yleven120@gmail.com>` and author date unchanged; committer the same identity; no trailers. The PR now shows one commit. |
| 2026-10-21 05:11:28 | Three workflow runs created on the new head — `CI`, `Nix flake check`, `Docker Build, Test, and Publish`. All three conclude `action_required`; the head still has zero check runs. |

CI on the head commit sits at `completed / action_required`, as it did before the rebase: approval is
recorded per pull request, so a new push does not inherit one. Evidence from the two catalog PRs —
#141341 ran its workflows once a maintainer had approved that PR, and later pushes executed without
a fresh approval; #141340 has never been approved, and its three runs on the 2026-10-21 head are
`action_required` again. `action_required` is not a failure: nothing executed, and no step log exists.

## Local verification before the 2026-10-21 push

The rebased branch was tested on this machine before it was pushed, with the repository's own runner
(`scripts/run_tests.sh`) and the local Python 3.11 environment:

| Test file | Result |
|---|---|
| `tests/hermes_cli/test_plugin_validate.py` | 49 passed, 1 failed |
| `tests/hermes_cli/test_plugin_capabilities.py`, `test_plugin_catalog.py` | 49 passed, 0 failed |
| `tests/hermes_cli/test_plugins_loader.py`, `test_plugin_manifest_v2.py`, `test_plugin_install_manifest_version.py` | 30 passed, 2 failed, 1 file did not collect |

The same failures, and the same file that does not collect, reproduce on an untouched checkout of
`main` at `cf4f3a7` in a separate worktree. They are properties of this Windows environment, not of
the change, and the change adds no failing test. Whether they pass on the CI platforms is not
established here.

## What changed in the text after submission

Both items were edited after they were opened, so this folder keeps the first version and the
current version of each.

- `issue-body-original.md` → `issue-141339-current.md`: an **Environment** section was appended
  (reproduced on Linux and Windows, Python 3.14, two Hermes commits, plus the before/after output
  of `hermes plugins validate`). Nothing above it was altered; the appended block starts after the
  original final line.
- `pr-body-original.md` → `pr-141340-current.md`: the body was rewritten into the repository's PR
  template (section headings, a type-of-change box, a checklist, and a note that the second commit
  maps the contributor email). The technical description of the fix is carried over unchanged.

GitHub does not expose earlier revisions of an issue or pull request body through its API, so the
"original" files here are the texts as recorded on this machine when they were posted, not a
retrieval from GitHub. Where a claim depends on that, it is the version kept here that is quoted.

## Contents

- `issue-body-original.md`, `issue-141339-current.md`, `pr-body-original.md`, `pr-141340-current.md`
- `snapshot.json` — raw API payloads (item, comments, timeline, files) as returned on 2026-10-19 16:58 +08:00
- `patch/0001-validate-accepts-the-legacy-hooks-key.patch` — the substantive commit only; the
  contributor-email mapping commit is excluded on purpose
- `evidence.md` — reproduction, before/after output of the validator, the tests and the lint lane
- `archive.md` — Internet Archive snapshots
