# Evidence — SchemaStore #6421

## What the change is

`src/schemas/json/package.json` gains `"minLength": 1` on the `version` property; the negative-test
fixture `src/negative_test/package/version-empty` gains a case. Two commits, `+6 / −1` across two
files.

## What the project's own checks proved

Check-runs on the merge commit `a83ce6f194e77d36174527f18820f170c2e58033`:

```console
check-runs total=5: codeowners-merge=success, deploy=success, build=success,
                    Run pre-commit checks=success, validate=success
combined status: success
```

Workflow runs at the same commit:

```console
Maintenance: Codeowners Detection and Mention=success
Validate JSON Schemas=success
Make Commit=cancelled
Maintenance: precommit checks=success
Deploy to GitHub pages=success
```

`validate` is the job that builds every schema and runs the negative-test corpus, which is what
makes the added fixture meaningful: the empty-version case is rejected by the schema after the
change and would have passed before it.

## What is not archived here

No local run of the SchemaStore tooling is kept. The evidence for this PR is the project's CI at
the merge commit, which is cited above; inventing a local transcript afterwards would be worse than
saying so.
