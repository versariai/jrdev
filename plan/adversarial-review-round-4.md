# Adversarial review of the jrdev.ai plan — round 4

Reviewed: 2026-10-01. Scope: [the current plan](jrdev-ai-plan.md), checked against [round 1](adversarial-review.md), [round 2](adversarial-review-round-2.md), and [round 3](adversarial-review-round-3.md). The plan and previous reports were not edited.

Source SHA-256: `b181513fca7b4c60392420fc9294890cc81d5c2820dffade8df62073d3e2807e`.

## Verdict

The revision gives substantive design responses to all nine round-3 findings: explicit compatibility and migration rules, a staged help oracle, an FSRS item contract, narrow achievements, staged breakpoint checks, consent/deletion handling, deduplication, configuration tracking, and isolated grading. Those responses are not implementation evidence, but the earlier omissions should not be reported as unchanged.

This round finds **2 high-priority and 5 medium-priority issues** in how those mechanisms interact. High means a central evaluation or evidence-integrity claim can fail under the specified design. Medium means the affected mechanism needs a clearer boundary or lifecycle before shipping. Failure scenarios below are design inferences unless explicitly identified as verified behavior.

| ID | Priority | Finding | Plan location |
|---|---|---|---|
| P4-01 | High | Assessment assistance detection differs between study arms | §5.5, §8.1–8.3 |
| P4-02 | High | Finalized evidence is not bound to an immutable submission and authorized score | §5.5, §6.3 |
| P4-03 | Medium | Help authorization is session-scoped but should be task-scoped | §5.4, §8.2 |
| P4-04 | Medium | Ignoring new control fields does not preserve policy semantics | §5.3, §5.6 |
| P4-05 | Medium | Added-line scanning and private marker registration leave coverage gaps | §5.4 |
| P4-06 | Medium | Project identity and multi-root context boundaries are undefined | §5.2, §6.1–6.2 |
| P4-07 | Medium | Restoring a snapshot can forget task exposure | §5.5–5.6 |

## Findings

### P4-01 — Arm-specific monitoring can change which outcomes count

**Evidence:** Phase 1 assessments rely on a no-help declaration plus automatic coached classification when a **jrdev-hooked** Claude Code session uses tools in the assessment directory. The efficacy comparator is the same AI tool **without jrdev**. The primary outcome uses assessed-transfer tasks, while coached attempts cannot produce the same qualifying independent evidence.

**Failure scenario:** Two participants use Claude Code to inspect their assessment files and both report no help. The intervention participant is automatically classified as coached; the comparator has no jrdev hook and remains apparently eligible. Even a benign tool call, such as checking a filename, can create this asymmetry because the rule flags any tool use. Whether a score is excluded, reclassified, or treated as missing can now depend on assignment rather than performance.

**Impact:** Randomization and blinded scoring do not fix unequal assessment ascertainment. The comparison can favor either arm depending on how contaminated attempts are analyzed. This is a different problem from the previous tutoring-context isolation finding.

**Required change:** Specify the same assessment protocol and assistance ascertainment for both arms, independent of the tutoring intervention. Separate the scored outcome from its assistance/validity flags and define their treatment in the primary analysis. Avoid adding a monitoring component to the comparator that itself tutors or changes practice. If monitoring remains asymmetric, disclose it and do not treat flags as equivalent evidence of independence across arms.

**Acceptance:** Equivalent tool use, self-reported help, and missing monitoring data receive the same assessment treatment in both arms. An arm-blind analyst can determine scoring and validity without knowing whether the participant installed the tutoring plugin. All randomized participants remain accounted for, including coached or invalidated attempts.

### P4-02 — A score needs a stable artifact and an authenticated finalization

**Evidence:** `assess submit` packages files and uploads via a pre-signed URL. A scorer later finalizes the result. The evidence schema records task/rubric versions and scorer provenance, but not a submission digest, immutable storage version, attempt revision, or the authority/check used to import a finalized result.

**Failure scenario:** The upload object is replaced after grading but before a mentor opens its evidence reference. The score now appears to certify different code. An upload retry can also race with scoring. Separately, local assessment files and summaries could be edited to claim finalization unless imported results are validated against an authoritative grading record; protected control paths currently cover session/config/exception state rather than score provenance.

