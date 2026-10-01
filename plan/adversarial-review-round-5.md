# Adversarial review of the jrdev.ai plan — round 5

Reviewed: 2026-10-01. Scope: [the product plan](jrdev-ai-plan.md), compared with [round 4](adversarial-review-round-4.md) and the earlier review history. The plan and previous reports were not edited.

Source SHA-256: `6bfd47591b89e6fbe3fe9ad5b5858e1556257f4475cdaed6d177761ae0bf49b0`.

## Verdict and previous findings

The plan now specifies symmetric study assessments, immutable submission identity, authenticated finalization, task commands, policy capabilities, staged-blob checks, project identity, and a server-side exposure ledger. These substantially address round 4 at the design level. Their acceptance cases still need implementation evidence.

P4-03 is only partly resolved: explicit task transitions now exist, but the acceptance case that an ordinary question about task B automatically starts at stage 0 exceeds the specified mechanism. Only a user command changes the task; model recognition is advisory. Either require the explicit transition in that acceptance case or narrow the promise. This is a carry-forward clarification, not a new finding counted below.

This round adds **1 high-priority and 5 medium-priority findings**. The high issue concerns verification before plugin execution. The medium issues concern recovery lifecycle, response authorization, storage deletion, submission scope, and assessment environment parity. Failure scenarios are design inferences, not reproduced jrdev defects.

| ID | Priority | Finding | Evidence |
|---|---|---|---|
| P5-01 | High | Integrity checks have no required gate before activation or update execution | §7.4, §8.3, §9 |
| P5-02 | Medium | Recovery overrides can outlive the recovery and defeat re-enabling | §5.1–5.3 |
| P5-03 | Medium | Single-response solution authorization has no consumption or fork lifecycle | §5.4, §5.7 |
| P5-04 | Medium | Write-once storage is not yet reconciled with deletion promises | §5.5, §6.3 |
| P5-05 | Medium | Assessment packaging has no minimal-file contract | §5.5, §6.3 |
| P5-06 | Medium | Recording the grader image does not establish learner/grader environment parity | §5.5, §8.1 |

## Findings

### P5-01 — A valid signature checked after execution is too late

**Evidence:** The independent procedure locates an installed plugin, verifies a manifest, and hashes its files. The plan does not require fetching and checking artifacts before enabling/loading hooks, or rechecking an updated artifact before activation. It explicitly assumes Claude Code does not verify release signatures. Updates are marketplace SHA changes.

**Failure scenario:** A learner installs the plugin and starts a session before performing the manual integrity check. SessionStart code has already run. Similarly, a marketplace update can replace a formerly verified artifact; the next session or plugin reload can load it without a new manifest check. An immutable SHA identifies the selected release but does not authenticate a newly selected SHA or authorize it for this user.

