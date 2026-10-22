# OpenAIModel: disable client retries on ApiModel.retryer replays (fixes request multiplication)

_Captured 2026-10-22T13:50:49Z via the GitHub API; the body below is the text as it stands on the platform._

---

Closes #3034.

On a rate limit, OpenAIModel and AzureOpenAIModel currently send 9 requests: ApiModel.retryer makes 3 attempts and the openai client inside each one keeps its default max_retries=2.

This change keeps the client's own retries on the first attempt and uses self.client.with_options(max_retries=0) only on the retryer's replays. A 429 now costs 3 + 1 + 1 = 5 requests. 5xx errors, dropped connections and timeouts are unchanged, because the retryer does not retry them and the client's retries are their only retry. Simply setting max_retries=0 on the client would drop those to 1 request (measured: results/e1/smol-openai-mr0/ in the repository below).

Measured against a local fake server, before -> after:
- 429, Retry-After: 1: 9 -> 5 requests
- 500: 3 -> 3
- dropped connection: 3 -> 3
- no answer: 3 -> 3

Tests: two new tests fail before the change and pass after; a third checks the first attempt keeps the client's retries. make quality passes.

Not in scope: how a rate limit is detected (#2186, #2382), and whether the retryer's waits should read Retry-After.

Raw logs and the fault server: https://github.com/simin-yuan/agent-double-exec-bench

AI disclosure: I directed this work. The patch, its tests and the measurements were produced by AI assistants I operate. Every number above is derived from the server logs in the linked repository, and I will answer review comments on it.