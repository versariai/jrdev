# Adversarial review of the jrdev.ai plan — round 9

Reviewed 2026-10-01. Scope: the overview and all seven specifications, with emphasis on the fixes to rounds 7 and 8. The plan and previous reports were not edited.

**Result: three medium findings; no new high-severity finding established.** The recent fixes supply the missing contracts identified in the previous rounds. This pass finds narrower gaps in persistence, counting, and the lifecycle of the new planning dataset. These are design findings, not observed implementation failures.

## Previous findings

| ID | Design disposition |
|---|---|
| P7-01 | Attempt IDs, revisions, duplicate handling, reopening, and retained score history are now specified. The unit used by the pilot gate still needs clarification: P9-02. |
| P7-02 | The environment contract now expressly applies to both routes, including infrastructure errors. |
| P7-03 | Separate append-only assistance observations and sticky disqualifiers replace the conflicting status enum. |
| P7-04 | Numeric planning scores, separate consent, a task-combination rule, a sufficiency threshold, and a fallback are now specified. The new analysis copy needs a lifecycle: P9-03. |
| P8-01 | Restore quarantine and a separate instruction ledger now protect against replaying old consent. The ledger's write/completeness contract remains unspecified: P9-01. |
| P8-02 | A separate metadata collector now supplies common configuration fields in both arms, with explicit unknown states and its own data inventory. |

These dispositions acknowledge the changes without claiming their acceptance cases have passed.

## Findings

| ID | Severity | Finding |
|---|---|---|
| P9-01 | Medium | Consent changes can be acknowledged without a recoverable ledger entry |
| P9-02 | Medium | Assessment feasibility counting mixes attempts with a learner denominator |
| P9-03 | Medium | Planning-score copies are outside the explicit assessment deletion procedure |

### P9-01 — The instruction ledger needs a durable write and completeness contract

**Evidence:** [Feedback restore rules](specs/01-website-and-feedback.md) say deletions and consent changes are also written to an append-only ledger in a separate store. Restores replay its latest revisions and block processing if it is unavailable or incomplete. The spec does not define write order, when a request may be acknowledged, retry behavior, or how completeness is detected across those two stores.

**Failure case:** the primary database records a withdrawal and the user receives success. The process crashes before the separate ledger append is durable. A later restore recovers an older database and an apparently healthy ledger that lacks the withdrawal. The ledger is reachable and syntactically valid; without a completeness check, recovery cannot tell that it is missing an acknowledged instruction. The new restore procedure then repeats the failure P8-01 was intended to prevent.

**Required change:** choose a crash-consistent protocol for acknowledged changes. For example, make the durable ledger instruction authoritative before acknowledging success, then apply it idempotently to the primary database; processing must respect any unapplied withdrawal. Another design is acceptable if it gives the same recovery guarantee. Define operation IDs/revisions, duplicate retries, durable progress markers, and what establishes that replay is complete. Do not rely solely on the ledger's availability to establish completeness.

**Acceptance before the collection channel opens:** inject interruption before and after each durable write and before the success response. Retry the same withdrawal/deletion, then restore an earlier database. Every acknowledged instruction remains effective; retries are idempotent. An uncertain or incomplete ledger blocks external processing/publication. No real participant data is needed for these fixtures.

**Relationship to P8-01:** the separate ledger and quarantine are retained. This finding specifies how their inputs remain trustworthy through a partial write; it does not treat the restore fix as absent.

### P9-02 — The assessment gate needs a participant-level numerator

**Evidence:** [Manual assessment counting](specs/04-assessment-and-grading.md) now says each attempt ID counts once, regardless of revisions or rescoring. [Pilot decision table](specs/07-evaluation-and-studies.md) measures pilot-scored assessments taken against a threshold of at least 50% of completers. It does not explicitly require distinct completer learners in the numerator, or define which delayed attempt qualifies.

**Failure case:** six learners complete the pilot. One completer takes three separately assigned, scored assessments; the other five take none. Following the attempt-count rule produces three assessments divided by six completers, meeting 50%. A learner-level completion measure produces one of six, approximately 16.7%, which falls below the Stop threshold. Deduplicating revisions does not prevent this discrepancy because the assignments have different attempt IDs.

