# Upstream contributions

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23470460.svg)](https://doi.org/10.5281/zenodo.23470460)

A record of my contributions to upstream open-source projects, kept with the original texts,
timestamps and evidence.

Platform records are not permanent. An issue can be deleted, a repository can be renamed or taken
private, a comment can be edited, and a pull request that was reviewed carefully last month can
become a 404 next year. This repository holds a copy of each contribution — the text as filed, the
text as it stands now, the raw API payloads, the patches, and the output that was produced while the
work was being done — so the record survives independently of the platform that hosts it.

Only contributions to established projects are kept here. Link-list entries are kept only where the
list is the main directory of an established ecosystem (Hermes Agent, JSON Schema); other list
additions are not.

Each entry points at a folder that holds the text as filed, the text as it stands now, the raw API
payloads, the patches, the output produced while the work was done, and an Internet Archive snapshot
of the platform page.

This repository is snapshotted too:

- [The repository page, 2026-10-19 09:23 UTC](https://web.archive.org/web/20261019092302/https://github.com/simin-yuan/upstream-contributions), as it stood before the DOI was added
- [The same page with the DOI in place](https://web.archive.org/web/20261019095049/https://github.com/simin-yuan/upstream-contributions?capture_all=on)
- [The release `v2026.10.19`](https://web.archive.org/web/20261019095540/https://github.com/simin-yuan/upstream-contributions/releases/tag/v2026.10.19)
- [The Zenodo record](https://web.archive.org/web/20261019095414/https://zenodo.org/records/23470461)

## Index

| Date | Project | Type | Title | Status |
|---|---|---|---|---|
| 2026-10-22 | NousResearch/hermes-agent | PR | [Pass `platform` to the tool-path hooks](hermes-agent/2026-10-22-tool-hooks-platform/) — [#142974](https://github.com/NousResearch/hermes-agent/pull/142974) | open, `P3` `type/feature` `comp/agent` `comp/cli` `comp/plugins` `area/i18n`; workflows awaiting approval, no review |
| 2026-10-22 | NousResearch/hermes-agent | issue | [`platform` is missing from `pre_tool_call` / `post_tool_call` / `transform_tool_result`](hermes-agent/2026-10-22-tool-hooks-platform/) — [#142973](https://github.com/NousResearch/hermes-agent/issues/142973) | open, `P2` `comp/agent` `type/bug`; maintainer triaged, unassigned |
| 2026-10-21 | NousResearch/hermes-agent | PR | [Add `skill-lineage` to the plugin catalog](hermes-agent/2026-10-21-catalog-skill-lineage/) — [#142381](https://github.com/NousResearch/hermes-agent/pull/142381) | open, `P3` `comp/plugins` `type/feature`; a review is required |
| 2026-10-21 | NousResearch/hermes-agent | PR | [`api_max_retries` counts attempts, not retries](hermes-agent/2026-10-21-api-max-retries-attempts/) — [#142179](https://github.com/NousResearch/hermes-agent/pull/142179) | open, no review |
| 2026-10-20 | NousResearch/hermes-agent | PR | [Add `provenance-gate` to the plugin catalog](hermes-agent/2026-10-20-plugin-catalog-provenance-gate/) — [#141935](https://github.com/NousResearch/hermes-agent/pull/141935) | merged |
| 2026-10-19 | NousResearch/hermes-agent | issue | [`hermes plugins validate` rejects the legacy `hooks:` manifest key that the loader accepts](hermes-agent/2026-10-19-validate-legacy-hooks/) — [#141339](https://github.com/NousResearch/hermes-agent/issues/141339) | open |
| 2026-10-19 | NousResearch/hermes-agent | PR | [Make the plugin validator accept the legacy `hooks:` key](hermes-agent/2026-10-19-validate-legacy-hooks/) — [#141340](https://github.com/NousResearch/hermes-agent/pull/141340) | open, rebased onto `main`; workflows awaiting approval |
| 2026-10-19 | NousResearch/hermes-agent | PR | [Add `hermes-degeneracy-guard` to the plugin catalog](hermes-agent/2026-10-19-plugin-catalog-degeneracy-guard/) — [#141341](https://github.com/NousResearch/hermes-agent/pull/141341) | merged |
| 2026-10-19 | NousResearch/hermes-agent | issue | [Bedrock and Codex streams don't cut a repetition loop mid-stream (#130551 covers chat_completions and Anthropic only)](hermes-agent/2026-10-19-runaway-bedrock-codex/) — [#141477](https://github.com/NousResearch/hermes-agent/issues/141477) | open |
| 2026-10-19 | NousResearch/hermes-agent | PR | [cut runaway repetition loops mid-stream on Bedrock and Codex](hermes-agent/2026-10-19-runaway-bedrock-codex/) — [#141479](https://github.com/NousResearch/hermes-agent/pull/141479) | open, awaiting CI approval |
| 2026-10-19 | 0xNyk/awesome-hermes-agent | issue | [Listing request for `hermes-degeneracy-guard`](awesome-hermes-agent/2026-10-19-listing/) — [#524](https://github.com/0xNyk/awesome-hermes-agent/issues/524) | open |
| 2026-10-19 | openai/codex | issue | [Stop a streamed reply that turns into a repetition loop (Codex CLI sends no output cap)](openai-codex/2026-10-19-issue-55279/) — [#55279](https://github.com/openai/codex/issues/55279) | open, awaiting triage |
| 2026-10-19 | langchain-ai/langchain | comment | [Independent measurement across six agent frameworks, and two notes on the #41286 patch](langchain/2026-10-19-issue-41285-comment/) — [#41285](https://github.com/langchain-ai/langchain/issues/41285) | open, no assignee |
| 2026-10-19 | langchain-ai/langchainjs | comment | [Same behaviour on the JavaScript side; #11494 only covers the output-limit case](langchainjs/2026-10-19-issue-11469-comment/) — [#11469](https://github.com/langchain-ai/langchainjs/issues/11469) | open |
| 2026-10-08 | sourcemeta/awesome-jsonschema | PR | [Add greencheck to Development Tools](awesome-jsonschema/2026-10-08-pr-67/) — [#67](https://github.com/sourcemeta/awesome-jsonschema/pull/67) | merged |
| 2026-09-28 | SchemaStore/schemastore | PR | [`package.json`: reject an empty `version` string](schemastore/2026-09-28-pr-6421/) — [#6421](https://github.com/SchemaStore/schemastore/pull/6421) | merged |

## What each folder contains

| File | Contents |
|---|---|
| `record.md` | What was filed, why it matters, the timeline of who did what, and what changed in the text after submission |
| `*-current.md` | The body as it stands on the platform, fetched through the API |
| `*-original.md` | The body as first posted, where it was later edited and a copy of the first version was kept |
| `snapshot.json` | Raw API payloads: the item, its comments, its timeline (labels, comments, cross-references, merges) and its changed files |
| `patch/` | The substantive commits, exported with `git format-patch` or from the commit API. Bookkeeping commits (email mapping, formatting) are excluded |
| `evidence.md` | Reproduction steps and the real output produced while the work was done — including the runs that failed, and a note wherever something could not be verified |
| `archive.md` | An independent Internet Archive snapshot of the platform page |

Two rules were applied to the evidence and are worth stating: nothing is written into `evidence.md`
that was not actually produced at the time, and whenever a claim rests on the project's own CI rather
than on a local run, it says so. A record that only collects successes is not evidence of anything.

## License

The prose in this repository — `README.md`, the `record.md`, `evidence.md` and `archive.md` files,
and the archived issue and pull request texts written by their respective authors — is licensed
under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), by Simin (Simon) Yuan.

The patches in each `patch/` directory are the work of their authors as recorded in the commit
headers, and remain under the licence of the project they were submitted to:

| Project | Licence |
|---|---|
| NousResearch/hermes-agent | MIT |
| SchemaStore/schemastore | Apache-2.0 |
| sourcemeta/awesome-jsonschema | CC0-1.0 |
| 0xNyk/awesome-hermes-agent | no licence declared upstream; the quoted listing text is reproduced for the record |
| langchain-ai/langchain | MIT — no patch is reproduced here, only quoted file paths and line numbers |
| langchain-ai/langchainjs | MIT — same |

Quoted comments from maintainers are reproduced verbatim with attribution, which is what the record
is for.

## Archiving and citation

Zenodo archives each tagged release of this repository and assigns a DOI. All versions share the
concept DOI [10.5281/zenodo.23470460](https://doi.org/10.5281/zenodo.23470460); the release
`v2026.10.19` is [10.5281/zenodo.23470461](https://doi.org/10.5281/zenodo.23470461).

```bibtex
@misc{yuan_upstream_contributions,
  author    = {Yuan, Simin (Simon)},
  title     = {Upstream contributions: a record of issues and pull requests to upstream open-source projects},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.23470460},
  url       = {https://doi.org/10.5281/zenodo.23470460}
}
```

---

Simin (Simon) Yuan · ORCID [0009-0002-6843-3390](https://orcid.org/0009-0002-6843-3390) · https://github.com/simin-yuan
