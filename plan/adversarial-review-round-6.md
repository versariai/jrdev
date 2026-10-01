# Adversarial review of the split jrdev.ai plan — round 6

Reviewed 2026-10-01. Scope: the overview and all seven specifications. This round focuses on contracts between documents, phase gates, and failure cases introduced or exposed by the split. The plan and previous reports were not edited.

**Result: 2 high and 4 medium findings.** The split improves navigation and preserves the historical section map, but the documents do not yet agree on several implementation and evaluation contracts. Resolve the high findings before treating Phase 1 as implementation-ready.

## Findings

| ID | Severity | Finding |
|---|---|---|
| P6-01 | High | The recommended manual Phase 1 assessment route has no compatible assessment or privacy contract |
| P6-02 | High | Default-disabled installation is conditional, so the first-execution guarantee remains too broad |
| P6-03 | Medium | Feedback lifecycle acceptance is gated after Phase 0 already collects real data |
| P6-04 | Medium | The command and hook layout omits interfaces required by other specs |
| P6-05 | Medium | The evaluation oracle permits stage-4 output after its authorization grant is consumed |
| P6-06 | Medium | Finalized assessment results lack an explicit eligibility rule for independent evidence |

### P6-01 — High: the recommended manual assessment route has no compatible contract

**Evidence:** [Overview, roadmap and decision 11](jrdev-ai-plan.md) recommends a manual Phase 1 process with one scorer, a hash log, and emailed submissions. [Assessment spec](specs/04-assessment-and-grading.md) instead explicitly requires an assessment service in Phase 1: task selection from an authoritative exposure ledger, one-time upload tokens, identity-free grader payloads, signed finalization, and controlled submission storage. [Privacy inventory](specs/05-data-flows-and-privacy.md) describes that service's archive storage and deletion lifecycle, not email attachments or scorer mailboxes.

**Failure case:** the team follows the recommended small-pilot route. Submissions arrive in the scorer's inbox with sender identity, defeating the specified blinding; copies remain in email and downloads outside the inventory. A hash log does not supply the required signed finalization or exposure service. The pilot either violates the detailed contract or cannot produce the finalized assessment records counted by its decision table.

This is not an objection to a manual pilot. It is an unresolved choice between two materially different Phase 1 systems. No precedence rule tells an implementer which requirements the recommendation defers.

**Required change:** choose the active Phase 1 route before implementation and update specs 04, 05, and 07 together. If manual, define task assignment and exposure tracking, pseudonymous submission routing, scorer blinding, archive custody, deletion of email/download copies, and exactly what evidence a manual result can create. Retain sandbox isolation before executing submissions. Explicitly defer service-only guarantees instead of labelling a hash log as equivalent to signed service results.

**Acceptance:** trace one real pilot attempt from assignment through scoring, the pilot denominator, and deletion using only the selected route. Every identity-bearing copy has an owner and retention rule; every result category is supported by that route's actual controls.

### P6-02 — High: default-disabled installation does not cover every supported configuration

**Evidence:** [Release trust](specs/06-repo-and-release-trust.md) now sets `defaultEnabled: false` and states that installation remains disabled and no jrdev code runs before explicit enablement. This is a useful response to P5-01, but the guarantee exceeds the mechanism.

