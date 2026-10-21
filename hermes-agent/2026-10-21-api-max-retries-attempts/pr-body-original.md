## What does this PR do?

Three files carried the same claim about `agent.api_max_retries`: that it is a count of *retries*, on top of the first call — which makes the default of `3` read as four requests. The value is the total number of attempts, with the first call counted as one, so the default is three.

Docs and user-facing copy only: no behaviour change.

## Related Issue

None — documentation-only. #126096 reasons about the same loop (quoted below) but reports a separate design question about the Z.AI backoff ceiling, not this wording.

## Type of Change

- [x] 📝 Documentation update

## Changes Made

- `website/docs/user-guide/configuration.md` — the `api_max_retries` paragraph, and the sample `agent.api_max_retries` comment (`Retries per provider` → `Attempts per provider`).
- `website/i18n/zh-Hans/docusaurus-plugin-content-docs/current/user-guide/configuration.md` — both, in the translation.
- `locales/*.yaml` (all 17 bundled catalogs) — the `tips.t240` startup tip: — the `tips.t240` startup tip in all 17 language catalogs: "how many times the agent retries a failed API call" → "how many attempts the agent makes at a failed API call". Each catalog keeps its own sentence structure; only the count noun changed, so a native-speaker pass on the non-English strings is welcome.

## Back-translation check (the non-English strings)

All 17 reworded tips were translated back to English by an independent model (qwen-max) that saw only the target-language string and not the original. Sixteen read back as "<n> attempts ... before the error is shown — lower it for faster fallback" and match the English sentence.

One was fixed in `88515e120f`: the Korean string used 대체 ("replacement") for fallback, which reads back without an object; it now says 폴백, the term Korean developer docs use.

## Why the value counts attempts

1. `agent/agent_init.py:1444` — `_api_retries = max(int(_agent_section.get("api_max_retries", 3)), 1)`
2. `agent/conversation_loop.py:1672` — `s.api_start_time, s.retry_count, s.max_retries = time.time(), 0, agent._api_max_retries`
3. `agent/conversation_loop.py:1510` — `while s.retry_count < s.max_retries:`
4. `agent/turn_api_error.py:139` (also `agent/turn_response_check.py:283` and `agent/turn_iteration_prep.py:501`) — `retry_count += 1` on the failure paths

`retry_count` starts at `0`, so the loop body runs exactly `max_retries` times and the first call consumes one of them.

Two independent readings of the same code agree:

- `agent/retry_utils.py:173` docstring — "the default ``api_max_retries`` of 3 equals ``short_attempts``".
- #126096 — "Hermes does 3 short retries and aborts" and "The retry loop gives up when `retry_count >= max_retries` before computing the attempt's backoff".

Measured in an isolated fake-provider harness (`agent-double-exec-bench`, `results/hermes/e5-isolation`), where the cycle count is pinned so attempts per cycle are directly countable:

| config | streaming attempts | cycles | attempts per cycle |
|---|---|---|---|
| `api_max_retries: 3`, `auto_recovery_cycles: 0` | 3 | 1 | **3** |
| `api_max_retries: 0` (clamped to 1), `auto_recovery_cycles: 0` | 1 | 1 | **1** |
| defaults | 18 | 6 | **3** |

## Overlap with other open PRs

#104465 corrects two *other* stated defaults — the `TERMINAL_DOCKER_IMAGE` fallback in `hermes_cli/status.py` (line 108), and `HERMES_STREAM_RETRIES` documented as `3` where the code defaults to `2`. It does not mention `api_max_retries` and does not touch `configuration.md`, so this PR is not a duplicate of it and shares no changed line with it.

Its `HERMES_STREAM_RETRIES` wording is the same confusion in another key — retries versus attempts — and is phrased, in that PR, as "default 2, so three attempts in total". This PR follows that phrasing so the two read the same way.

Same error, left untouched on purpose: the `HERMES_STREAM_RETRIES` startup tip, which now lives at `locales/*.yaml` (`tips.t367`) rather than in `hermes_cli/tips.py`, and is covered by #104465. Also untouched: `agent/retry_utils.py:173`, whose docstring already says "the default ``api_max_retries`` of 3 equals ``short_attempts``" — correct as written.

## How to Test

```
1. grep -n "api_max_retries", agent/agent_init.py        # default 3, clamped to >= 1
2. grep -n "retry_count < s.max_retries", agent/conversation_loop.py   # loop header
3. grep -n "s.retry_count, s.max_retries", agent/conversation_loop.py  # reset to 0 per API call
4. grep -n "retry_count += 1", agent/turn_api_error.py  # counter advances on failure
5. grep -n "short_attempts", agent/retry_utils.py      # docstring stating the equality
```

Ran on the branch:

```
$ python scripts/check
=== health (ok) ===
scripts/check: 12 checks, ok

$ python -m pytest tests/agent/test_i18n.py tests/hermes_cli/test_tips_catalog.py \
      tests/agent/test_i18n_tui_packs.py tests/hermes_cli/test_plugin_validate_locales.py -q
80 passed in 23.20s
```

The i18n suite is included because this PR edits all 17 catalogs and key parity is enforced there.

## Checklist

### Code

- [x] I've read the [Contributing Guide](https://github.com/NousResearch/hermes-agent/blob/main/CONTRIBUTING.md)
- [x] My commit messages follow [Conventional Commits](https://www.conventionalcommits.org/)
- [x] I searched for [existing PRs](https://github.com/NousResearch/hermes-agent/pulls) — the closest, #104465, corrects a different stated default (details above)
- [x] My PR contains **only** changes related to this fix
- [x] `python scripts/check` passes (12 checks, ok)
- [x] `pytest` passes for every test file covering what this PR touches (80 passed: i18n catalogs + tips catalog)
- N/A no unit test added — wording only, the loop semantics it describes are already asserted by existing retry tests
- [x] I've tested on my platform: Windows 11 (docs and catalog edits only)

### Documentation & Housekeeping

- [x] I've updated relevant documentation (`configuration.md`, its zh-Hans mirror, and the 17 catalogs)
- N/A `cli-config.yaml.example` — the commented sample carries no count claim
- N/A `CONTRIBUTING.md` / `AGENTS.md` — no architecture or workflow change
- N/A cross-platform impact — prose only
- N/A tool descriptions/schemas — no tool behaviour changed

## Screenshots / Logs

```
$ python scripts/check
=== health (ok) ===
code health: no measured files changed
scripts/check: 12 checks, ok

$ python -m pytest tests/agent/test_i18n.py tests/hermes_cli/test_tips_catalog.py tests/agent/test_i18n_tui_packs.py tests/hermes_cli/test_plugin_validate_locales.py -q
80 passed in 23.20s
```

Drafted by an AI agent under my direction; I reviewed the diff and ran the checks above.
