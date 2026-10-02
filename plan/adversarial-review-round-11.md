# Adversarial review of the jrdev.ai plan — round 11

Reviewed 2026-10-01. Scope: the revised overview, feedback-ledger design, privacy inventory, and pilot decision rules, checked against round 10. The unchanged command, workflow, assessment, and release specs retain their previous review scope. No source plan or earlier report was edited.

**Result: one medium finding; no new high-severity finding established.** The revised decision bands resolve P10-02 at the design level. Segments and checkpoints address P10-01's missing maintenance contract, but historical checkpoint snapshots now conflict with the promised expiry of per-item markers.

## Previous findings

| ID | Disposition |
|---|---|
| P10-01 | Sealed segments, authenticated checkpoints, current-state snapshots, and an ordered maintenance protocol are now specified. The retained historical snapshot content is the narrower issue below. Operational crash and restore checks remain unexecuted. |
| P10-02 | Support and percentage bands now use complete, non-overlapping intervals with classification before display rounding. The 30.5-minute example is explicitly Revise. |

## P11-01 — Medium: permanent checkpoints retain expired item markers

**Evidence:** [Feedback checkpoint design](specs/01-website-and-feedback.md) says checkpoints form a chain that is never pruned. Each checkpoint records a state snapshot containing per-item IDs, consent scopes/revisions, and deletion flags. An item's current state leaves the snapshot after its retention conditions expire. [Privacy inventory](specs/05-data-flows-and-privacy.md) likewise gives minimal instruction markers a finite retention period.

**Failure case:** checkpoint C1 records item A's identifier and consent state. A is deleted, all relevant backups expire, and checkpoint C2 eventually omits A as required. C1 still contains A's snapshot, because the checkpoint chain is never pruned. Pruning the original ledger segment does not remove the duplicate in C1. The item disappears from authoritative current state but remains indefinitely recoverable from retained historical checkpoint content.

The specification does not distinguish a permanently retained verification header from an expiring snapshot payload, or explain whether the authenticated checkpoint commits to that payload separately. Removing an identifier from the newest snapshot is therefore not enough to implement the stated retention rule.

**Required change:** separate checkpoint verification metadata from retained per-item state. Define which small, content-free headers or commitments remain permanent, which snapshot payloads expire, and how historical payload removal preserves verification of the retained chain and current state. Keep a current recoverable snapshot and all deletion instructions needed by restorable backups. Include snapshot copies and backups in the inventory and expiry procedure. If an item-level payload cannot expire under the chosen design, disclose that retention rather than claiming the marker has disappeared.

**Acceptance before enabling checkpoint pruning:** create C1 containing two synthetic items; delete one and advance beyond every applicable retention and backup window; create later checkpoints and prune eligible material. Inspect every retained checkpoint payload and backup, not only the latest snapshot. The expired item's identifier, scopes, and deletion flags are no longer retrievable under the stated policy; the active item's consent remains available. The retained verification chain still validates and supported restores still apply current instructions. Permanent verification metadata must not embed the expired per-item snapshot.

**Relationship to P10-01:** this finding credits the checkpoint solution. It concerns a new duplication/retention consequence of keeping full historical snapshots, not a request to return to an uncheckpointed ledger.

## Verification and limits

Source hashes were compared with round 10 to distinguish revised documents from unchanged ones. The checkpoint retention statements were traced through the feedback spec and privacy inventory. Report links and reviewed source hashes were checked.

This is a static design review. No ledger implementation, cryptographic verifier, database restore, provider retention, or participant-data deletion was executed. The proposed acceptance scenario remains implementation work. No external factual claim was needed for this document-contract finding.

## Reviewed snapshot

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `2aef59900f3b742ccfc91a1c3ebfa7bf310a590f735e1739cde7693ad4345c53` |
| `specs/01-website-and-feedback.md` | `712a11517ba66f64fb23d2254dca27f6213764f51f2afdc36cc087f91dde9cea` |
| `specs/02-commands-and-policy.md` | `2ce5832aec2d18507ec2c5c2197f421a2be789bedf318d8ec1f15135dc403e8b` |
| `specs/03-modes-and-workflows.md` | `70e976e367065427d689f380d7ab83a7849e0c387592827a2bfa51dc7cbd6ea8` |
| `specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `specs/05-data-flows-and-privacy.md` | `9d94059ee3c48bdd8c2816a89d69167702779d60016e62511d8505a86efd70ee` |
| `specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `specs/07-evaluation-and-studies.md` | `7b88212afe1246ef408458e36287babd18d91278f9ea06070763e94546fde79e` |
