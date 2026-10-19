# Internet Archive snapshot

Requested 2026-10-19 through `https://web.archive.org/save/`, then verified by fetching the returned
snapshot.

| Page | Snapshot |
|---|---|
| https://github.com/openai/codex/issues/55279 | https://web.archive.org/web/20261019145429/https://github.com/openai/codex/issues/55279 |

Returns HTTP 200, and the snapshot's title matches the page it records:

> Stop a streamed reply that turns into a repetition loop (Codex CLI sends no output cap) · Issue #55279 · openai/codex · GitHub

Its purpose is to be a copy held by a third party: if the issue is deleted, locked, or the repository
goes private, the text as it stood on 2026-10-19 still exists somewhere outside GitHub's control.
