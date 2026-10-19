#130551 added `RunawayStreamWatch`, which stops a stream as soon as it turns into a repetition loop. It is wired into the chat_completions stream (`agent/chat_completion_helpers.py:3054`) and the Anthropic Messages stream (`:3396`). The Bedrock Converse stream (`agent/bedrock_adapter.py`, `stream_converse_with_callbacks`) and the Codex Responses stream (`agent/codex_runtime.py`, `_consume_codex_event_stream`) don't use it.

The providers docs say a looping reply is still stopped, so an uncapped endpoint can't keep a looping model running (`website/docs/integrations/providers.md`, around line 1300). The next paragraph says Converse omits `maxTokens` by default. So Bedrock is the uncapped path, and it's one of the two without the guard. `_BedrockStream`'s docstring also says it aims for the same UX as the Anthropic and chat_completions streams.

**Repro (at `2e83e2a`, no network):** feed ~141K characters of one repeated line into each consumer.

- `RunawayStreamWatch` trips at 16,074 characters.
- `stream_converse_with_callbacks` reads all 141,000.
- `_consume_codex_event_stream` reads all 141,000.

The post-stream check still rejects the turn, so the answer is right in the end. The cost is the whole output budget first, and on Bedrock that budget is the model's maximum.

I found this by reading the code and confirmed it with the repro above. #127242, #125351 and #127780 touch the same guard but not these two paths; #117756 and #138770 tune the reasoning-channel cut on the chat_completions path, and neither touches these two consumers. #80701 adds a total-lifetime cap, which is time-based and complements this.

A fix with tests is in #(PR). For context, I also measured how other agent frameworks handle the same loop: https://github.com/simin-yuan/agent-runaway-bench. Hermes is the only one of them with a mid-stream guard at all.