For one possible storage implementation, Amazon S3 explicitly allows repeated use of a pre-signed URL until expiry and replacement of an existing object at the same key. A pre-signed upload is not inherently one-time or immutable. This is a conditional example, not an assumption that jrdev has selected S3. [AWS pre-signed URL documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html).

**Impact:** Human grading and immutable evidence categories cannot establish provenance if the submitted bytes or finalization authority can change underneath the record. The local progress report should not imply verification simply because a JSON field says `finalized`.

**Required change:** Assign an attempt and submission revision; bind the received archive digest, storage identity, task/rubric versions, test environment, and result. Make accepted submissions immutable or explicitly versioned. Define duplicate-upload and grading-retry behavior. Only an authorized scorer/service may finalize; local clients verify an authenticated receipt or authoritative result before displaying verified evidence. Preserve pseudonymous identity mapping separately so this does not undermine scorer blinding.

**Acceptance:** Reusing an upload URL, changing files after submission, uploading twice, and attempting to import a fabricated finalized record cannot silently change certified evidence. Every displayed achievement resolves to the exact graded bytes and authorized result. Repeated callbacks cannot produce conflicting finalizations.

### P4-03 — Stage 4 can authorize a different task accidentally

**Evidence:** Hooks store `help_stage` in session state, advance it per user `/stuck`, and reset it “when the task changes.” The plan defines no task identifier, task-change command/event, or authority that decides the task boundary. The one-call edit exception is file-scoped, but the response oracle is stage-scoped.

**Failure scenario:** A learner reaches stage 4 for task A, then asks a new question about task B in the same conversation. A hook sees ordinary prompt text rather than a declared task transition, leaving stage 4 active. The tutor supplies a complete B solution and the oracle treats it as authorized. A file-scoped edit exception does not prevent answer disclosure in chat.

**Required change:** Bind help authorization to a task identity and define explicit creation/reset/completion transitions. Decide whether an ambiguous new request resets assistance conservatively or asks the learner to choose the task. Treat any model classification of task changes as advisory unless tested and disclosed. Keep stage progression and write exceptions independently scoped to the same task.

**Acceptance:** Reach stage 4 on A, switch to B in the same file, and then switch to B in another file. B starts at its defined stage and cannot inherit authorization merely because the session persists. Resume and compaction preserve the correct task binding.

### P4-04 — Additive fields can still change what an old reader should do

**Evidence:** §5.6 says control state is additive-only, old readers ignore unknown fields, and a newer file therefore never makes an older release deny work. §5.3 uses control state to determine edit permission and recovery.

**Failure scenario:** A later policy adds an optional restriction that removes an allowlist exemption. An older reader ignores that field and allows an edit the new policy would deny. Both versions parse the file successfully, but their effective rules disagree. A field being syntactically optional does not make its semantics optional.

**Required change:** Define which additive fields are safe to ignore and which require a minimum policy-engine capability. Store the applicable policy semantics/version independently of learner-record schemas. For unsupported enforcement semantics, either implement a safe degraded policy or clearly suspend the unsupported policy with a notice; do not claim seamless enforcement compatibility. Allow recovery to remain runtime-independent without promising that all future policies are backward-compatible.

**Acceptance:** An old reader encounters a synthetic new restrictive field. It cannot report the new policy as active while applying the older permissive rule. Concurrent versions show their effective capability and policy identity consistently. Purely informational new fields still remain harmless.

**Distinct from P3-01:** Backup and schema migration now exist; this finding concerns policy meaning despite successful parsing.

### P4-05 — A staged diff is not the complete staged program

**Evidence:** The new pre-commit design scans **added lines** in a cached diff and blocks only marker IDs registered in private session state. Unregistered markers merely warn. The broader claim is that commits containing jrdev breakpoint markers are blocked.

**Failure scenario:** A previously present marker survives in unchanged lines while another part of the file changes, or the marked file is renamed without content changes. Added-line scanning sees no marker. On another machine or after session-record cleanup, an active inserted marker loses its local registration and becomes warning-only. Documentation text and real breakpoint code are being distinguished partly by local history rather than by staged content.