Claude Code documents that a marketplace entry can override the manifest's default, existing `enabledPlugins` choices persist, and an enabled plugin's dependency starts enabled regardless of this default. These are documented exceptions, not hypothetical hook bypasses. [Claude Code plugin manifest reference](https://code.claude.com/docs/en/plugins-reference#defaultenabled).

**Failure case:** jrdev is installed as a dependency of an already-enabled plugin, or through an entry that overrides its default. The learner relies on the stated install → verify → enable sequence and starts a session before verification. The manifest alone does not establish the assumed disabled state.

**Required change:** constrain the documented verification procedure to configurations in which the effective plugin is demonstrably disabled. Cover marketplace metadata, existing settings, and enabled dependency relationships before relying on installation as safe staging. Where that cannot be established, specify a staging process that does not expose the unverified artifact to an active Claude Code session. Keep the existing admission that this is a user procedure rather than a client-enforced signature gate.

**Acceptance:** test a clean direct install, a conflicting marketplace default, a retained enabled setting, and installation as an enabled dependency. Instrument handler execution: each supported verification route must reach zero jrdev handlers before verification. Unsupported routes must be identified before installation, rather than covered by an unconditional promise.

### P6-03 — Medium: the privacy gate occurs after discovery collection

**Evidence:** [Overview gates](jrdev-ai-plan.md) and the [website spec header](specs/01-website-and-feedback.md) gate feedback consent/deletion acceptance before Phase 1 pilot data collection. Phase 0 already launches newsletter and feedback collection and conducts 10 junior and 5 mentor interviews.

**Failure case:** discovery launches on schedule with provisional forms and a privacy page, while the tested deletion receipt, consent recheck, provider retention, and removal of derived notes are scheduled for the later gate. Real discovery data has already entered the system when lifecycle acceptance finally becomes mandatory.

The substantive privacy requirements are present; the readiness milestone is misplaced. A privacy page or drafted consent form does not demonstrate those operations work.

**Required change:** move acceptance for each collection channel before its first real participant or subscriber, including Phase 0. Permit team-generated fixtures while unfinished. Make the milestone cover the selected newsletter provider and the actual interview storage/notes flow as well as website feedback.

**Acceptance:** before discovery opens, exercise consent withdrawal and deletion on representative feedback, subscription, and interview records, including queued jobs and derived notes. Document backup retention and any copies that cannot disappear immediately.

### P6-04 — Medium: cross-spec interfaces are missing from the technical layout

**Evidence:** [Commands and policy](specs/02-commands-and-policy.md) declares `/jrdev:task start|done`, while [Typing](specs/03-modes-and-workflows.md) depends on that user command for authoritative task boundaries. The technical layout's skill list omits `task` and supplies no alternative command source. Its hook event list also omits `PostModelSwitch`, required by [configuration logging](specs/07-evaluation-and-studies.md), and `DirectoryAdded`, used by the [privacy spec](specs/05-data-flows-and-privacy.md) for additional-root handling.

**Failure case:** an implementer follows the declared layout. The learner cannot create the task required for `/stuck`, or the plugin silently lacks the observers that the other specs promise. Separate teams can each satisfy their document while the combined installation fails.

**Required change:** define one phase-aware command/event registry, with each interface's implementation location, consumer, and minimum supported client version. Add the missing command and applicable event registrations to the layout, or explicitly defer the dependent behavior and its claims. A sketch should be labelled incomplete if it is not intended as the installation contract.

**Acceptance:** the clean-install check enumerates the command registry and invokes task start, escalation, and task done. Event fixtures reach the registered model-switch and additional-directory handlers in the phases that promise them. Compare the registry against every consumer spec so omissions fail review.

### P6-05 — Medium: the stage-only oracle misses consumed-grant leakage

**Evidence:** [Typing](specs/03-modes-and-workflows.md) makes stage 4 terminal, but permits a worked solution only during the turn bound to a fresh `(session_id, task_id, grant_id)`. The next user prompt consumes that grant. [Behavior-review oracle and pilot decision table](specs/07-evaluation-and-studies.md) instead define allowance and serious leakage from recorded `help_stage`; stage 4 allows a complete solution without an active-grant condition.

**Failure case:** reach stage 4, consume the grant with an ordinary follow-up, then receive another complete solution on the same task without a new `/stuck`. The workflow contract says there is no authorization for that solution turn. The stage remains 4, so the oracle can pass the output and omit a serious incident from the pilot gate. Merely recording that escalation was previously authorized does not distinguish these states.

**Required change:** derive the effective response allowance from both stage and current grant validity. Specify the coaching allowance after a stage-4 grant is consumed. Include session, task, grant, and turn bindings in fixtures and incident classification; distinguish requested escalation from authorization valid for the response being judged.

**Acceptance:** an identical complete solution passes during its grant's turn and fails on the next ordinary prompt. Also cover status commands, interruption, retry, resume, fork, and replay of a previously valid grant. Apply the same oracle to release checks and pilot leakage counts.

### P6-06 — Medium: finalization and independence eligibility are conflated

**Evidence:** [Assessment lifecycle](specs/04-assessment-and-grading.md) says finalized assessments create assessed-transfer records and raise demonstrated evidence. Self-reported help explicitly makes an attempt coached. Detected jrdev tool use, however, gets a separate validity flag with its score retained; the spec supplies no corresponding eligibility rule. [Study protocol](specs/07-evaluation-and-studies.md) correctly keeps flagged scores in the primary intention-to-treat analysis.

**Failure case:** a learner reports no help, but the product records AI tool use in the assessment directory. The human scorer finalizes the attempt and the service signs the result. A client following the evidence table promotes it to demonstrated ability because it is finalized, despite the positive assistance flag. Authentic grading and independence are different properties.

**Required change:** define separate result fields for finalization/authenticity, assistance or validity status, and eligibility for independent evidence. State which flags preclude a demonstrated claim and which merely disclose uncertainty; do not assume any tool call proves assistance without defining the signal. Preserve all-attempt study scoring without automatically importing every study result as an independently demonstrated product achievement.

**Acceptance:** assisted, positively flagged, clean, and uncertain attempts all retain their scores and provenance, but only eligible attempts raise independent evidence. Study primary analysis still includes every attempted score according to its protocol. This is a remaining ambiguity in the evidence contract, not a request to reverse the symmetric study design.

## What this round checked

The overview and all seven specs were read together, including their gates, command/event references, assessment lifecycle, privacy inventory, release procedure, and evaluation oracle. Existing local document targets resolve; a heading-slug check found no unmatched anchors among 36 checked links. This was a static document check, not browser rendering or a runnable plugin test.

The previous round's recovery, grant lifecycle, submission payload, storage, and grading-environment changes are represented in the split. P6-02 and P6-05 identify residual gaps in the revised contracts; they do not treat those fixes as absent. No learner data was collected, no grading code was run, and no implementation acceptance test was claimed.

## Reviewed snapshot

SHA-256 hashes identify the source text reviewed, so subsequent edits can be distinguished from this report.

| Source | SHA-256 |
|---|---|
| `jrdev-ai-plan.md` | `f81c9dbc8abdf04c2843cac9e5dd96bc8ec8bbb3fbac8ee83cd7e62fac100414` |
| `specs/01-website-and-feedback.md` | `9649b9b530d0f9c4556cd4ee81687aa9665d65597a516a22ca895cac90ad137e` |
| `specs/02-commands-and-policy.md` | `12d78b5e61f7f8fc0299fdb830deba7dc248945ebcf03ba191e09483c2088717` |
| `specs/03-modes-and-workflows.md` | `2441b811b1867553343914229fb7345afd56e7b5f11fa5365bd6ff376b495445` |
| `specs/04-assessment-and-grading.md` | `1ff742524398a273a673e75eb24f95274d90512406253d9c429c4017588dd398` |
| `specs/05-data-flows-and-privacy.md` | `8ca86e90d196715b11c4707149458ac88016fee88f4701134ca2a8436d89015f` |
| `specs/06-repo-and-release-trust.md` | `e2d7750078f8a6974ea1f4c7cd7fb701a81875cc9d6f482bf9bacc9f4dc43be2` |
| `specs/07-evaluation-and-studies.md` | `f77fc534d9c493da8a8143ffac971b5382a1858b1b881ec563ca56ce2d9f7d80` |
