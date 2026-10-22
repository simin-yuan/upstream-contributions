# `Retrying` ignores `Retry-After`, and the first retry already waits 2–3× `wait_seconds`

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

## What happens

Two things, checked on `smolagents==1.26.0` and on current `main` (96f33fa).

1. **`Retry-After` is never read.** `grep -ri "retry-after" src/` returns nothing. When a provider answers `429` and says "come back in 3 seconds", the model-level retry sleeps for whatever the exponential schedule says instead.

2. **The first retry waits 2–3× the configured `wait_seconds`.** Is multiplying before the first sleep intended? In `Retrying.__call__` the delay is multiplied by `exponential_base` (plus jitter) and *then* slept, so the first wait is `wait_seconds × 2 × (1 + jitter)` rather than `wait_seconds`. Measured below with `RETRY_WAIT=2`: the first gap between retry groups is **5.16 s**, where the configured value was 2 s.

With the `ApiModel` defaults (`max_attempts=3`, `wait_seconds=60`, `exponential_base=2`, jitter on) this means a single rate-limited call sleeps 120–240 s and then 240–960 s — **6–20 minutes** before it gives up.

## Reproduction

A local OpenAI-compatible server that answers the first *N* `/v1/chat/completions` requests with `429` and a `Retry-After: 3` header, then a normal response; one `OpenAIModel` call through a `ToolCallingAgent`.

```bash
RETRY_WAIT=2 python r429.py     # RETRY_WAIT only scales the waits down so the run finishes quickly
```

Requests the server received for one logical call, with `Retry-After: 3` on every 429:

| # | t (s) | `x-stainless-retry-count` | note |
|---|-------|---------------------------|------|
| 1 | 2.25 | 0 | first request |
| 2 | 5.25 | 1 | 3.00 s later — the openai client *does* honour the header |
| 3 | 8.25 | 2 | 3.00 s later |
| 4 | 13.41 | 0 | **model-level retry: 5.16 s after the previous attempt** |
| 5 | 16.41 | 1 | 3.00 s |
| 6 | 19.41 | 2 | 3.00 s |
| 7 | 34.23 | 0 | **14.82 s later** |
| 8 | 37.23 | 1 | 3.00 s |
| 9 | 40.23 | 2 | 3.00 s |

So: 9 HTTP requests for one logical call. The openai client's own retries honour `Retry-After` (3.00 s apart, the counter going 0→1→2), but the two waits between retry groups are **5.16 s** and **14.82 s** — with `RETRY_WAIT=2` those should have been 2 s and 4 s if the schedule started at `wait_seconds`, or 3 s if the header had been used. Neither wait relates to the `Retry-After: 3` the server sent.

Raw data (public, reproducible): `evidence/e0-retry-after/ra3_smol.jsonl`, `r429.py` (the server) and `NOTE.md` in https://github.com/simin-yuan/agent-double-exec-bench.

## Why it matters

On a 429 with a short `Retry-After`, the agent stays blocked for minutes when the provider was ready in seconds. With `max_attempts=3` and the default 60 s base, the worst case is 20 minutes of wall-clock for one call, which is longer than most CI jobs and than most people's patience with a CLI.

Related but different: #2586 reports that `wait_seconds` defaults to `0.0` (so backoff is off) and that `reraise` has no effect — that is about the default value and the error path. This one is about the schedule when a wait *is* configured, and about ignoring a server-declared cooldown.

## What I'd expect

- If the server sends `Retry-After`, sleep that long — and never sleep less than the server asked for.
- If the doubling before the first sleep is intentional, a comment saying so would help: as written, `wait_seconds` means "half of the first wait", which is easy to misread. If it isn't intentional, the first retry should wait `wait_seconds` and doubling should start after that.

## Status

A patch with five tests is ready locally (first-retry delay, jitter bounds, `Retry-After` honoured, opt-out flag, fallback when the header is absent). I have not opened a PR: I have one open PR (#3036) and I'll follow the one-open-PR rule until it lands. Happy to post the diff here, or to open the PR as soon as the issue is accepted and #3036 is resolved — whichever the maintainers prefer.

---

*Disclosure: I used an AI assistant to help draft this issue and to write and run the patch. I have read the change, run the tests, and will answer review comments.*


---

## Comment `6297488568`, posted `2026-10-20T17:53:55Z`

Prior art on one of the two points, which I should have checked before opening this.

**#2596** (opened 2026-08-02, still open) fixes exactly my point 2: it moves the sleep before the
multiplication, so the first retry waits `wait_seconds` and the doubling starts after that. Its tests
assert sleeps of 1, 2, 4 with the default. I re-derived the same change without knowing that PR existed;
that finding is theirs, not mine.

**#2662** (2026-08-20) and **#2617** (2026-08-11) touch the same class but only bring the `wait_seconds`
default to 1.0 and make `reraise` wrap into `RetryError`. Neither moves the sleep, so neither covers
point 2, and neither has had a maintainer reply either. All three edit the same lines of
`Retrying.__call__`, and `#2586` — which all three say they fix — has been open since 2026-07-29 without
a label.

What is left that none of them has: `Retry-After` is never read (`grep -ri "retry-after" src/` returns
nothing), and the cost of that is measurable — with `RETRY_WAIT=2` the waits between retry groups were
5.16 s and 14.82 s while the server asked for 3 s, 40 s of wall clock for one logical call, and at the
shipped defaults (`wait_seconds=60`) 6–20 minutes.

So the honest state of this issue: if #2596 lands, point 2 is closed and only the header work is left. I
would rather say that than leave four competing changes over the same two lines.

*Disclosure: I used an AI assistant to help draft this comment. I will answer replies myself.*