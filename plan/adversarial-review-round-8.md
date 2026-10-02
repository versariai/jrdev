# Adversarial review of the jrdev.ai plan — round 8

Reviewed 2026-10-01. Scope: the overview and seven specifications, with a further pass over recovery, consent, and study observability. The source hashes are identical to round 7: no intervening plan revisions were detected. This report distinguishes new findings from unresolved findings rather than renumbering the latter.

**Result: two new medium findings, plus four medium findings carried forward from round 7. No new high-severity finding established.** This is a static review of the design, not an implementation or provider audit.

## Findings carried forward

See [round 7](adversarial-review-round-7.md) for evidence, proposed changes, and acceptance cases. All four remain open because their source contracts are unchanged.

| ID | Severity | Remaining gap |
|---|---|---|
| P7-01 | Medium | Manual assessment archives have custody hashes but no canonical attempt/revision lifecycle for retries, replacements, and appeals. |
| P7-02 | Medium | Phase 1 explicitly inherits sandbox isolation, but not the service section's task-environment and infrastructure-error contract. |
| P7-03 | Medium | Assistance reporting, tool observations, and unavailable monitoring overlap; the single status field has no preservation/precedence rule. |
| P7-04 | Medium | The later study expects a Phase 1 variance input, but the manual-results contract does not define its quantitative dataset or permitted planning use. |

## New findings

| ID | Severity | Finding |
|---|---|---|
| P8-01 | Medium | Restoring a feedback backup can resurrect deleted records and withdrawn consent |
| P8-02 | Medium | The comparison arm has no defined source for intervention-session configuration records |

### P8-01 — Consent and deletion need to survive backup restoration

**Evidence:** [Feedback lifecycle](specs/01-website-and-feedback.md) requires deletion of primary records, queued jobs, memberships, and derived notes, and checks current consent at processing/publication time. It allows backups to age out within 30 days. [Privacy inventory](specs/05-data-flows-and-privacy.md) stores consent records in the same jrdev database and retains them only while the item exists. Neither document specifies how a restored database is reconciled with deletions or withdrawals performed after the backup was taken.

**Failure case:** a backup contains a feedback item, an enabled LLM consent scope, and a queued processing job. The submitter later withdraws consent or deletes the item. After a database failure, the team restores that earlier backup and resumes workers. Reading the restored database's “current” consent passes the existing check, although the submitter's latest instruction was withdrawal. A deleted item can also return to theme counts or publication queues.

The disclosed backup retention window explains why bytes may remain temporarily. It does not define permission to reactivate those bytes after recovery. This extends the existing lifecycle requirements; it does not demand immediate removal from every backup.

**Required change:** include deletion and consent reconciliation in the restore runbook. Preserve minimal deletion markers and latest consent revisions independently of the snapshot being restored, or use an equivalent mechanism that cannot roll those instructions back. Keep jobs, publication, and restored records quarantined until reconciliation finishes. Define retention and access for these minimal markers so the remedy does not become an indefinite copy of deleted content. If the latest instructions cannot be recovered, block external processing/publication until permission is re-established.

**Acceptance before the affected collection channel opens:** create and back up three synthetic items; delete one and withdraw LLM/publication consent from another; restore the old snapshot. The deleted item stays absent from active records, counts, and jobs. The withdrawn item never reaches a processor or publication step. A third unchanged item resumes normally. Also exercise loss of the reconciliation store: processing remains blocked rather than treating old consent as fresh authorization.

**Scope limit:** this finding concerns jrdev-controlled feedback/database recovery. Provider-specific backup handling needs its own documented guarantees; no claim is made that a particular provider currently restores records this way.

### P8-02 — Configuration observations need a source in the comparison arm

**Evidence:** [Study design](specs/07-evaluation-and-studies.md) compares jrdev with the same AI tool **without jrdev**. Its configuration protocol names jrdev's per-session logs and `PostModelSwitch` observer, freezes the model setting for both arms, and promises publication of observed versions/configurations. [Interface registry](specs/02-commands-and-policy.md) places that observer inside the jrdev plugin. No independent source, manual collection procedure, or explicit observation limit is defined for AI sessions in the comparison arm.

**Failure case:** a treatment participant changes models and creates a jrdev log entry. A comparison participant does the same, but runs no jrdev observer. The team can verify changes and version provenance for one arm while only assuming them for the other. Running the existing acceptance simulation solely with jrdev installed would not expose the missing comparison-arm record.

The shared hosted assessment workspace already addresses assessment-condition symmetry. It does not record which AI configuration participants used during the preceding intervention sessions. This finding is separate from P4-01's resolved assessment-validity issue.

**Required change:** specify an arm-neutral source for the common configuration fields: for example, a separately consented metadata collector without tutoring behavior, a controlled study launcher, or a documented manual procedure. Name what is observed versus merely configured or self-reported in each arm. Keep treatment-only fields such as `/stuck` usage distinct, and record plugin identity as absent/not applicable for the comparison arm. If comparable observations cannot be collected, narrow the fidelity claims and report that limitation explicitly rather than implying the plugin's log covers both arms.

**Acceptance before the efficacy study:** simulate an initial session, a model-setting change, a client update, and missing metadata separately in both arms. Each yields the specified common configuration record or explicit unknown state. The comparison arm receives no jrdev tutoring or edit-policy behavior through the measurement mechanism. Check the collector's data inventory and consent, and avoid collecting conversation content merely to obtain configuration metadata.

## Review outcome and limits

The next useful design changes are the four unresolved round-7 contracts and the two recovery/observability contracts above. More unchanged-document reviews do not establish that the prototype works. The existing command, grant, install, sandbox, and deletion acceptance gates still need actual execution at their designated milestones.

This pass rechecked the unchanged source snapshot and traced the new failure cases through the relevant specs and interface registry. Report links and source hashes were checked. No source document, prior report, or unrelated workspace file was modified. No external service, participant data, or executable plugin was used to validate the proposed scenarios.

## Reviewed snapshot

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `3f83d24d1c8469e9b7a7d35ef4935f07363c84811aeb76ea744665458dc2cf6b` |
| `specs/01-website-and-feedback.md` | `46263ff8171bbdab04715a42f990daa4a13369d21c3d5b11368ad2f4a64bfdde` |
| `specs/02-commands-and-policy.md` | `2ce5832aec2d18507ec2c5c2197f421a2be789bedf318d8ec1f15135dc403e8b` |
| `specs/03-modes-and-workflows.md` | `70e976e367065427d689f380d7ab83a7849e0c387592827a2bfa51dc7cbd6ea8` |
| `specs/04-assessment-and-grading.md` | `fd7dd409e7b0eb84dce39875f8af6058e1eb2e5bb1e85d12c23b44c07b812bfd` |
| `specs/05-data-flows-and-privacy.md` | `85c24a9d46ca4d11ceaaa88409673d724ab4cb7f81071269aa56c10ae6bb77fe` |
| `specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `specs/07-evaluation-and-studies.md` | `4d45fee24de49d9b9903179ac7759a6836e34ca271f38ec6f9b4be2e72cbc015` |
