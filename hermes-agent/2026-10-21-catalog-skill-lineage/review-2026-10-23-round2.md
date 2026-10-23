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
and 1 MB in a fresh interpreter with a hard timeout. Nothing blows up: the largest single
measurement is `_COMMAND` on repeated `"$( "` at 0.469 s for 1 MB, and `_COMMAND` on backticks at
0.413 s.

The growth ratios in the first single-shot pass were taken while two other suites were running, and
one of them looked super-linear — `_PATH` on repeated `"./"` read x7.6 for x4 data. Re-measured on an
idle machine, best of three:

| size | time | step |
|---|---|---|
| 0.25 MB | 0.0145 s | |
| 0.5 MB | 0.0324 s | x2.23 |
| 1.0 MB | 0.0593 s | x1.83 |

That is linear (x4.08 for x4 data); the x7.6 was contention, not the pattern. It is worth saying
plainly because at 0.02 s scale the ratios are noise: the regression test therefore asserts two
things — the 1 MB case under 2.0 s, and a 4x data step under 8x time, a bar that rejects quadratic
blowup (x16) while leaving room for that noise.

`_EMAIL` is no longer expensive either: 0.004/0.007/0.019 s for `"a."` at 0.25/0.5/1 MB. The 2 MB
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

Against that parent the two files behave identically — but not for the reason I first wrote. The clean
measurement below is on the fixed tree; the parent was only ever run under the same contention, and
it failed in exactly the same place (`test_8`, `mklink`, 62.97 s), so the comparison is "same under
load", not "both measured quiet":

- `tests/test_e2e_hermes.py`: **40 passed in 299.87 s** as a whole file in one process, and 40/40 when
  run one test per process (8-14 s each). It runs. The earlier failures — `mklink` hanging past 60 s,
  `test_8` failing with `CalledProcessError` — happened while three suites plus the benchmarks were
  running at once; the hang traceback sits in `communicate()`, waiting for a stdout EOF that a
  contended machine does not deliver. So the junction is fine, my own load was the problem.
- `tests/test_hermes_strings.py`: the `built` fixture spawns `tests/hermes_strings_worker.py`, which
  asks Hermes to prepare an isolated runtime and gets stuck in dependency preparation —
  `preparing dependencies for this data root (the checkout's updates belong to
  C:\Users\Simon\AppData\Local\hermes)` → `Installing Python dependencies`, five full cycles in
  330 s, never reaching the first test. This one is a real blocker, and it is a harness-versus-live-
  install conflict: the file drives the live Hermes checkout on this machine, so it belongs to CI and
  should not be run here.

The earlier "the batch never finishes" is the same thing seen from outside: the worker loops in
dependency preparation and prints nothing, so the batch looks hung; the fixture's own `timeout=300`
(line 31) eventually raises `TimeoutExpired`, which means each of the 24 tests costs five minutes and
the file only "finishes" after two hours. Killing my shell instead of waiting left orphaned workers
behind (five, killed). The live runtime was checked afterwards — `.hermes-runtime/python` still dated
10-01, no package directory touched, checkout HEAD unchanged.

Neither file is touched by the change (the diff above lists every file). What the regex change
actually touches is covered by the runs that do finish: `tests/test_performance.py` **19 passed**
(four sub-process adversarial cases at a 20 s hard timeout), `tests/test_corpora.py` **213/213 and
174/174**, and the 3.11 run of everything else **732 passed, 3 skipped, 6 xfailed**. GitHub CI on
0.1.1 is green.

So the honest position for the re-pin: `tests/test_e2e_hermes.py` passes here — the whole file, 40
passed in 299.87 s — and `tests/test_hermes_strings.py` cannot run here because it starts a dependency
install against the live install. Neither outcome depends on the change. On CI both run, and CI is
green.

## Re-pin prepared, not opened

`provenance-gate` 0.1.0 → 0.1.1, commit `d85c508a0f` on a local branch `catalog/provenance-gate`,
`python scripts/validate_plugin_catalog.py plugin-catalog/provenance-gate.yaml` → `OK: 1 file(s)
valid`. Not pushed, no PR. Waiting for approval.

`degeneracy-guard`: checked, no quadratic pattern (`check_text` at 0.89/1.84/3.79/7.69 MB =
0.84/2.33/4.60/9.13 s), does not read `HERMES_HOME`, unchanged.
