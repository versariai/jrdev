# Adversarial review of the jrdev.ai plan — round 7

Reviewed 2026-10-01. Scope: the current overview and all seven specifications, checked against round 6. This is a document review; proposed acceptance cases below were not executed. No source plan or earlier review was edited.

**Result: four medium findings; no new high-severity finding established.** Round 6's changes resolve its main contradictions at the design level. The remaining concerns are primarily in the newly specified manual assessment route and its handoff to later evidence and study planning.

## Round 6 disposition

| Finding | Current disposition |
|---|---|
| P6-01, assessment route | Manual Phase 1 is now selected, with separate custody, privacy, and evidence rules. Remaining route details are P7-01, P7-02, and P7-04 below. |
| P6-02, installation defaults | Supported routes, pre-flight checks, marketplace defaults, and a route test matrix are now specified. Their operational correctness remains a prototype check. |
| P6-03, collection readiness | Acceptance now precedes each channel's first real collection, including Phase 0. |
| P6-04, missing interfaces | A command/event registry now includes task commands and the missing observers; layout precedence is explicit. |
| P6-05, grant-aware oracle | Both the oracle and pilot leakage measure now use grant validity as well as help stage. |
| P6-06, evidence eligibility | Finalization and eligibility are separate, and positively flagged attempts cannot raise independent evidence. The assistance field still needs an overlap rule: P7-03. |

These are design dispositions, not claims that implementation tests have passed.

## Findings

| ID | Severity | Finding |
|---|---|---|
| P7-01 | Medium | Manual submissions have hashes but no defined attempt/revision lifecycle |
| P7-02 | Medium | Manual grading inherits sandbox isolation but not a clear environment and infrastructure-error contract |
| P7-03 | Medium | Mutually overlapping assistance observations are represented as one status without precedence |
| P7-04 | Medium | Phase 3 requires a Phase 1 variance input that the manual-results contract does not define |

### P7-01 — Manual submissions need an attempt and revision lifecycle

**Evidence:** [Manual assessment lifecycle](specs/04-assessment-and-grading.md) records pseudonym, task code, archive hashes, timestamps, rubric version, and grader image. It does not define an attempt identifier, duplicate handling, replacement authority, appeal rescoring, or which submission becomes the canonical pilot result. The service's explicit attempt/revision and idempotency rules are deferred.

**Failure case:** a learner uploads twice for the same task, then retries after a transfer interruption. All three archives have valid hashes. The coordinator and scorer can reasonably choose different versions or create multiple scored rows. An appeal can replace the visible score without preserving which bytes and rubric produced the original. Matching hashes establish custody for an archive; they do not choose the authorized submission.

**Required change:** define a lightweight manual attempt ID and revision number, one accepted submission per revision, and the handling of identical and changed re-uploads. Require an explicit coordinator/scorer action to reopen an accepted attempt. Preserve original scores when rescoring, and state how multiple revisions contribute to the pilot assessment count. This can remain a private sheet and ordinary files; it does not require the deferred service.

**Acceptance:** process an identical retry, a changed second upload, an interrupted transfer, and an appeal. Each ends with one unambiguous canonical result per attempt, retains the earlier provenance, and contributes once to the intended feasibility count.

### P7-02 — Manual grading needs its own environment and error rules

**Evidence:** [Both-route grading environment](specs/04-assessment-and-grading.md) correctly applies isolation, resource limits, and hostile-code fixtures to Phase 1. However, the task environment contract—pinned runtime/dependencies, declared local-versus-grader differences, feature fixtures, and `infra_error` instead of a low score—remains inside the explicitly deferred **service route**. The manual route records the grader image after scoring but does not expressly adopt those requirements.

**Failure case:** a manual task's public tests run against a learner's installed runtime, while hidden tests run in a different grader image. A supported dependency is absent on the grading host. The sandbox safely contains the run, but the scorer records a failed pilot assessment caused by infrastructure. Recording the image identifies the mismatch afterwards; it does not define how the score or feasibility measure should treat it.

**Required change:** move the environment and infrastructure-error contract into a shared section or specify a smaller equivalent for Phase 1. Pin the grading environment per task before assignment, declare supported learner/public-test conditions, and distinguish a valid incorrect solution from a provisioning or grader failure. Define retries and their effect on the scored-assessment count.

**Acceptance:** a declared-dependency fixture works in the supported public-test and hidden-test environments. A missing dependency or wrong image is recorded as an infrastructure failure, not learner failure; a retry retains the same submitted bytes and does not create duplicate feasibility credit.

### P7-03 — Assistance observations overlap but have no precedence

**Evidence:** [Result fields and eligibility](specs/04-assessment-and-grading.md) defines one `assistance_status` with values `none-reported`, `self-reported-help`, `tool-flag`, and `monitoring-unavailable`. The first two describe learner reporting, the third describes an observed tool event, and the fourth describes monitoring availability. Eligibility accepts `monitoring-unavailable` but rejects reported help and tool flags.

**Failure case:** a learner reports help while hooks were unavailable. Both statuses are true. If a later health check writes `monitoring-unavailable` into the single field, a disqualifying report can become an eligible state. Similarly, monitoring can fail after a tool flag has already been recorded. The current contract provides no precedence or preservation rule for these combinations.

**Required change:** retain separate observations for self-report, detected events, and monitoring coverage, then derive eligibility. Alternatively, define explicit precedence and retain the underlying observations so a non-disqualifying status cannot erase known assistance. Recompute eligibility when new evidence arrives; do not upgrade an attempt merely because monitoring later stops.

**Acceptance:** exercise reported help plus unavailable monitoring, a positive tool flag followed by a hook failure, and no reported help with partial monitoring. Scores and every observation remain intact. Either disqualifying assistance signal keeps the attempt ineligible regardless of processing order.

### P7-04 — The later study's variance input lacks a pilot data contract

**Evidence:** [Phase 3 study planning](specs/07-evaluation-and-studies.md) sets sample size from Phase 1 variance and a precision target. [Manual result scope](specs/04-assessment-and-grading.md) authorizes feasibility measures and qualitative scorer notes, returns rubric bands, and excludes manual results from assessed-transfer evidence. It does not define the quantitative score dataset used for that variance, its scale, or how different pilot tasks and rubrics are reconciled.

**Failure case:** the pilot satisfies its feasibility gate and retains only bands and notes. Phase 3 planning then expects a quantitative variance that nobody agreed to collect. Alternatively, different scorers privately retain incompatible scores and the team chooses a convenient interpretation after seeing them.

This is a missing input contract, not a claim that unsigned manual scores are inherently unusable for planning. Exploratory planning data can remain separate from product achievements and efficacy claims.

**Required change:** before the pilot, specify whether comparable numeric rubric scores will be collected for exploratory planning, with task/rubric versions, permitted use, retention, and consent. Define which observations contribute and how task differences are handled. If Phase 1 will not provide that input, choose another documented basis for the later precision calculation rather than promising a variance estimate from undefined data.

**Acceptance:** a synthetic pilot dataset produces the exact planning input required by the chosen Phase 3 procedure. Band-only data and incompatible task scores are either handled by a prespecified rule or explicitly reported as insufficient. Nothing is promoted to independent product evidence merely because it was used in study planning.

## Review limits and snapshot

The review covers the eight source documents together and compares their revised contracts with round 6. It does not validate Claude Code hook ordering, install behavior, sandbox containment, provider deletion, or statistical performance. Those remain implementation/protocol acceptance work. No external factual claim was needed for these document-contract findings.

SHA-256 identifies the reviewed text:

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
