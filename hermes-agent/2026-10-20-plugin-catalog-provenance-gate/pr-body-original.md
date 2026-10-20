Adds `provenance-gate` to the plugin catalog: one file, `plugin-catalog/provenance-gate.yaml`, pinned to `8813469d45c82fb0f5af6724c3761169ca7c3e86` (v0.1.0). I am the repository owner.

## What it does

An agent sometimes writes "the user decided X" into its memory or a skill when the user never decided X. This plugin checks every `memory` and `skill_manage` write for sentences that attribute a decision to the user (English and Chinese patterns, shipped as data with tests). Each such sentence needs a quote that appears verbatim in text the user typed in this session, shares key words with the decision and does not drop a negation. A patch is checked by the text it produces (the operations of a call are replayed over the current files), so a claim cannot be assembled from existing text.

- **Observe mode (default):** nothing is refused; flagged writes are logged as `would_block` and `hermes provenance-gate report` lists them.
- **Block mode (opt-in, `PROVENANCE_GATE_MODE=block`):** the write is refused with a message telling the model how to fix it. No approval prompt is used, so it behaves the same in cron, gateway and unattended runs.

Measured on a blind set of 133 lines written and labelled by an independent reviewer before reading the code: precision 1.000, recall 0.694 (the misses are published in the README).

## Hermes surfaces

- `pre_llm_call`: records the text the user typed. Blocks Hermes adds to the user message (reply pointers, attachment notes, skill bodies, quoted-reply blocks, system notes) are removed first, and turns Hermes writes itself (async delegation results, process notices, goal and heartbeat prompts, using Hermes' own list in `agent/context_compressor.py`) are not recorded. Returns nothing.
- `pre_tool_call`: checks `memory` and `skill_manage`; returns `{"action": "block", ...}` only in block mode.
- `ctx.register_cli_command("provenance-gate", ...)`: `report`, `log`, `check`, `purge`.
- Public imports only (`hermes_constants`, `plugins.plugin_storage`, `tools.skills_tool.skill_search_dirs` / `locate_skill` / `skill_lookup_path_error`); `hermes plugins validate` reports no warnings.

## Disclosure (rule 13)

- Stores the text the user types, with its `sender_id` and the compression `parent_session_id`, in `<HERMES_HOME>/plugin-data/provenance-gate/said.jsonl`, and an evidence log of checked claims (claim, quote, verdict) in `evidence.jsonl`. Files are created 0600 on POSIX. 30-day default retention; `hermes provenance-gate purge` deletes on demand.
- Reads `MEMORY.md`, `USER.md` and skill files under HERMES_HOME to reproduce what a patch produces. Writes nothing outside its plugin-data folder.
- No network, no credentials, no subprocesses, no background threads, no self-update.

## Verification

- `hermes plugins validate` passes with `warnings: []`; `hermes plugins doctor provenance-gate --ci` passes (0 tools, 2 hooks).
- 760 tests, including end-to-end tests through a real `AIAgent` against a fake model server (foreground, background review fork, curator fork, subagents, patches, batches), pass on Hermes `b4170e73`, on `main` and on the v0.22.2 release. CI runs the unit tests on Python 3.11–3.14 on Ubuntu and Windows, and a weekly job against Hermes `main`.
- Three independent audits; every bypass they reported is a test.
