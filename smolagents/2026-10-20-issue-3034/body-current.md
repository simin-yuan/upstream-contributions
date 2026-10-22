# OpenAIModel: client retries and ApiModel retries multiply (9 requests per rate limit)

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

### What happens
`OpenAIModel` (and `AzureOpenAIModel`) wraps `client.chat.completions.create` in `ApiModel.retryer`
(3 attempts). The `openai` client inside keeps its default `max_retries=2`. On a 429 every outer attempt
makes 3 requests, so one rate limit costs 9 requests.

### Minimal repro
Any endpoint that always answers `429` with `Retry-After: 1`:

    from smolagents import OpenAIModel
    model = OpenAIModel(model_id="any", api_base="http://127.0.0.1:PORT/v1", api_key="dummy")
    model.generate([{"role": "user", "content": [{"type": "text", "text": "hi"}]}])

Measured against a local fake server (main @ 96f33fa, openai 2.54.0, Python 3.13, default settings):

| fault (every request) | now: requests / time to error | with the change below |
|---|---|---|
| 429, Retry-After: 1 | 9 / 835 s | 5 / 600 s |
| 500 | 3 / 16 s | 3 / 17 s |
| connection dropped | 3 / 16 s | 3 / 17 s |
| no answer | 3 / 1816 s | 3 / 1817 s |

Each cell includes about 10 s of Python start-up and framework import (elapsed time minus the step duration
the framework logs); in the 500 and dropped-connection cells the actual call therefore took about 5 s of the 16.
The 429 times vary between runs because the waits are jittered.
Raw logs (`results/e3/before/` and `results/e3/after/`): https://github.com/simin-yuan/agent-double-exec-bench

### Proposed change (small, `OpenAIModel` only)
Keep the client's own retries on the first attempt; on the retryer's replays use
`self.client.with_options(max_retries=0)`. 5xx, connection errors and timeouts behave exactly as today
(only the client retries them); a rate limit costs 3 + 1 + 1 requests instead of 3 × 3.
The OpenAI Agents SDK does the same (`run_internal/model_retry.py`: provider retries stay on for the
first attempt and are switched off on the runner's own replays).

This fixes the request count, not the waiting time: the retryer's two waits (120–240 s, then 240–960 s)
do not read `Retry-After`. Whether they should is a separate question, so I left it out of this change.

Not in scope: how a rate limit is detected. #2186 and #2382 propose status-code detection; one data point
for them: a 500 whose message contains "rate limit" or "429" gets the rate-limit path (9 requests instead of 3),
and the same 429 response whose body does not mention rate limits costs 9 requests through `OpenAIModel`
(the client's message includes "Error code: 429") but 3 through `LiteLLMModel` (its message does not).

I have a patch with tests (they fail before and pass after; `make quality` and the full test suite pass;
log in `upstream/TEST_LOG.txt` of the same repository)
The fix is in #3036.

AI disclosure: I directed this work. The instrumented runs, the patch and its tests were produced by AI
assistants I operate. Every number above is derived from the server logs in the linked repository.


---

## Comment `6292983610`, posted `2026-10-20T14:16:46Z`

Thanks for the interest, Shivansh! I have a patch with tests ready and will open the PR today.

One thing I measured while writing it: constructing the client with max_retries=0 also removes the only retries smolagents currently has for 5xx errors, dropped connections and timeouts, since ApiModel.retryer only catches rate limits. With max_retries=0, a single 500 fails the run after 1 request instead of 3 (results/e1/smol-openai-mr0/ in https://github.com/simin-yuan/agent-double-exec-bench).

So the patch keeps the client's own retries on the first attempt and turns them off only on the retryer's replays, the same pattern the OpenAI Agents SDK uses: 3 + 1 + 1 = 5 requests on a 429, everything else unchanged. Happy to have your review on it.