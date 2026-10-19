# SchemaStore — `package.json`: reject an empty `version` string

| | |
|---|---|
| Pull request | [#6421](https://github.com/SchemaStore/schemastore/pull/6421) |
| Submitted | 2026-09-27 21:23 (UTC+08:00) — `13:23:29Z` |
| Merged | 2026-09-28 08:14 (UTC+08:00) — `00:14:05Z` by @hyperupcall |
| Merge commit | `a83ce6f194e77d36174527f18820f170c2e58033` |
| Branch | `package-version-min-length` → `master` |
| Change | `+6 / −1` across 2 files |

## What it is, and why it mattered

`"version": ""` validated against the `package.json` schema while npm refuses it outright, and
node-semver cannot parse an empty string either — which the field's own description requires. The
change adds `"minLength": 1` to `version`, matching the constraint the schema already put on
`name`, plus a negative test. Scope was kept minimal on purpose: `version` stays optional, because
private packages and workspace roots legitimately omit it; only the empty string is rejected. No
semver `pattern` was added either, so no string that node-semver accepts can be rejected by this
schema.

The point of the report was narrow: a schema that accepts a value the toolchain always rejects
turns a hard error into a silent one for anyone generating manifests.

## Timeline

| Time (UTC) | Event |
|---|---|
| 2026-09-27 13:23:29 | PR opened, two commits. |
| 2026-09-27 13:23:43 | Bot comment: the schema area is owned by @madskristensen and @hyperupcall, and a "LGTM" from an owner is what merges it. |
| 2026-09-28 00:14:01 | @hyperupcall: |

> Thanks, this is great.
>
> — `hyperupcall`, 2026-09-28 00:14:01Z

| Time (UTC) | Event |
|---|---|
| 2026-09-28 00:14:05 | Merged by @hyperupcall as `a83ce6f`. |

## Contents

- `pr-6421-current.md` — the body as filed (unmodified)
- `snapshot.json` — raw API payloads as returned on 2026-10-19 16:58 +08:00
- `patch/0001-reject-empty-version-string.patch`,
  `patch/0002-negative-test.patch` — both commits, exported from GitHub's commit API
- `evidence.md` — what the change does and what the project's own CI proved
- `archive.md` — Internet Archive snapshot

GitHub does not serve earlier revisions of a pull request body, and this one was not edited, so the
current body is also the original.
