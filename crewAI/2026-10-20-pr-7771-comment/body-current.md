# fix(llm): honor retry-after on SDK response objects

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

## Related issue

Follow-up to #7677. No separate issue; found while reviewing the newly merged retry implementation.

## Summary

Honor numeric `Retry-After` headers on SDK response objects, not just plain dictionaries. OpenAI's `RateLimitError.response` is an `httpx.Response` and its headers are an `httpx.Headers` mapping. Previously both sync and async retries ignored that hint and slept about one second even when the provider requested seven seconds.

Accept `Mapping` headers and read object response headers while retaining the existing AWS dictionary response handling. No new dependencies or retry-policy changes.

## Verification

- [x] Added a regression using a real OpenAI `RateLimitError` and HTTPX response, parameterized over sync and async execution.
- [x] Both cases fail on main (`4ed2abc7`): actual delay approximately 1s, expected 7s.
- [x] Full `lib/crewai/tests/llms/test_retry.py`: 16 passed after the fix.
- [x] Ruff check, format check and `git diff --check` passed for the changed code.

Tests inject sleep callbacks and make no network calls.

## Additional context

AI-assisted contribution. Tested on macOS with Python 3.13.

The contribution requires the `llm-generated` label. I requested it on creation, but this account cannot apply repository labels (HTTP 403); a maintainer will need to apply it.

### CI follow-up

The Python 3.13 group 7 job failed in the existing `test_infrastructure_flows_do_not_pollute_outcome_signals` telemetry test (missing `AgentExecutor` completion event). The retry suite and the entire telemetry test file pass together locally: 41 passed. I could not reproduce that failure locally. Re-running the failed CI job requires repository admin rights, which this account does not have.

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Narrow change to header parsing for retry delays; no retry policy or auth changes, covered by new regression tests.
> 
> **Overview**
> Fixes rate-limit retries ignoring provider **`Retry-After`** when errors carry an SDK **`response`** (e.g. OpenAI **`RateLimitError`** with **`httpx.Response`**) instead of a plain dict.
> 
> **`_retry_after_seconds`** now pulls headers from **`response.headers`** when they are not on the exception directly, and treats headers as any **`Mapping`** (not only **`dict`**) so **`httpx.Headers`** qualifies. Existing AWS-style dict **`ResponseMetadata`** / **`HTTPHeaders`** behavior is unchanged.
> 
> Adds a sync/async regression test using a real **`RateLimitError`** and **`Retry-After: 7`**, asserting the retry sleep is **7** rather than default backoff (~1s).
> 
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit e210cd873b0d57c435f13bf67d17a8deeb6ca8d1. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Current validation (2026-10-02)

Refreshed the branch onto main `8078f913` in `e85f588b`. The previous failed CI shard reported `test_infrastructure_flows_do_not_pollute_outcome_signals` (telemetry), rather than a retry-header regression.

Validation on the merged head: all 41 tests across `test_retry.py` and `test_flow_telemetry.py` pass, including that previously failing telemetry case. Ruff, formatting, targeted mypy and diff checks pass. The PR diff remains unchanged in scope: the retry-header fix and its regression tests. Fresh CI is running on the updated head.



---

## Comment `6290556192`, posted `2026-10-20T12:12:14Z`

A data point for this, measured with a local fake server on CrewAI 1.15.28, default settings:

- Every request answered `429` with `Retry-After: 3600` (quota exhausted): **9 requests within about 10 s**
  (gaps 1.3, 1.7, 0.1, 1.0, 1.8, 0.0, 1.0, 2.4 s). The `openai` client itself correctly refused to retry:
  in openai-python 2.54.0, `_should_retry` returns False when Retry-After exceeds `MAX_RETRY_AFTER_DELAY`
  (120 s; `_base_client.py` L823–829, `_constants.py` L15). The two CrewAI layers above it retried anyway.
- Every request answered `429` with `Retry-After: 1`: 27 requests (three layers of 3).

Logs and a runner: https://github.com/simin-yuan/agent-double-exec-bench (`results/e1/crewai/429-hard/` and `results/e1/crewai/429/`).
