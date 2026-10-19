`"version": ""` currently validates against the `package.json` schema, but npm refuses it:

```console
$ npm pack --dry-run
npm error Invalid package, must have name and version
```

An empty string is also not parsable by node-semver, which is what the field's own description requires. This adds `"minLength": 1` to `version`, matching the existing constraint on `name`.

Scope is deliberately minimal:

- `version` stays optional (private packages and workspace roots legitimately omit it); only the empty string is rejected.
- No semver `pattern` is added, to avoid rejecting any version string node-semver accepts.

Tests: new negative test `src/negative_test/package/version-empty-string.json`. It validates against the current schema and is rejected with this change; the existing positive and negative tests under `package/` are unaffected.

Found by mutation-testing the schema against a real `package.json` with [greencheck](https://github.com/simin-yuan/greencheck); details in its [third-party case study](https://github.com/simin-yuan/greencheck/tree/main/case_study/third_party).
