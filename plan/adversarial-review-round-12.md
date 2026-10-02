# Adversarial review of the jrdev.ai plan — round 12

Reviewed 2026-10-01. Scope: the changes since round 11, their privacy-inventory contract, and their relationship to the existing plan. The overview, feedback spec, and privacy spec changed; the other five specifications match the previously reviewed hashes. No source plan or previous report was edited.

**Result: no new concrete document-level finding established in this pass.** P11-01 is addressed at the design level. This is not an implementation approval or a claim that the acceptance gates have passed.

## Round 11 disposition

[P11-01](adversarial-review-round-11.md) identified indefinite retention of item identifiers inside historical checkpoint snapshots.

The revised [feedback spec](specs/01-website-and-feedback.md) now separates a checkpoint into:

- A permanent signed header containing checkpoint metadata and a payload commitment, without per-item state.
- An expiring payload containing the current item identifiers, consent revisions, and deletion flags.

Previous payloads are deleted after a new checkpoint is durably written, verified, and referenced by both mirrored heads. The verifier expects old payloads to be absent. The [privacy inventory](specs/05-data-flows-and-privacy.md) separately lists header retention, payload retention, and the additional 30-day ledger-backup window. The proposed expiry fixture inspects historical copies as well as current state.

That resolves the specific conflict between permanent verification history and finite item-marker retention. The earlier report remains an accurate record of the snapshot it reviewed.

## Adversarial cases considered

| Case | Current design response | Remaining verification |
|---|---|---|
| Expired item survives in an old checkpoint | Historical payload deletion and bounded backup retention; permanent headers exclude item state | Inspect all retained payloads and backups after expiry |
| Restore tries to replay a pruned segment | Apply authoritative current state; verify retained headers and the latest payload | Restore snapshots taken on both sides of checkpoint maintenance |
| Crash before new checkpoint publication completes | Previous checkpoint remains authoritative before the mirrored-head update | Exercise each intermediate state, including updating only one mirror |
| Crash during segment or payload deletion | Covered segments or an old payload may linger; maintenance retries | Repeat recovery and maintenance without losing current consent |
| Consent withdrawal remains unapplied in the main database | Ledger-derived current consent is authoritative for processing jobs | Run a job while database application is delayed |

These are checks of the stated contract, not executed tests. The design's guarantees depend on a consistent checkpoint snapshot, serialized sequence/revision allocation, and readers selecting a fully published checkpoint. The specification's correctness claims should be tested against concurrent consent writes during maintenance, in addition to its listed interruption fixtures. No separate finding is asserted merely because those implementation details remain to be built.

## What remains unproven

The next evidence should come from the Phase 0 prototype and channel-readiness fixtures. In particular, implement and exercise durable acknowledgment, idempotent retries, checkpoint publication, backup reconciliation, payload expiry, and concurrent writes using synthetic records. Preserve the plan's rule that an unavailable or incomplete authority blocks external processing/publication.

The unchanged plugin, assessment, release, and study contracts retain their previous review dispositions. Their operational correctness has not been established by this targeted pass. A document review cannot establish hook ordering, installation safety, sandbox containment, provider deletion, or study fidelity.

## Verification

Compared all eight source hashes with round 11, read the revised feedback and privacy contracts, and checked the overview's review-history update. Verified this report's local links and snapshot hashes. No external services or participant records were accessed, and no implementation acceptance test was run.

## Reviewed snapshot

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `59eaf1d1700c3a6891e1b1123a295f2315d5a16772d46bad67426288fac4910c` |
| `specs/01-website-and-feedback.md` | `42ff48ab63a808939389b1231e0130b0dddde4b82c819bc7c45fb47d226667c4` |
| `specs/02-commands-and-policy.md` | `2ce5832aec2d18507ec2c5c2197f421a2be789bedf318d8ec1f15135dc403e8b` |
| `specs/03-modes-and-workflows.md` | `70e976e367065427d689f380d7ab83a7849e0c387592827a2bfa51dc7cbd6ea8` |
| `specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `specs/05-data-flows-and-privacy.md` | `6b448ef75accb2e2cf676f6be9367b5a81601f0ea775800577ea8f1cbf230e5c` |
| `specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `specs/07-evaluation-and-studies.md` | `7b88212afe1246ef408458e36287babd18d91278f9ea06070763e94546fde79e` |
