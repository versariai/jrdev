# Adversarial review of the jrdev.ai plan — round 10

Reviewed 2026-10-01. Scope: the revised overview and specifications, checked against round 9. This pass concentrates on the new ledger protocol and the pilot's decision rules. No plan or earlier report was edited.

**Result: two medium findings; no new high-severity finding established.** The prior fixes are present at the design level. The remaining issues concern ledger maintenance and an incomplete decision interval. These are static design findings, not observed production failures.

## Round 9 disposition

| Finding | Current design disposition |
|---|---|
| P9-01, durable instruction writes | The ledger is now authoritative, durably committed before success, read by jobs, and checked against mirrored heads. Crash fixtures are specified. P10-01 concerns subsequent pruning, not the absence of this write protocol. |
| P9-02, attempt versus learner counts | The delayed-assessment gate now uses distinct completers and defines its window, qualifying assignment, and zero-completer case. |
| P9-03, planning-data deletion | Planning IDs, an explicit pseudonymization claim, consent checks, ordered deletion, derived-input invalidation, and mapping deletion last are now specified. |

Implementation acceptance remains outstanding; the document changes do not demonstrate that those tests have passed.

## Findings

| ID | Severity | Finding |
|---|---|---|
| P10-01 | Medium | Ledger pruning lacks a compatible verification and current-consent contract |
| P10-02 | Medium | Fractional support medians fall between decision bands |

### P10-01 — Pruning must preserve the ledger's verification contract

**Evidence:** [Feedback instruction ledger](specs/01-website-and-feedback.md) requires gapless sequence numbers, a hash chain containing predecessor hashes, and a head at or beyond every mirrored head. The same section requires pruning entries after their retention period. Jobs read the latest ledger revision as their authoritative consent source. No checkpoint, segment rotation, or retained-current-state rule connects pruning to these requirements.

**Failure case:** entries for different items are interleaved. One item's retention expires while another item's instructions must remain. Removing the expired entries leaves sequence gaps and missing predecessor entries, so the declared restore check treats legitimate maintenance as corruption and blocks processing. Keeping the entries indefinitely avoids the gap but violates the declared pruning policy. Rewriting the remaining chain changes its hashes without a defined relationship to mirrored heads or old backups.

There is also no explicit rule for preserving an active item's authoritative consent if the entry carrying its latest revision becomes eligible for pruning. A database cache cannot silently replace the ledger's stated authority.

**Required change:** define retention-compatible ledger maintenance. Specify authenticated checkpoints or sealed segments, the sequence range a verifier expects, and how checkpoint/head updates and pruning recover after interruption. Define how active consent revisions remain authoritative and how deletion instructions remain effective for every restorable backup. State exactly when an instruction can disappear; a generic pruning statement is insufficient.

**Acceptance before enabling pruning:** interleave instructions for two synthetic items, expire only one item's retention, and compact/prune. Verification still succeeds for the retained state, expired content is removed under the declared policy, and current consent stays unchanged. Restore backups on either side of the checkpoint and interrupt maintenance at each durable boundary. Legitimate pruning must be distinguishable from missing unexpired instructions.

**Scope:** this is an inconsistency between the plan's own maintenance and verification rules. It does not require a particular storage product or assert that the proposed durable-write mechanism has failed.

### P10-02 — Decision intervals must cover fractional medians

**Evidence:** [Pilot support-burden row](specs/07-evaluation-and-studies.md) uses the median of all staff minutes per learner per week. Proceed is at most 30 minutes, Revise is 31–60, and Stop is above 60. No rounding rule is specified. The overall gate requires each row to produce a decision.

**Counterexample:** for eight learners with support totals of `30, 30, 30, 30, 31, 31, 31, 31`, the median is **30.5 minutes**. It is neither at most 30, at least 31, nor above 60. This example uses integer inputs and a pilot size explicitly allowed by the plan; sub-minute time tracking is not needed to expose the gap.

**Required change:** use complete, non-overlapping intervals—for example, Proceed `≤ 30`, Revise `> 30 and ≤ 60`, Stop `> 60`. Alternatively, prescribe rounding before classification and explain its effect. Apply the same convention to percentage bands, which also use integer-looking ranges, so a future cohort size or denominator change does not introduce an unclassified fractional value.

**Acceptance before freezing the protocol:** classify support medians `30`, `30.5`, `31`, `60`, and `60.5`, including the eight-learner fixture above. Every valid value gets exactly one decision. Check percentage boundaries using the chosen exact-value or rounding convention. No decision should require choosing a rounding policy after pilot results are observed.

## Verification and limits

The revised feedback, assessment, privacy, and study contracts were checked against the round-9 snapshot; unchanged command, workflow, and release contracts retain their prior review scope. Source hashes and local report links were verified. Python's median calculation confirmed the synthetic 30.5-minute counterexample.

No plugin behavior, database persistence, ledger restore, provider retention, or pilot outcome was tested. The acceptance scenarios above are proposed checks for implementation and protocol development. No external factual claim was needed for these findings.

## Reviewed snapshot

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `a8a474bae2fb1d7fde811f4943f0ba03242412099f7fb616446175201e7bbf9d` |
| `specs/01-website-and-feedback.md` | `60876e0c2afaeb61411b028059a31f81aeae4ab64de6fd77b2dbeb6094ccd0af` |
| `specs/02-commands-and-policy.md` | `2ce5832aec2d18507ec2c5c2197f421a2be789bedf318d8ec1f15135dc403e8b` |
| `specs/03-modes-and-workflows.md` | `70e976e367065427d689f380d7ab83a7849e0c387592827a2bfa51dc7cbd6ea8` |
| `specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `specs/05-data-flows-and-privacy.md` | `a8bde54c70baacbe1b2fec0c3b76521eea5ed204282a18822646fbed7235f366` |
| `specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `specs/07-evaluation-and-studies.md` | `cb1f2a8269998a1679e44d92db8fee58e974f320aed1b27f2decdb82c8308efd` |
