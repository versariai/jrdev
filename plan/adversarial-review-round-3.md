# Adversarial review of the jrdev.ai plan — round 3

Reviewed: 2026-10-01. Scope: [the product plan](jrdev-ai-plan.md), with [round 1](adversarial-review.md) and [round 2](adversarial-review-round-2.md) used to avoid repeating findings.

Source SHA-256: `ca066dbefbf400fd509b8dba8285bf0406d5c3420caecee9d72ee268edb3108a`.

## Verdict and previous findings

The plan changed during this review. This report was reconciled against the newer source identified above. It now separates edit policy, workflow, and learning; moves Phase 1 assessments outside the tutor; defines a command channel, recovery overrides, and locked exception consumption; moves private records outside repositories; limits global injection; specifies independent integrity verification; and adds explicit pilot decision rules. Those changes substantially address all nine round-2 findings **at the design level**, with implementation verification still pending. The previous mode-composition and tutor-context findings are not reported as unchanged blockers.

This round adds **1 high-priority and 8 medium-priority findings**, concentrating on grading isolation, saved-data compatibility, learning records, operational review, and feedback lifecycle. High identifies an unspecified execution boundary for untrusted assessment submissions. Medium means a material design decision is needed before the affected feature ships. None is claimed to be a reproduced vulnerability in implemented software.

| ID | Finding | Plan location | Required before |
|---|---|---|---|
| P3-01 | Source rollback does not roll back saved-data schemas | §5.3, §7.4 | Public updates |
| P3-02 | Tutor review does not distinguish prohibited leakage from authorized help | Principles 2/7, §5.4, §8–9 | Phase 1 pilot |
| P3-03 | FSRS has no defined review item or rating contract | §5.4, §6, §9 | Phase 2 reviews |
| P3-04 | A qualifying assessment has no defined scope of demonstrated ability | §5.5, §6, §8 | Progress levels |
| P3-05 | Breakpoint checks need to inspect committed content and preserve existing hooks | §5.4 | Optional hook distribution |
| P3-06 | Consent and deletion do not extend through the feedback processing lifecycle | §3.3, §6 | Feedback collection/processing |
| P3-07 | Feedback frequency is vulnerable to duplicates and organized submissions | §3.3, §9 | Monthly prioritization |
| P3-08 | Model and intervention changes can invalidate behavior reviews and study comparability | Principle 7, §7.4, §8.1 | Regression claims and efficacy study |
| P3-09 | Hidden-test grading introduces unspecified execution of untrusted submissions (**High**) | §5.5, §6.3, §9 | Phase 1 assessments |

## Findings

### P3-01 — Package rollback leaves newer data behind

**Evidence:** State files have schema versions; unknown schemas deny edits. Updates and rollback are described as changing the marketplace SHA, with bundled runtime checks. No persistent-data migration or backward-compatibility policy is specified.

**Failure scenario:** Version B migrates a profile or session file written by version A. A user rolls the package back to A after a regression. A now sees an unknown schema, denies work, or cannot display existing evidence. Two installed versions accessing the same global profile can produce the same incompatibility without an intentional rollback.

**Consequence:** The release acceptance test can pass for clean installs while the documented recovery fails for returning users. Rolling back executable code is not the same as restoring compatible learner data.

**Required change:** Define the supported read/write compatibility range, migration owner, backup/restore policy, and behavior when two versions share a profile. Separate schema upgrades from ordinary session-start work; do not silently reinterpret evidence categories. Decide whether rollback uses compatible data, a migration reversal, or an explicitly reviewed restore. Preserve learner records rather than treating reset as the default fix.

**Acceptance:** Run A → B → A using populated goals, pending recaps, assessments, and review history. Interrupt migration and exercise two versions concurrently. The documented result preserves records and gives a usable recovery path; a schema change cannot silently upgrade coached work into demonstrated evidence.

**Distinct from P2-04:** That finding concerns recovering from restrictive defaults or malformed state. This one concerns data compatibility across legitimate releases.

### P3-02 — The behavior-review oracle can reject the escape hatch or accept accidental solutions

