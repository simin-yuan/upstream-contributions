# Text as filed

Issue body exactly as submitted, copied from the API (`body` field, `created_at` 2026-10-19T13:15:07Z).

---

### What variant of Codex are you using?

CLI (`codex exec`), 0.163.0

### What feature would you like to see?

A client-side check that ends a streamed reply once it has clearly turned into a repetition loop, instead of reading it to the model's output limit.

**What happens today.** The main turn request carries no output cap (`ResponsesApiRequest` in `codex-rs` has no `max_output_tokens` field), and I couldn't find a mid-stream repetition check. I pointed `codex exec` at a local Responses endpoint that streams one sentence on repeat, about 300,000 characters. Codex read all of it and printed all 300,049 characters to stdout:

```sh
export LOOP_KEY=dummy
codex exec --skip-git-repo-check \
  -c 'model_providers.loop={name="loop",base_url="http://127.0.0.1:8765/v1",wire_api="responses",env_key="LOOP_KEY"}' \
  -c model_provider=loop -m loop-model "hello" < /dev/null > out.txt
wc -c out.txt   # 300049
```

The endpoint and runner are here: https://github.com/simin-yuan/agent-runaway-bench

**Why it matters for `codex exec`.** Nobody is watching to press Ctrl-C. In CI or a scheduled job the loop goes into the log or the next step, and the turn runs until the model's own limit. Repeating and degenerate output has been reported here recently (#40957, where the app streamed repetitive garbage tokens into the output, and #40369), so this isn't only a synthetic case.

**A possible approach.** Hermes Agent does this on its chat-completions and Anthropic streams (`RunawayStreamWatch`, NousResearch/hermes-agent#130551): it watches the streamed text, and once a long stretch is dominated by verbatim copies of a few lines, it closes the stream and ends the turn with a clear message. It only fires on long repetition (tens of thousands of characters), so a reply where the user asked for repetition, or a table with similar rows, still comes through. Two details that came up when I wired the same check into two more of Hermes's streams (NousResearch/hermes-agent#141479): skip any drain of the remaining stream after the cut, and drop a tool call that was cut halfway, so the next request doesn't carry a call without a result.

### Additional information

- This measures the client only. I didn't test whether the hosted Responses API stops a looping generation on its side; if it does, the cost side of this is smaller, though `codex exec` would still print the loop up to that point.
- Measured at `openai/codex` main `89461ea` and the 0.163.0 npm release, Ubuntu 24.04.
- Same test against other frameworks, for comparison: Hermes stops at about 16,000 characters; OpenAI Agents SDK 0.24.0, smolagents 1.26.0 and LangGraph 1.2.16 read the whole stream.
