# Next candidates

Things learned from this record that are worth doing next. Each entry carries its origin, its
cheapest first step and what it is waiting on; the first one now has a built, unpublished branch,
noted under its own heading.

## 1. Three stages, three defaults: record → judge → settle

**Origin.** Raised in discussion under the r/hermesagent post for
[`platform` in the tool-path hooks](hermes-agent/2026-10-22-tool-hooks-platform/), continuing a
thread from r/LangChain. u/jylusdev supplied the adversarial cases (a relayed decision, an unmet
condition, a hypothetical, a negated quote); u/Feeling_Sun_6436 supplied the settlement half —
record the claim and its evidence, let a separate component own status changes, append the evidence
for a change while preserving the original conditional claim, and leave it pending when nobody can
verify the condition.

**The model.** The three plugins look inconsistent because their defaults differ: `skill-lineage`
records and refuses to edit history, `provenance-gate` judges once at write time, and the settling
step has not been built. Read as one chain — record, judge, settle — the differing defaults are a
consequence of the stage, not an accident.

**Cheapest first step.** Append, do not overwrite. Today `provenance-gate` stores
`condition: unresolved` on the evidence row and nothing can move it. The next version pins the code
revision at write time (which is a hard fact the gate can capture itself) and leaves the row pending
until someone names the check (which it cannot: agents write "only if the benchmark passes", not
which benchmark).

**Pinning the revision: empty beats wrong.** A revision is only capturable when the session actually
operated on that repository. Most claims have no repository at all ("the user decided on Postgres",
"Alice answers next week"), and an agent's working directory is frequently some other repository.
A commit captured from an unrelated cwd is worse than an empty field, because it will be read as a
hard fact. So the field is optional: fill it only when the repository was touched in this session,
otherwise leave it empty.

**Constraint that decides the design.** Whatever appends a resolution must not be the agent that
wrote the claim, or the condition is graded by its author. In practice that means the user or CI.

**Waiting on.** Nothing but the reply. The branch below is built; publishing it — a release, an
install note, a word in the thread — still waits, so the wording can credit u/Feeling_Sun_6436.

**State (2026-10-23).** Branch `feat/settle-a-stored-condition` in `hermes-provenance-gate`, commit
`c4007db`, local only: no PR, no release, nothing said publicly. Built during the wait so that the
answer to a reply can be "it is done" instead of "we will". What exists: `settlement.py`
(append-only `settlements.jsonl`; a claim id derived from session, target, claim and quote, so it
survives a prune; only `user` or `ci` may settle; the session that wrote the claim cannot settle its
own; a revision must be a real git sha or be absent), `repos.py` (pin `HEAD` only when the session
touched exactly one repository — none, two, or an unreadable `HEAD` give an empty string on purpose),
`pending` and `settle` commands, and 18 tests. The verdict is untouched: the six relayed/conditional
holes still `xfail` deliberately, because blocking a relayed claim is a false-block risk rather than
a missing check.

## 2. A public corpus of memory-write claims

**Origin.** The blind set behind the measured accuracy numbers, plus the adversarial cases the
discussions above contributed.

**Why.** Comparable tooling needs a shared set to compare against; a small labelled dataset of
memory and skill lines (claim / not a claim, with the adversarial shapes labelled separately) is
cheap to publish and gives the measurement a home.

**Waiting on.** Candidate 1 landing. The dataset should describe the tool as it behaves then, not as
it behaves today.