**Evidence:** Typing escalates through hints, pseudocode, partial snippets, and finally a worked solution. The new Phase 1 leakage definition is a complete solution while `edit:type` is on **without `/stuck`**. It does not distinguish the first hint request from authorization of the final solution stage.

**Failure scenario:** The learner invokes `/stuck` once to request a hint. The tutor returns the complete solution. Because `/stuck` occurred, the literal serious-leakage definition does not catch it, even though the escalation workflow has skipped its intermediate stages. Conversely, an evaluation that ignores escalation state can reject the final, properly authorized worked solution.

**Consequence:** The release gate has no stable oracle. It can make the escape hatch unusable or certify the very behavior the tutoring product aims to prevent.

**Required change:** Define expected behavior by workflow, help stage, task, and learner authorization. Separate unsolicited solution disclosure from authorized escalation. Include correctness, misleading hints, and inappropriate refusal in the checklist as well as solution leakage. Record stage transitions independently of the model's description of what it did, and retain reviewed examples of both acceptable and unacceptable responses.

**Acceptance:** The same worked solution fails when volunteered at the hint stage and passes the authorization check when requested at the final stage, while remaining recorded as coached practice. Include misleading hints and a refusal to honor a valid escape request. Repeated runs reveal variation rather than one favorable transcript deciding the release.

**Distinct from P2-02/P2-09:** This defines what a tutoring review should judge, rather than assessment isolation or the numerical phase gate.

### P3-03 — A scheduling library does not define what the learner is reviewing

**Evidence:** Phase 2 introduces FSRS, but the global queue is described as topic IDs. No card/item identity, prompt version, answer criteria, rating source, or review-to-evidence mapping is specified.

**Failure scenario:** “Async programming” is treated as one scheduled item. One session asks for a definition; another asks the learner to diagnose a concurrency bug. Both update the same memory state, even though the tasks have different requirements. An agent marks an answer easy after supplying a hint, causing a long interval based on assisted recall.

