# Next candidates

Things learned from this record that are worth doing next. Nothing here has been started; each
entry is a candidate with its origin, its cheapest first step, and what it is waiting on.

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

**Constraint that decides the design.** Whatever appends a resolution must not be the agent that
wrote the claim, or the condition is graded by its author. In practice that means the user or CI.

**Waiting on.** The platform PR closing. No new line opens before that.

## 2. A public corpus of memory-write claims

**Origin.** The blind set behind the measured accuracy numbers, plus the adversarial cases the
discussions above contributed.

**Why.** Comparable tooling needs a shared set to compare against; a small labelled dataset of
memory and skill lines (claim / not a claim, with the adversarial shapes labelled separately) is
cheap to publish and gives the measurement a home.

**Waiting on.** Candidate 1 landing. The dataset should describe the tool as it behaves then, not as
it behaves today.
