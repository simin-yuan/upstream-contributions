# awesome-hermes-agent — listing request for `hermes-degeneracy-guard`

| | |
|---|---|
| Issue | [#524](https://github.com/0xNyk/awesome-hermes-agent/issues/524) |
| Submitted | 2026-10-19 16:47 (UTC+08:00) — `08:47:38Z` |
| State | open |
| Labels | none |
| Category requested | Skills & Plugins → Plugins |

## What it is, and why it mattered

A listing request for the plugin described in
`../2026-10-19-plugin-catalog-degeneracy-guard/`. It was deliberately filed after the plugin
reached 1.3.0 and after the Hermes catalog PR was rewritten around the approval gate: asking a list
to carry a version whose detection is known to refuse legitimate writes would have pushed the defect
onto more people than it helped.

The body states what the plugin does, what it does not do (no network, shell, threads, telemetry or
credentials), and why it exists at all — the core truncates runaway text but lets provider-confirmed
tool calls through by design, which is correct for the core and leaves the judgment of "is this a
loop or a payload" to the person who can actually answer it.

## Timeline

| Time (UTC) | Event |
|---|---|
| 2026-10-19 08:47:38 | Issue opened. |
| 2026-10-19 08:47:39 | Cross-referenced by Hermes PR #141341. |
| — | No labels, no comments as of 2026-10-19 16:58. |

## Contents

- `issue-524-current.md` — the body as filed (unmodified since)
- `snapshot.json` — raw API payloads as returned on 2026-10-19 16:58 +08:00
- `archive.md` — Internet Archive snapshot

No patch: a listing request carries no code. The plugin's code lives in
[simin-yuan/hermes-degeneracy-guard](https://github.com/simin-yuan/hermes-degeneracy-guard).
