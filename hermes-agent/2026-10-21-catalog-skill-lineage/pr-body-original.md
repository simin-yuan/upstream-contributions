## What it does

Records where each skill version's content came from, and keeps that record
verifiable afterwards.

For every skill write — a foreground `skill_manage` call, a background-review
edit, or a curator consolidation — the plugin stores the content fingerprint
before and after, the sources present earlier in the same session (user text,
tool results, skills that were read), the features that link a source to the
added lines, and a `seq -> prev_hash -> hash` chain over the entries. Text is
never stored; hashes are keyed per install and the key stays in
`plugin-data/skill-lineage`.

`hermes skill-lineage trace <skill>` / `descendants <url|skill>` / `report` /
`verify` / `head` answer where a version came from, what a source produced,
which versions carry lines of no traceable origin, and whether the chain (or a
recorded head value) still matches. A key reset is recorded in the chain and
shown in `status` rather than happening silently.

## Hermes surfaces used

Four hooks, all read-only observers: `on_session_start`, `pre_llm_call`,
`pre_tool_call`, `post_tool_call`. One CLI command group
(`hermes skill-lineage ...`). No tools, no middleware, no env vars, no core
override. Capabilities in the entry match what the pinned commit registers.

## Disclosure (rule 13)

Reads tool results, user messages and skill files to compute keyed hashes.
Stores hashes (never text) for 7 days per session, plus the lineage chain
(hashes and skill names) under `plugin-data/skill-lineage`. No network calls, no
credentials, never blocks or changes a tool call.

## Tests

`python -m pytest -q` on the pinned commit (Hermes `e06df9c2ea`, Python 3.11.16
and 3.14.7): 46 passed, 1 skipped. `hermes plugins validate .`: passes, no
warnings. The skip is the end-to-end suite, which needs a Hermes importable by
the running interpreter and skips with a printed reason rather than passing
silently.

Manual end-to-end smoke run on Hermes `e06df9c2ea` (2026-10-21): installed into
an isolated `HERMES_HOME`, enabled, and driven by one real session that created
a skill through `skill_manage`; `status`, `trace`, `descendants`, `report`,
`verify` and `head` all returned as documented. Raw output in
[tests/smoke-2026-10-21.txt](https://github.com/simin-yuan/hermes-skill-lineage/blob/main/tests/smoke-2026-10-21.txt).

## Relation to existing work

- #139096 asks for read-back of *installed-skill provenance* (source URL,
  revision, files, install path). That is where a skill came **from at install
  time**; this records which session evidence a skill version's **content was
  derived from**. Complementary, not overlapping.
- `skill-integrity` (catalog) detects that installed skill files **changed**.
  This records **origin**: which source, which lines, whether the record is
  intact. A drift detector and an origin record answer different questions.
- No private core imports: the skill lookup scans `$HERMES_HOME/skills` instead
  of importing `tools.skills_tool` (rule 9).

Owner submission (rule 5): I am the repository owner.