## What does this PR do?

Adds `plugin-catalog/degeneracy-guard.yaml` for https://github.com/simin-yuan/hermes-degeneracy-guard at `ffc28d0ab9ac6b6c0cbb118ed5fb0c06909a3dc1` (version 1.3.0). I own the plugin repository.

Core's `RunawayStreamWatch` truncates runaway text, and by design lets a provider-confirmed tool call through even when its arguments are repetition-dominated (`agent/chat_completion_helpers.py`: "a provider-confirmed call may legitimately write repetitive data"). That case is where a looping model can write its loop into a file or a command. The plugin covers it without overriding the core decision: content alone cannot show whether a repeat is intended, so instead of refusing, `pre_tool_call` returns `{"action": "approve", "rule_key": "degeneracy-guard:<tool>"}` and Hermes' own approval gate asks the user. Denied or unanswered calls are refused (fail-closed, including cron and other unattended runs); "always" stops prompts for that tool; `DEGENERACY_GUARD_ALLOW_TOOLS` exempts tools entirely.

Payloads that parse as data (JSON, tables, matrices, CSV and similar) are not questioned. The README reports measured numbers on two hold-out sets, including the cases it misses and the legitimate payloads that would still prompt.

## Related Issue

None. This is a catalog entry proposal, not a fix; the review thread for it is this PR.

## Type of Change

- [ ] 🐛 Bug fix (non-breaking change that fixes an issue)
- [x] ✨ New feature (non-breaking change that adds functionality)
- [ ] 🔒 Security fix
- [ ] 📝 Documentation update
- [ ] ✅ Tests (adding or improving test coverage)
- [ ] ♻️ Refactor (no behavior change)
- [ ] 🎯 New skill (bundled or hub)

## Changes Made

- `plugin-catalog/degeneracy-guard.yaml` — new community entry, pinned to `ffc28d0ab9ac6b6c0cbb118ed5fb0c06909a3dc1`, version `1.3.0`, `requires_hermes: ">=0.22.2"`.
- `contributors/emails/yleven120@gmail.com` — contributor email mapping, which the contributor check requires.

**Hooks used**
- `pre_tool_call`: request confirmation (the only way it affects a tool call)
- `on_stream_delta`: score streamed text incrementally and record an incident; cannot abort a stream
- `transform_llm_output`: replace a flagged turn's output with a short notice
- `post_llm_call`: clear per-turn state
- `pre_llm_call`: a note once a session passes 500 messages, then every 50

State goes through `plugins.plugin_storage.plugin_data_dir`.

**Disclosure (rule 13)**
- No network calls, shell commands, subprocesses, background threads of its own, telemetry, or credentials.
- Writes JSON incident records and a session-length counter to `<HERMES_HOME>/plugin-data/degeneracy-guard/` (profile-scoped; kept by `hermes plugins update` and `remove`), or `DEGENERACY_GUARD_HOME` when set.
- A stream-side incident record keeps the last 800 characters of the flagged output. Tool-argument records keep the tool name, session id, reason and argument length, not the argument text.
- Changes turns: it can ask the user to confirm a tool call (refused when denied or unanswered), replace a turn's final output, and add context to the next request.

## How to Test

- `hermes plugins validate --install-deps` at the pinned sha: passes; security scan `safe`.
- `hermes plugins doctor degeneracy-guard --ci`: `OK: runtime discovery, manifest parsing, import, and registration passed`, `0 tool(s), 5 hook(s)`.
- `scripts/validate_plugin_catalog.py plugin-catalog/`: OK.
- `requires_hermes: ">=0.22.2"`: the plugin's tests, validate and doctor pass at the v0.22.2 tag and at current main.

## Checklist

### Code

- [x] I've read the [Contributing Guide](https://github.com/NousResearch/hermes-agent/blob/main/CONTRIBUTING.md)
- [x] My commit messages follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat(plugin-catalog):`, `chore:`)
- [x] I searched for [existing PRs](https://github.com/NousResearch/hermes-agent/pulls) to make sure this isn't a duplicate
- [x] My PR contains **only** changes related to this fix/feature (no unrelated commits)
- [ ] I've run `pytest tests/ -q` and all tests pass — not claimed here: this PR is data only, and the suite on this machine has failures that reproduce identically on unmodified main (missing optional dependencies in a local venv). The catalog lane is covered by the commands under "How to Test".
- [x] `python scripts/check` passes (the blocking lint and code-health checks CI runs) — `python scripts/check --commit HEAD --base upstream/main`: 12 checks, ok
- [ ] I've added tests for my changes — this PR adds no code; the entry is checked by `scripts/validate_plugin_catalog.py` and the catalog tests CI runs.
- [x] I've tested on my platform: Windows 11

### Documentation & Housekeeping

- [x] I've updated relevant documentation (README, `docs/`, docstrings) — or N/A (N/A: the entry and the plugin README are the documentation for this plugin)
- [x] I've updated `cli-config.yaml.example` if I added/changed config keys — or N/A
- [x] I've updated `CONTRIBUTING.md` or `AGENTS.md` if I changed architecture or workflows — or N/A
- [x] I've considered cross-platform impact (Windows, macOS) per the [compatibility guide](https://github.com/NousResearch/hermes-agent/blob/main/CONTRIBUTING.md#cross-platform-compatibility) — or N/A (N/A: the plugin is standard library only and is tested on Windows and Linux)
- [x] I've updated tool descriptions/schemas if I changed tool behavior — or N/A (N/A: the plugin registers no tools)

## Screenshots / Logs

```
hermes plugins validate --install-deps  (pinned sha, plugin own venv)
✓ declared hooks — matches registrations
✓ security scan — safe
✓ no core override — no runtime rebinds of Hermes core
Validation passed.

hermes plugins doctor degeneracy-guard --ci
manifest: degeneracy-guard 1.3.0 (standalone)
OK: runtime discovery, manifest parsing, import, and registration passed
registrations: 0 tool(s), 5 hook(s)
```

## Changes since this PR was opened

The pin moved from 1.1.0 to 1.3.0. 1.1.0 refused legitimate repetitive writes (tables, JSON arrays, matrices); 1.3.0 asks the user through the approval gate instead of refusing, does not question payloads that parse as data, scores streams in linear time, uses relative imports, and gives every incident file a unique name. The second commit maps my contributor email.
