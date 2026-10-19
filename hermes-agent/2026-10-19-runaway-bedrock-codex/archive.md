# Internet Archive snapshots

Requested 2026-10-19 through `https://web.archive.org/save/`, each verified by fetching the returned
snapshot.

| Page | Snapshot |
|---|---|
| https://github.com/NousResearch/hermes-agent/issues/141477 | https://web.archive.org/web/20261019095649/https://github.com/NousResearch/hermes-agent/issues/141477 |
| https://github.com/NousResearch/hermes-agent/issues/141477 (second capture, 18:27 Beijing) | https://web.archive.org/web/20261019102746/https://github.com/NousResearch/hermes-agent/issues/141477?capture_all=on |
| https://github.com/NousResearch/hermes-agent/pull/141479 | https://web.archive.org/web/20261019095741/https://github.com/NousResearch/hermes-agent/pull/141479 |

All return HTTP 200, and each snapshot's title matches the page it is meant to record. Their
purpose is to be an independent copy held by a third party: if the issue or the pull request is
deleted, or the repository goes private, the page still exists somewhere outside anyone's control.

A note on the second issue capture: requesting the plain URL again on the same day returns a `302` to
the morning capture rather than making a new one, so the fresh request carried `?capture_all=on`,
which is why that link has a query string. The morning capture already contains the comment from
`@alt-glitch` (checked by fetching it and matching the text); the second one records the page as it
stands after this folder was updated.
