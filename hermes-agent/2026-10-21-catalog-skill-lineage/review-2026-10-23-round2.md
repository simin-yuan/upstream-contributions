# PR #142381 — second round of fixes, 2026-10-23

Round 2 after Simon's review of round 1. Nothing here was published as a comment: the comment is
drafted and held for approval, and no new issue or PR was opened against hermes-agent.

## P0-1 — Python floor for the possessive quantifiers

`hermes-agent` declares `requires-python = ">=3.11,<3.15"` (pyproject.toml). The same line at
`v0.22.2`, which is the oldest Hermes `provenance-gate` claims (`requires_hermes: ">=0.22.2"`), is
`>=3.11,<3.15` too. Possessive quantifiers and atomic groups are Python 3.11 features, so every
Hermes release that can load the plugin can parse them.

Checked on the floor itself: `uv run --python 3.11 … pytest` in `pg-main-wt` with
`tests/test_hermes_strings.py` and `tests/test_e2e_hermes.py` excluded (both need a live Hermes;
they skip without one, and the ones that do run are in the 732 below):
**732 passed, 3 skipped, 6 xfailed**. `skill-lineage` 0.1.8: **131 passed, 1 skipped** on 3.11.16
and on 3.14.7. `skill-lineage` uses no 3.11-only syntax — the review's change is a regex and a
function-local import.

Decision: keep the possessive form; no rewrite.

## P0-2 — the red `hermes-tools` job

The failure is not one thing:

| commit | job step that failed | failing tests |
|---|---|---|
| `8df4bcb` (0.1.3) | `hermes plugins validate` | none — the e2e step passed |
| `554a951a` (0.1.4) | all tests, end-to-end included | `test_foreground_verbatim_unrelated_and_user_rule[plain]`, `[stream]` |
| `466345a` (0.1.6) | all tests, end-to-end included | the same two |

The validate failure at `8df4bcb` was already gone by 0.1.6 (that step passes there). The e2e
failure starts at 0.1.4, which is the release that added `6bd396e`.

Cause: `6bd396e` dropped user turns from the edge pass outright —

```python
if item.cls == "user":
    continue
```

— with the comment "the eval's user-dictated floor is 0/200". The floor is about the *page* read in
the same session, and the same-class discount already prevents that page from being credited; the
`continue` additionally removed the turn as an origin, so a rule the user dictates has no origin at
all. The plugin's own recorded smoke run (`tests/smoke-2026-10-21.txt`, before that release) shows
what it did: `src:user:26edf93a… [strong, 5 shingles; into skill smoke-probe]`.

Fix (0.1.7): drop the `continue`, keep the same-class discount, and replace the one test that
asserted the stronger rule with two tests — a dictated rule is traced to the turn, and a page read
in the same session is still not credited for it.

The eval is unchanged: 240 scenarios, D1 80/80, D2 40/40, D4 40/40, N1 0/160, N2 0/80, N3 0/80,
precision 1.0, every pre-registered floor met.

CI: v0.1.7 `3b667043caaa96b821982e9e0c423920f6fd3dee` →
https://github.com/simin-yuan/hermes-skill-lineage/actions/runs/40013256285 (success, all 9 jobs).
v0.1.8 `c21178d4578680297b6100638fff59b75d1b572e` →
https://github.com/simin-yuan/hermes-skill-lineage/actions/runs/40013410910 (success).

## P0-3 — the extra behaviour change is gone

The bounded pattern stays exactly as reviewed. `anchor_strings` strips a leading `.`, `+` or `-`
from the match and drops the result if nothing is left, so `-name@example.com` records
`name@example.com` as 0.1.4 did. Without that, an address next to a separator hashed one way in an
older chain and another way now, and one address counted as two.

Compared against the 0.1.4 pattern on 13 samples: **2 differ**, both a local part longer than 64
octets (`prefix.` + 63 chars, and 65 chars), which is the one intended difference.

Timing with the post-processing: `anchor_strings` on 2 MB of `"a."` = 0.631 s (0.664 s before it,
run-to-run noise).

