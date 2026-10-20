# Evidence — `provenance-gate` catalog entry

Produced on 2026-10-20, Windows 11, git 2.x / GitHub CLI, against the delivered bundle and the
public repositories as they stand. Every output below is copied from the run that produced it.

## 1. The rebuilt history, verified from the bundle

```
$ git bundle verify ../hermes-provenance-gate-8813469.bundle
../hermes-provenance-gate-8813469.bundle is okay
The bundle contains these 2 refs:
8813469d45c82fb0f5af6724c3761169ca7c3e86 HEAD
8813469d45c82fb0f5af6724c3761169ca7c3e86 refs/heads/main
The bundle records a complete history.

$ git rev-parse HEAD
8813469d45c82fb0f5af6724c3761169ca7c3e86

$ git log --format='%an <%ae> | %cn <%ce>' | sort | uniq -c
     38 Simin (Simon) Yuan <yleven120@gmail.com> | Simin (Simon) Yuan <yleven120@gmail.com>
```

38 commits, one author and one committer identity, no `Co-Authored-By`, no generated-by trailer.

## 2. Redline scan over every commit, not just the tip

The scan used a fixed list of personal and environment identifiers — the list itself is kept outside
this repository, deliberately, so that scanning for a name does not publish it.

```
$ git grep -n -i -E "<redline list>" $(git rev-list --all) -- . \
    | grep -v "Simin (Simon) Yuan" | wc -l
0
```

The only exclusion is the author's own name as signed in `NOTICE` and `plugin.yaml`.

A second, wider pass (the author handle, the email domain, cloud provider names, a city name) over
every commit returns only the repository's own URL in `README.md` — `github.com/simin-yuan/hermes-provenance-gate`,
the public address of the repository itself.

## 3. The public repository

```
$ gh repo view simin-yuan/hermes-provenance-gate --json name,visibility,isEmpty,defaultBranchRef
{"defaultBranchRef":{"name":"main"},"isEmpty":false,"name":"hermes-provenance-gate","visibility":"PUBLIC"}

$ gh api repos/simin-yuan/hermes-provenance-gate/commits/main --jq .sha
8813469d45c82fb0f5af6724c3761169ca7c3e86

$ git ls-remote --heads https://github.com/simin-yuan/hermes-provenance-gate.git
8813469d45c82fb0f5af6724c3761169ca7c3e86	refs/heads/main

$ git ls-remote --tags https://github.com/simin-yuan/hermes-provenance-gate.git | wc -l
0

$ gh run list -R simin-yuan/hermes-provenance-gate -L 3 --json databaseId,status,conclusion,headSha
[{"conclusion":"success","databaseId":39387728408,"headSha":"8813469d45c82fb0f5af6724c3761169ca7c3e86","status":"completed","workflowName":"ci"}]
```

One branch, no tags, CI green at the tip. Unauthenticated requests: the repository page and the raw
README both return HTTP 200, so the README is public without a token.

The private archive:

```
$ gh repo view simin-yuan/hermes-provenance-gate-private --json name,visibility,defaultBranchRef
{"defaultBranchRef":{"name":"main"},"name":"hermes-provenance-gate-private","visibility":"PRIVATE"}
```

## 4. The entry passes the repository's own validator

```
$ python scripts/validate_plugin_catalog.py plugin-catalog/provenance-gate.yaml
OK: 1 file(s) valid
```

`plugin-catalog/provenance-gate.yaml` in the PR is byte-identical to the file in the delivery
package, 1501 bytes, sha256 `110dee7c97be4eadc219ae245a89dd210071d62a59c19a525f164fd9bc809faa`.

## 5. The PR body is what was submitted

```
remote_chars= 3310 local_chars= 3310
exact_equal= True
remote_sha= e42edfde45d0296d3365a9fa14ce674636ba9f62fe168331b60e38d7026c6bb6
local_sha = e42edfde45d0296d3365a9fa14ce674636ba9f62fe168331b60e38d7026c6bb6
```

Compared through the API's JSON payload, not a `--jq` pipe (which appends its own newline).

## 6. The commit in the PR carries one identity

```
$ gh api repos/simin-yuan/hermes-agent/commits/e1fa5ad038 --jq '...'
e1fa5ad03832cfe21a6ea41cfa0a1d635e60471b | Simin (Simon) Yuan <yleven120@gmail.com> | Simin (Simon) Yuan <yleven120@gmail.com> | feat(plugin-catalog): add provenance-gate
```

## 7. Homepage README, read back from `raw`

```
$ git show HEAD:README.md | sha256sum
a825a5519877611155ec0109436f7790cdc4633072e1df1b3c7f583f4b1d8cca
$ sha256sum <raw.githubusercontent.com/.../main/README.md>
a825a5519877611155ec0109436f7790cdc4633072e1df1b3c7f583f4b1d8cca
```

The committed blob and the served file are byte-identical. Working-tree CRLF is the local checkout
conversion; the blob is LF, as `.gitattributes` requires.