**Required change:** write the formula explicitly: distinct completer learners with at least one qualifying delayed, pilot-scored assessment divided by all completers. Specify the delayed-assessment window, whether the first or a designated attempt qualifies, and the handling of non-completers and infrastructure failures. Retain attempt counts as a separate operational measure. If the intended metric is instead assessments per learner, give it that name and a matching denominator/decision rule.

**Acceptance before freezing the pilot protocol:** six completers with three scored attempts from one person yield 1/6, not 3/6, under a participant-completion metric. Three completers with one qualifying attempt each yield 3/6. Duplicate revisions, appeals, attempts outside the window, and non-completer submissions cannot inflate the qualifying numerator. Define the zero-completer case explicitly.

### P9-03 — Planning data needs withdrawal and deletion propagation

**Evidence:** [Planning-score contract](specs/04-assessment-and-grading.md) adds a separate consent scope and numeric scores keyed by attempt ID, task/rubric version, and assistance observations. Its deletion procedure removes archives, exposure/custody rows, and the identity mapping. [Privacy inventory](specs/05-data-flows-and-privacy.md) adds a private analysis sheet retained for pilot duration plus 12 months, separately from custody scores retained for six months. It does not add the analysis sheet to that deletion procedure or specify how planning-consent withdrawal affects derived planning inputs.

**Failure case:** a learner withdraws planning consent or requests deletion after their score is copied into the analysis sheet. The coordinator deletes the listed assessment copies and mapping entry, while the research owner keeps using the analysis row. Deleting the mapping first can also leave the team unable to locate all the learner's derived rows. Calling the dataset de-identified does not itself specify whether the retained attempt IDs remain linkable or how consent is checked.

**Required change:** define the planning dataset's owner, consent check at export/use, withdrawal procedure, and deletion propagation through analysis rows, exports, and unpublished derived inputs. Resolve all relevant attempt IDs before deleting the mapping. State whether any genuinely irreversible aggregate can be retained and how that limit is disclosed; do not silently treat an attempt-keyed sheet as such an aggregate. Extend the assessment trace test to include the analysis copy and its longer retention period.

**Acceptance:** export synthetic scores with valid planning consent, withdraw one participant's scope, and delete another participant's assessment data. Their rows cease contributing to future planning runs, and affected unpublished inputs are recomputed or invalidated under the stated rule. The unchanged consenting participant remains usable. Exercise withdrawal before export as well as after export, with the research owner and coordinator following the same procedure.

## Verification and limits

The review compared the revised assessment, feedback, inventory, and study contracts and checked their relationships to the overview, command registry, workflow, and release specs. The source hashes below and local report links were verified. The numerical example in P9-02 is a constructed counterexample, not pilot data.

No plugin, database recovery, grading container, provider deletion, or statistical procedure was executed. The proposed acceptance cases remain work for the appropriate prototype or protocol gate. No external factual claim is needed for these document-contract findings.

## Reviewed snapshot

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `c0347d66887e77208e0f54001e766135f0095c2a38a46e37ea3822fc851053cc` |
| `specs/01-website-and-feedback.md` | `c279eac8eeade597a67287dbed68fe2a7e1b2c38a422c0801809a42ef9b3d0d3` |
| `specs/02-commands-and-policy.md` | `2ce5832aec2d18507ec2c5c2197f421a2be789bedf318d8ec1f15135dc403e8b` |
| `specs/03-modes-and-workflows.md` | `70e976e367065427d689f380d7ab83a7849e0c387592827a2bfa51dc7cbd6ea8` |
| `specs/04-assessment-and-grading.md` | `12cea209feff7aaedce67a9b5c8f6ed8a6e06c74b1ba91e8bdc9697ff46d9e2d` |
| `specs/05-data-flows-and-privacy.md` | `b9c46c3d8920bc3b4e3365e54e6e83820584952bd20f6833daabffe9c1a4701d` |
| `specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `specs/07-evaluation-and-studies.md` | `883382bcb97a37febd861d1b7c676da41d1252c6d900b9cb78e448322da08aff` |