## P1-4 — hostile input for all eight patterns

21 shapes, three per pattern, built from each pattern's own structure; each measured at 0.25, 0.5
and 1 MB in a fresh interpreter with a hard timeout. Nothing is super-linear: the 4x size step
grows time by x2.4–x5.6. The slowest shape is `_COMMAND` on repeated `"$( "` — 0.469 s at 1 MB in
the 21-shape matrix, and x4.2 for x4 data in a separate growth run:

| size | time |
|---|---|
| 1.0 MB | 0.419 s |
| 2.1 MB | 0.849 s |
| 4.2 MB | 1.756 s |

`_EMAIL` is no longer expensive: 0.003/0.007/0.015 s for `"a."` at 0.25/0.5/1 MB. The 2 MB
reproducer through `anchor_strings` stays at 0.63 s against the 5 s bar. All of it is in
`tests/test_textfeat_perf.py` (43 tests, 16 s), each case in its own process.

## P1-5 — deferred

Creating a temporary profile and driving a real session needs Simon present, because the machine
running the test is the machine running Hermes. Left for tomorrow; the two unit tests for the
resolution order (`get_hermes_home()` wins over `HERMES_HOME`; the value is read per call, not at
import) stay as the coverage until then.

## P1-6 — why those two files never finish here

Two corrections before the numbers.

`8813469d` is not the parent of the fix. The e2e harness was rewritten after it (`f65eddf`, "share
one runtime store per run, and make the e2e install idempotent"): at `8813469d` the file has no
`mklink` call at all and every e2e test fails or hangs on that older path, so comparing against it
measures the harness generation, not the regex change. The real parent is `c83fdf6^`, and the fix
touches five files of which one is code:

```
CHANGELOG.md   plugin.yaml   provenance_gate/__init__.py (version)
provenance_gate/claims.py  +11/-2        tests/test_performance.py  +47
```

Against that parent the two files behave identically, and both are blocked by the environment:

- `tests/test_e2e_hermes.py:76` runs `subprocess.run(["cmd", "/c", "mklink", "/J", …])` to junction
  the real installs directory into a temp home. On this desktop that call does not complete —
  reproduced outside the repo, into a fresh temp dir, with a 20 s timeout: `TIMEOUT >20s`. Inside
  the suite it returns exit 1 and the test fails.
  `test_8_non_claim_writes_pass_untouched[plain]`, the first test that reaches the junction:
  **pre-fix `1 failed in 62.97s`, post-fix `2 failed in 157.83s`** — same command, same
  `CalledProcessError: … returned non-zero exit status 1`, only the duration differs.
- `tests/test_hermes_strings.py` blocks in its `built` fixture: `subprocess.run([... ,
  hermes_strings_worker.py])` never returns, and neither tree finishes it.

Neither file is touched by the change (the diff above lists every file), and the failure text names
the junction, not claim parsing. What the regex change actually touches is covered by the runs that
do finish: `tests/test_performance.py` **19 passed** (four sub-process adversarial cases at a 20 s
hard timeout), `tests/test_corpora.py` **213/213 and 174/174**, and the 3.11 run of everything else
**732 passed, 3 skipped, 6 xfailed**. GitHub CI on 0.1.1 is green — that machine can create the
junction, this one cannot.

So the honest position for the re-pin: the two files cannot run on this desktop before or after the
change, for a reason outside the plugin; on CI they run, and CI is green.

## Re-pin prepared, not opened

`provenance-gate` 0.1.0 → 0.1.1, commit `d85c508a0f` on a local branch `catalog/provenance-gate`,
`python scripts/validate_plugin_catalog.py plugin-catalog/provenance-gate.yaml` → `OK: 1 file(s)
valid`. Not pushed, no PR. Waiting for approval.

`degeneracy-guard`: checked, no quadratic pattern (`check_text` at 0.89/1.84/3.79/7.69 MB =
0.84/2.33/4.60/9.13 s), does not read `HERMES_HOME`, unchanged.