**Verified behavior:** In a disposable Git repository, a staged pure rename retained `jrdev-bp:registered-id` in the staged blob while the specified cached diff contained **zero added content lines**. No jrdev implementation was executed. Git documents rename metadata separately from content hunks. [Git diff documentation](https://git-scm.com/docs/git-diff).

**Required change:** State whether the hook prevents newly added markers or all active markers in the proposed commit. For the latter, inspect the staged blobs within the declared scan scope, not only additions. Define how active breakpoint syntax is recognized without depending solely on ephemeral local registration. Preserve the intentional distinction between documentation examples and executable breakpoint insertions.

**Acceptance:** Cover a pure rename, an unchanged marker in a modified file, a clone without session history, and deleted registration. Each produces the documented decision. Documentation examples remain distinguishable without granting every unknown active marker a warning-only exemption.

### P4-06 — A repository hash is not yet a project boundary

**Evidence:** Private records are keyed by `<repo-hash>`, and project-specific data must not cross projects. The hash input and identity lifecycle are unspecified. Hook path normalization is described for protected files, but not for selecting the project record or handling sessions that gain another repository root.

**Failure scenario:** Hashing a remote URL makes two clones share records unexpectedly; hashing a raw path makes aliases diverge or reuses records when a different checkout replaces a directory. A monorepo or multi-root session also needs an explicit decision about what constitutes one project. Adding project B to a conversation that already contains A's injected goals cannot erase that context.

Claude Code supports adding directories during a session. DirectoryAdded occurs after the addition and cannot block it, so an after-the-fact hook is not itself a preventive context boundary. [Claude Code hooks reference](https://code.claude.com/docs/en/hooks).

**Required change:** Define canonical project identity, alias/worktree/clone behavior, directory reuse, and non-Git projects. Specify multi-root support or require a fresh session before injecting another project's private context. Narrow the privacy promise to what jrdev controls: its selected storage and injection, rather than all information already present in an AI conversation.

**Acceptance:** Exercise symlink aliases, two clones, linked worktrees, replacement of a checkout at the same path, and adding another repository mid-session. Each has a declared identity and injection policy. Project-specific goals are not silently selected from the wrong record, and context already disclosed is not represented as removable by changing the current directory.

### P4-07 — Restoring old data must not make a previously seen task fresh

**Evidence:** Task novelty depends on a per-learner exposure log in assessment records. Migration backups and `jrdev restore <timestamp>` restore saved learner data. The plan does not separate restorable profile data from historical exposure facts.

**Failure scenario:** A learner snapshots their profile, sees task T, then restores the earlier snapshot while rolling back a release. If the exposure log is restored too, T can be selected as a fresh assessment even though the learner has already seen it. The same issue can arise when data is deleted or rebuilt: absence of a log is not evidence of no exposure.

**Required change:** Preserve exposure history across ordinary rollback or mark freshness as unknown when history is lost. Define reset/restore consequences separately for convenience data, evidence, and exposure facts. If an authoritative study-side exposure ledger is used, disclose its retention and deletion scope; do not silently retain data contrary to a deletion promise. Permit practice on repeated tasks without calling it fresh transfer evidence.

**Acceptance:** Take a snapshot, expose T, restore, and request a new assessment. T is excluded or labeled previously exposed/unknown. A reset or missing exposure log cannot establish protocol-controlled freshness. Study reports distinguish controlled selection from unverifiable exposure history.

## Recommended next checks

Before the pilot, define submission identity and finalization authority, and bind help stages to tasks. Before the efficacy study, make assessment ascertainment symmetric across arms. Before compatibility claims and breakpoint distribution, test policy semantics and staged-blob coverage rather than parsing success or happy-path diffs alone. Define project identity and restoration behavior before relying on them for privacy or freshness guarantees.

Review limits: the Git rename fixture was run in a temporary repository and removed automatically. The report does not claim that any proposed jrdev hooks, grading service, or study protocol have been implemented or tested. Primary documentation was checked for Git, Claude Code, and the conditional S3 upload example. Other failure scenarios are grounded in the plan's specified mechanisms and omissions.
