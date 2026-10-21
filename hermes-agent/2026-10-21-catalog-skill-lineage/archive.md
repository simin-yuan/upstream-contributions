# Internet Archive snapshot

Requested 2026-10-21 through `https://web.archive.org/save/`; the save endpoint answered `200` and
redirected to the capture below. The CDX index lists it:

```
$ http://web.archive.org/cdx/search/cdx?url=github.com/NousResearch/hermes-agent/pull/142381&output=json
["urlkey","timestamp","original","mimetype","statuscode","digest","length"],
["com,github)/nousresearch/hermes-agent/pull/142381","20261021093012",
 "https://github.com/NousResearch/hermes-agent/pull/142381","text/html","200",
 "TBT5SZ5462YM7IBTMPYCVO2IL7LNBKWS","61729"]
```

| Page | Snapshot |
|---|---|
| https://github.com/NousResearch/hermes-agent/pull/142381 | https://web.archive.org/web/20261021093012/https://github.com/NousResearch/hermes-agent/pull/142381 |

## Verified by reading it back

The capture was fetched again and the page it returns is this pull request: 351,442 bytes of
HTML, and the title carries the entry's subject. The capture holds the state of 2026-10-21 09:30 UTC —
before any review comment, so it is a record of the submission, not of a later discussion.
