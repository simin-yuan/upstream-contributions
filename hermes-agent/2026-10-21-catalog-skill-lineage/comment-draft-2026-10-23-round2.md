Fixed in 0.1.8 — `c21178d4578680297b6100638fff59b75d1b572e`.

Thanks for the quadratic e-mail case and for the bounded form; I used it as written at the
definition in `textfeat.py:20` (`(?<![\w.+-])[\w.+-]{1,64}@[\w-]+(?:\.[\w-]+)+`) and left the
pattern itself alone. Behaviour is fitted back on the one line that consumes it, described below.
On `"a."` repeated: 40 KB 4.85 s → 0.0009 s, 200 KB 149.87 s → 0.005 s, 2 MB (never finished
under the old pattern) → 0.049 s. End to end, `anchor_strings` on 2 MB is 0.63 s.

One behavioural difference remains against 0.1.4: a local part longer than 64 octets is no longer
an anchor. I compared 13 samples end to end and the other 11 are identical, `bob.smith+x@example.co.uk`
among them. The bounded pattern on its own would also start a match at a preceding separator, so
`anchor_strings` strips a leading `.`, `+` or `-` from the match and drops it if nothing is left —
that is what keeps `-name@example.com` recording `name@example.com` instead of a second hash of the
same address.

The same treatment went into `tests/test_textfeat_perf.py`: each of the eight patterns in
`textfeat.py` is timed on hostile input built from its own structure — 21 shapes at 0.25, 0.5 and
1 MB, every case in its own process under a hard timeout, so a pathological pattern fails the build
instead of hanging it. The worst is `_COMMAND` on backticks at 0.414 s for 1 MB; all eight scale
linearly.

`HERMES_HOME`: `skills.py` now asks `hermes_constants.get_hermes_home()` on each call (imported
inside the function, with `HERMES_HOME` and then `~/.hermes` as fallbacks for older releases), so
skill lookup follows the session's profile instead of the launch profile. Two tests cover the
resolution order.

While in there I checked my other two catalog plugins for the same class of problem.
`hermes-provenance-gate` had two patterns that went quadratic on whitespace — 0.67 s and >30 s on
1 MiB of spaces, which is inside the 2 MB a tool result can be, and `re` cannot be interrupted
once it is in there. Both scan linearly now (<10 ms), with sub-process tests at a 20 s hard
timeout; the corpus results are unchanged. `hermes-degeneracy-guard`: none, measured linear from
0.9 to 7.7 MB. The provenance-gate re-pin will come separately.

131 passed, 1 skipped on 3.11.16 and 3.14.7; `hermes plugins validate .` clean; the
`hermes-tools` job is green on the pinned commit.