The upstream ts-fsrs example schedules a card and applies a rating after the answer; the application still has to define the item and rating. [ts-fsrs documentation](https://github.com/open-spaced-repetition/ts-fsrs).

**Required change:** Specify review-item identity, learning objective, prompt/variant relationship, rubric, assistance handling, rating ownership, and stored scheduling history. Keep recall scheduling separate from independent programming assessments and demonstrated levels. State when changing an item requires preserving, resetting, or migrating its schedule.

**Acceptance:** A hint-assisted review and an independent answer produce the defined ratings and evidence categories. Replacing a definition question with a debugging exercise cannot silently reuse an unrelated schedule. Replaying recorded answers yields reproducible scheduling decisions.

### P3-04 — A valid narrow result can become an unjustifiably broad level

**Evidence:** Finalized assessed transfer can raise the demonstrated level. The global summary schema is `{topic_id, category, date, score_band}`. The plan does not define level dimensions, the mapping from a task to a topic, how much evidence warrants a broader level change, or how the summary links to task/rubric provenance.

**Failure scenario:** A learner independently passes one task about a Python exception handler. Their progress view raises “backend” or “debugging,” implying ability across frameworks, fault types, and unfamiliar repositories. A later rubric revision changes the meaning of the same displayed level while older results remain mixed into it.

**Consequence:** Even a perfectly isolated, accurately scored task can support a narrower conclusion than the interface claims. The record's category describes its conditions, not the breadth or durability of the competence demonstrated.

**Required change:** Define skill identifiers and level meanings narrowly enough to match tasks. Store task/rubric versions, scoring provenance, stack, date, and evidence references. Specify aggregation and recency rules before showing general levels. Until those rules are defensible, display concrete achievements such as “passed these tasks under these conditions” rather than inferring a broad ability rating.

**Acceptance:** One narrow task cannot raise unrelated or umbrella topics without an explicit, justified mapping. A rubric change does not silently reinterpret historical results. A reader can trace a displayed claim to the qualifying evidence and its limits.

**Distinct from P2-02:** This remains a problem even when the attempt is genuinely independent and the score is correct.

### P3-05 — Breakpoint protection must inspect the proposed commit, not just files on disk

**Evidence:** The optional pre-commit hook is said to block commits containing jrdev breakpoint markers. Installation consent is specified, but the scan target, interaction with existing hook systems, and uninstall behavior are not.

**Failure scenario:** A learner stages a breakpoint, then removes it from the working file without staging the removal. A working-tree scan passes even though the proposed commit contains the marker. The reverse case blocks a clean staged commit because an unstaged debug session contains a marker. Installing the jrdev hook may also replace a team's existing checks or write to a hooks directory Git is not using.

Git distinguishes staged changes (`git diff --cached`) from working-tree changes, and permits hook location to be configured through `core.hooksPath`. [Git diff documentation](https://git-scm.com/docs/git-diff), [Git hooks documentation](https://git-scm.com/docs/githooks).

**Required change:** Inspect the staged content that will be committed. Define marker syntax per supported language and distinguish active breakpoint insertions from documentation/test fixtures containing the marker text. Integrate with the existing hook mechanism through an explicit supported path; do not overwrite checks. Provide removal instructions that preserve pre-existing behavior.

**Acceptance:** Cover staged-marker/clean-working-file and clean-index/unstaged-marker cases, renamed files, filenames with spaces, an existing pre-commit hook, and a custom hooks directory. The relevant staged marker blocks, existing checks still run, and uninstall restores the prior setup. Keep the documented `--no-verify` limitation.

### P3-06 — A consent flag and retention number do not specify processing or deletion

**Evidence:** Free-text feedback can be anonymous, LLM processing can be declined, and feedback is deletable on request after up to 24 months. Processing includes tagging, clustering, human review, and publication of paraphrased themes. No consent version, withdrawal behavior, deletion identity, or handling of derived copies is defined.

**Failure scenario:** An item is queued for LLM clustering with permission, then the submitter withdraws before the job runs. A stale queue payload is processed anyway. Another anonymous submitter asks for deletion but has no receipt or identifier to locate the record safely. Deleting the primary DB row leaves an interview note, queue payload, or identifying theme summary behind.

**Consequence:** The disclosed opt-out and deletion controls may work only at submission time or only for one storage location. A paraphrase of a distinctive workplace incident can remain identifying even when it contains no direct quotation.

**Required change:** Specify a consent record and check it at the point of each external processing/publication action. Define withdrawal and deletion across primary data, notes, jobs, derived text, and backups, with honest limits for copies outside jrdev's control. Provide an opaque deletion receipt for anonymous submissions or narrow the promise. Review public paraphrases for identifying detail, and clarify whether they require the publication consent flag referenced in §6.

**Acceptance:** Withdraw processing permission between enqueue and execution; no external processing occurs. Delete a receipt-bearing anonymous submission and trace its queued/derived copies. A distinctive private incident cannot become public merely by changing its wording. State how already processed or published material is handled.

**Distinct from P2-06/P2-07:** Those concern local profile injection and repository files; this concerns the service-side lifecycle of submitted feedback.

### P3-07 — Raw feedback frequency is not a trustworthy demand signal

**Evidence:** Feedback arrives through anonymous forms, pulse surveys, per-skill reports, interviews, and public GitHub channels. Prioritization multiplies frequency by severity and feasibility; no counting unit, cross-channel deduplication, or abuse handling is defined.

**Failure scenario:** One enthusiastic learner reports the same issue in five channels. A group floods the anonymous form with near-identical requests. Clustering turns those submissions into a high-frequency theme, and human review confirms that the theme is intelligible without discovering that the apparent demand is duplicated. Conversely, a serious issue affecting quiet learners ranks below a frequently repeated cosmetic request.

**Required change:** Distinguish submission count from estimated affected learners and from independently corroborated observations. Use proportional spam controls and manual duplicate review without claiming perfect anonymous deduplication. Treat severity as a separately reasoned decision, not something easily cancelled by low frequency. Keep public-channel requests separate from feedback by the intended pilot audience when interpreting demand.

**Acceptance:** Inject duplicates, cross-channel copies, and a high-severity singleton into a synthetic triage batch. Duplicates cannot automatically create stronger evidence of demand; the serious singleton receives explicit consideration. The themes board shows the evidence basis and uncertainty rather than an unsupported affected-user count.

### P3-08 — Model drift and mid-study releases need an explicit protocol

**Evidence:** Behavior review must run after every model/prompt/version change. The efficacy comparison uses the same AI tool in both arms, while Phase 3 also develops new modes. The plan does not define the tested model configurations, how changes are discovered, or what happens when either arm's environment changes during the study.

**Failure scenario:** Regression transcripts are generated on one model configuration, but pilot sessions select another or use a fallback. During the efficacy study, a prompt update changes tutoring intensity, an optional mode becomes available, or one arm receives substantially more task-specific support. Participants are randomized correctly, but the treatment and comparator no longer have stable definitions.

Claude Code documents model-switch events, but also states that a one-turn fallback substitution does not fire PostModelSwitch. That event alone cannot substantiate review after every effective-model change. [Hooks reference](https://code.claude.com/docs/en/hooks).

**Required change:** Declare the configurations covered by each behavior review and record the configuration actually used where the tool exposes it. State which changes cannot be observed. Define a release/model change policy for the study, intervention fidelity measures, permitted extra modes/resources, and support provided to each arm. Separate product iteration from the preregistered efficacy period, or record deviations and prespecify their treatment in analysis. Safety-critical fixes should remain possible with documented consequences for the study.

**Acceptance:** Simulate a model switch, fallback, plugin update, and task-specific staff intervention. Each yields the specified re-review, warning, pause, or deviation record. The published study identifies the versions/configurations used and does not imply identical AI conditions merely because both groups used Claude Code.

**Distinct from P2-09:** This concerns exposure changes during observation, rather than thresholds, sample size, or missing outcomes.

### P3-09 — Moving grading outside the tutor introduces a separate execution boundary

**Priority: High. Evidence:** Phase 1 submissions are packaged, then a human scorer applies the rubric and hidden tests that are not on the learner's machine. The plan does not specify the grading host, submission format, execution isolation, resource limits, or protection of the private test bank. Human scoring removes model grading from the loop; it does not make submitted programs trusted.

**Failure scenario:** A scorer runs a submitted program with hidden tests on a normal workstation or shared runner. The program reads files available to that process, makes network calls, modifies the environment for later submissions, or hangs indefinitely. If reference answers are accessible to the execution process, the submission may also disclose them through output, undermining future assessment freshness. Deliberate misconduct is not required: ordinary learner code can exhaust memory or delete files accidentally.

**Required change:** Specify a disposable grading environment with bounded runtime/resources, restricted filesystem and network access, no ambient secrets, and separation between scorer infrastructure and submission execution. Define safe archive extraction and size limits before running anything. Decide which hidden-test information the learner receives, and how test-bank exposure is handled. Add the submission transfer, recipient, and storage details to the data-flow inventory. Do not imply that repository SHA pinning protects against participant code.

**Acceptance:** Grade controlled fixtures attempting external network access, reading a seeded host secret, writing outside the scratch area, traversing archive paths, and running forever. The defined boundary contains each attempt, terminates safely, and leaves the next submission's environment clean. Neither logs nor returned diagnostics disclose reference answers. These checks must pass before a scorer executes pilot submissions.

## Recommended order

Validate the newly specified round-2 mechanisms together in the prototype. Before the Phase 1 pilot, establish grading isolation, define the tutoring review oracle, and complete the feedback consent/deletion lifecycle. Before public updates, prove schema compatibility and rollback using populated learner data. Before Phase 2 progress features, define review items and the limited claims supported by each assessment. The pre-commit integration and efficacy configuration protocol are gates for their respective later features.

Review limits: this was a review of the revised written plan, not an implementation audit or participant study. The reviewer did not edit the plan or either earlier report. Primary documentation was checked for FSRS usage, Git behavior, and Claude Code model-switch behavior. Proposed acceptance cases are future checks, not tests represented as already passed.
