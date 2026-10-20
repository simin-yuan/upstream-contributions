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

## Verification status: not confirmed from this host

CDX records `statuscode 200` and `length 61532` — a capture of the live page, not an error page.

What could not be done: fetching the snapshot back. Three attempts from the machine that made the
request (16:05 and 16:09 UTC+08, spaced to respect the rate limit) each returned HTTP 429, the
Wayback Machine's "suspected abusive bot traffic" block, for the snapshot URL as well as for the
availability API. The digest and length above are the archive's own record, not a local check of the
rendered page.

Anyone reading this later can close the gap in one request — open the snapshot URL and look for
`feat(plugin-catalog): add provenance-gate` in the title. If that string is missing, treat the
snapshot as unusable and rely on `snapshot.json` and `pr-141935-current.md` in this directory, which
were fetched directly from the GitHub API at 15:47 and 16:05 (UTC+08).
