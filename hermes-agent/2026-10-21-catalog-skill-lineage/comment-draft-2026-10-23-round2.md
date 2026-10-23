Fixed in 0.1.8 — `c21178d4578680297b6100638fff59b75d1b572e`.

Thanks for the quadratic e-mail case and for the bounded form; I used it as written, at the
definition in `textfeat.py:20` only (`(?<![\w.+-])[\w.+-]{1,64}@[\w-]+(?:\.[\w-]+)+\b`) —
`anchor_strings` and its call sites are unchanged. On `"a."` repeated: 40 KB 4.85 s →
0.0009 s, 200 KB 149.87 s → 0.005 s, 2 MB (never finished under the old pattern) → 0.049 s.
End to end, `anchor_strings` on 2 MB is 0.63 s.

One behavioural difference remains against 0.1.4: a local part longer than 64 octets is no longer
an anchor. I compared 13 samples and the other 12 are identical, `bob.smith+x@example.co.uk`
among them. A leading `.`, `+` or `-` is stripped from the match, so `-name@example.com` still
records `name@example.com` rather than a second hash of the same address.

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
