# Internet Archive snapshot

Requested 2026-10-20 through `https://web.archive.org/save/`. The save endpoint answered with the
Wayback Machine's bot-traffic block page, but the capture itself was recorded — the CDX index lists
it:

```
$ http://web.archive.org/cdx/search/cdx?url=github.com/NousResearch/hermes-agent/pull/141935&output=json
["urlkey","timestamp","original","mimetype","statuscode","digest","length"],
["com,github)/nousresearch/hermes-agent/pull/141935","20261020080057",
 "https://github.com/NousResearch/hermes-agent/pull/141935","text/html","200",
 "OTZU3KSTEBTHCWREH57OPQNHVALD2J7N","61532"]
```

| Page | Snapshot |
|---|---|
| https://github.com/NousResearch/hermes-agent/pull/141935 | https://web.archive.org/web/20261020080057/https://github.com/NousResearch/hermes-agent/pull/141935 |

## Verified by reading it back

The first three read-back attempts (16:05, 16:09 and 16:12 UTC+08) returned HTTP 429 — the same
bot-traffic block, applied to the snapshot URL and to the availability API alike, not to the page
itself. The fourth attempt, at 16:22, returned the captured page:

```
$ curl -s https://web.archive.org/web/20261020080057/https://github.com/NousResearch/hermes-agent/pull/141935
http=200 size=342150
<title>feat(plugin-catalog): add provenance-gate by simin-yuan · Pull Request #141935 · NousResearch/hermes-agent · GitHub</title>
```

The PR title appears six times in the captured HTML. The snapshot is the reviewable state at filing:
open, two files, one commit, checks held at the first-time-contributor approval gate.

`snapshot.json`, `pr-files-141935.json` and `entry-contents-api.json` in this directory were fetched
directly from the GitHub API rather than from the archive, so they do not depend on this snapshot
surviving.