Claude Code documents that enabled plugins load into sessions and that, when marketplace auto-update is enabled, updated versions load automatically in the next session. Third-party marketplaces default to auto-update off, but users can enable it. [Plugin management documentation](https://code.claude.com/docs/en/discover-plugins).

**Impact:** Independent verification is correctly separated from the bundled verifier, but the user flow does not yet establish verification before first execution. A successful later check cannot undo earlier hook activity.

**Required change:** Document and test a fetch/verify/enable sequence that runs no plugin code before verification. If the tool cannot enforce that sequence, describe the limitation rather than implying pre-execution protection. Define the same gate for every update, including marketplace refreshes, reloads, and dependencies. Keep auto-update disabled where manual verification is the trust mechanism, and specify how the approved manifest/SHA is compared with the artifact actually loaded.

**Acceptance:** On a clean machine, an unverified or altered artifact cannot reach a jrdev startup handler under the documented procedure. After a valid release is installed, change the marketplace entry and exercise reload and restart. New code is either verified before activation or the limitation is surfaced explicitly. No bundled verifier is executed to decide whether the bundle is trusted.

**Distinct from P2-08:** Manifest coverage and independent checking now exist; this finding concerns the ordering of verification and execution.

### P5-02 — Turning Typing back on may not remove the higher-priority override

**Evidence:** A recovery override outranks session state and causes edits to be allowed. The global `DISABLED` file outranks all other decisions. `/jrdev:type on` changes session edit policy, while the command/status contract promises clear notices about effective enforcement. No command removes or expires recovery overrides or the kill switch.

**Failure scenario:** The learner uses `jrdev off --session ...` to recover, repairs the malformed config, then runs `/jrdev:type on`. The stored policy is `type`, but the recovery override still wins. With the global kill switch present, even a successful session-state change cannot restore enforcement. The UI can report the requested setting while every edit remains allowed.

**Required change:** Define recovery states and transitions, including explicit re-enable behavior, validation of repaired lower layers, expiry if any, and the scope of removal. Status must show the effective decision and the overriding reason. A session command must not silently remove a global disable affecting other sessions; it should explain the separate action needed. Preserve the independent shell recovery path.

**Acceptance:** Recover, repair config, and re-enable Typing in one session while another remains active. The next Edit matches the displayed effective rule. Repeat with the global kill switch. The user sees why enforcement is still disabled and can restore it through a documented transition without changing another session unexpectedly.

### P5-03 — “Next response” needs a defined event and non-inheritable authorization

**Evidence:** Stage 4 now permits the next response only, within the task, while later solutions require another `/stuck`. The state persists across resume and compaction. The technical layout includes Stop for reminders, but no specified transition consumes a response authorization or handles interruption, API failure, and session fork.

**Failure scenario:** A response delivers the solution and is interrupted before Stop. The durable stage remains authorized when the session resumes. Alternatively, a fork copies conversation instructions saying stage 4 is allowed while its new session state contains no matching authorization. Two branches can appear to share one user authorization, or a clarification/status response can consume it before the requested solution is produced.

Claude Code distinguishes fork from resume at SessionStart, replays saved context, and does not run Stop on a user interrupt. The authorization lifecycle must account for those events rather than rely on a normal Stop callback. [Hooks reference](https://code.claude.com/docs/en/hooks).

**Required change:** Define whether the grant covers one assistant turn, one completed response, or another observable unit. Bind it to a task and turn identity and choose explicit consumption semantics for partial delivery, error, retry, and cancellation. Forked sessions should obtain fresh authorization rather than inherit a consumable grant. Reinject authoritative state so replayed instructions do not imply an active grant. Keep solution-content compliance labelled advisory; state bookkeeping alone cannot guarantee what the model says.

**Acceptance:** Interrupt after partial and complete solution delivery, resume, compact, fork, and retry an API failure. The documented consumption rule applies without two qualifying solution turns sharing one grant. A status query cannot silently create or revive stage-4 authorization.

**Distinct from P4-03:** Explicit task identity now exists. This concerns consuming the new single-response grant, not recognizing a task change.

### P5-04 — Immutability and deletion need a deliberate storage choice

**Evidence:** Accepted archives are in write-once storage, with “object lock / no overwrite” offered as mechanisms. The inventory promises deletion with the assessment attempt. The plan does not distinguish immutable revisions from retention locks that prohibit deletion or specify physical-object handling for content-addressed storage.

**Failure scenario:** The chosen backend applies a retention lock, but the user deletes the attempt before retention expires. The record disappears while the archive remains recoverable. A successful ordinary delete API response may not prove physical removal. If identical archives share a digest/object, deleting one attempt also needs a decision about remaining references and access.

For the conditional S3 implementation, compliance-mode Object Lock prevents deletion of a protected version during retention; an ordinary delete can instead add a delete marker while retaining the version. Governance mode permits privileged exceptions. These are materially different choices from application-level no-overwrite. [AWS Object Lock documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html).

**Required change:** Select an immutability/deletion model before collecting archives. Prefer immutable revision identity with a separately authorized deletion path if prompt deletion is promised, or disclose an unavoidable retention interval. Specify all-version deletion, backup handling, reference counting where applicable, and what remains of signed evidence after the underlying archive is removed. An achievement whose source is deleted must not claim its bytes are still retrievable.

**Acceptance:** Delete an accepted attempt before the normal retention date and verify the documented result at the storage-version level, not just in the UI. Cover two attempts with the same digest and a retained signed receipt. Neither another participant's evidence nor the deletion promise is silently invalidated.

### P5-05 — Safe archive extraction does not prevent unnecessary disclosure

**Evidence:** `assess submit` packages the scratch-directory submission. Intake rejects unsafe paths and enforces size limits. The archive goes to remote storage and human scorers. No allowlist of submitted files, payload preview, or treatment of incidental files is defined.

**Failure scenario:** A learner copies a project fragment into scratch and leaves `.env`, notebook output, editor history, or Git author metadata beside the solution. The archive is structurally safe but sends those files to the service. Identifying metadata can also compromise scorer blinding despite pseudonymous attempt IDs.

**Required change:** Define a task-specific submission manifest and minimal payload. Exclude incidental files by default, show the exact included paths before upload, and require a deliberate decision for any extra data needed by the task. Separate administrative/identity metadata from grader-visible material. Screening can warn about likely secrets, but must not be presented as guaranteed redaction.

**Acceptance:** Seed scratch with a valid solution plus a private env file, identifying metadata, generated logs, and a required task fixture. Only the declared task material is included after review; the fixture still works. Scorers receive the pseudonymous payload without routine account or Git metadata.

**Distinct from P3-09:** Archive traversal and execution isolation now have checks. This concerns disclosure through a valid archive before or during grading.

### P5-06 — The graded environment must be part of the task contract

**Evidence:** Results now record the grader image digest, public tests run locally, hidden tests run in the grading sandbox, and study tasks run in a hosted workspace with restricted networking. The plan does not bind task versions to the language/runtime/dependency environment used in each location.

**Failure scenario:** A locally valid solution uses a supported library or language feature, but the grader image contains an older version. A hosted workspace lacks a dependency and cannot fetch it because of its network policy. The failure is recorded as weak independent performance rather than an environment mismatch. Recording the image digest after the fact makes this reproducible without making it a fair assessment.

**Required change:** Publish a task environment contract: runtime/dependency versions, preinstalled resources, test invocation, relevant limits, and any nondeterministic inputs. Align local/public-test, hosted-workspace, and hidden-test environments or disclose their intentional differences in the task. Classify infrastructure/provisioning failures separately from learner errors and specify retry/finalization handling. Preserve the same setup in both study arms.

**Acceptance:** Run a fixture using each declared feature and dependency through all three environments. A mismatched image or missing provisioned dependency produces an infrastructure status rather than a finalized learning failure. Retrying infrastructure work preserves the accepted submission revision and does not require a newly exposed assessment task unnecessarily.

## Recommended next gate

Resolve pre-execution verification before public distribution. Before the pilot, prove recovery/re-enable transitions, single-response authorization consumption, minimal upload packaging, and the archive deletion path. Bind task environment versions before interpreting scores. Keep the round-4 fixes, but clarify the residual task-change acceptance promise rather than silently treating an advisory inference as a controlled transition.

Review limits: this is a design review, not an audit of implemented jrdev services or hooks. No real submissions were uploaded and no plugin was installed. Technical lifecycle and conditional storage claims were checked against primary Claude Code and AWS documentation. Proposed acceptance cases remain future checks.
