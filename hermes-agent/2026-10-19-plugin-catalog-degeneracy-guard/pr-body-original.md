Adds `plugin-catalog/degeneracy-guard.yaml` for https://github.com/simin-yuan/hermes-degeneracy-guard at `bb66961ff402fde6b323107bc3e96e449ffbff09` (version 1.1.0). I own the plugin repository.

**What it does**

It detects repetition-loop ("degeneracy") output and keeps it from acting.

- `pre_tool_call` blocks a tool call whose string arguments match the loop criteria, returning `{"action": "block"}` before the tool runs.
- `on_stream_delta` scores streamed text and records an incident. It cannot abort a stream.
- `transform_llm_output` replaces a flagged turn's output with a short notice.
- `post_llm_call` clears per-turn state.
- `pre_llm_call` adds a note asking for a fresh session once a session passes 500 messages, and again every 50 messages after that.

It uses only these five hooks. It has no tools, middleware, required env vars or Python dependencies (standard library only), and it does not modify Hermes core.

**Hermes surfaces used**

Hooks (`on_stream_delta`, `transform_llm_output`, `post_llm_call`, `pre_tool_call`, `pre_llm_call`) and `plugins.plugin_storage.plugin_data_dir` for its state directory.

**Disclosure (rule 13)**

- No network calls.
- No shell commands or subprocesses.
- No background processes or threads of its own.
- No telemetry or usage reporting.
- No credentials read, stored or requested.
- Writes JSON incident records and a small session-length counter to `<HERMES_HOME>/plugin-data/degeneracy-guard/`. This directory is profile-scoped and is kept by `hermes plugins update` and `remove`. It writes to `DEGENERACY_GUARD_HOME` instead when that is set.
- A stream-side incident record keeps the last 800 characters of the flagged output. Tool-argument incident records keep the tool name, session id, reason and argument length, not the argument text.
- Changes turns: it can block a tool call, replace a turn's final output, and add context to the next request.

**Validation**

- `hermes plugins validate --install-deps` passes at the pinned sha. Security scan: safe, no caution findings.
- `hermes plugins doctor degeneracy-guard --ci`: OK, 0 tool(s), 5 hook(s).
- `scripts/validate_plugin_catalog.py plugin-catalog/` passes.
- `requires_hermes: ">=0.22.2"`: the plugin's tests, `hermes plugins validate` and `hermes plugins doctor` pass against the v0.22.2 release and against current main. `plugin_data_dir` is present at v0.22.2.

The second commit maps my contributor email (`contributors/emails/`), which the contributor check requires.
