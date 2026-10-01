# Adversarial review of the jrdev.ai plan — round 2

Reviewed: 2026-10-01. Scope: the revised [product plan](jrdev-ai-plan.md), following [round 1](adversarial-review.md). The plan and previous report were not edited.

Reviewed source SHA-256: `e0385dd645c660186438176341d9df003fdb3d16d58ed04394640f07e0b84896`.

## Verdict

The revision addresses several first-round problems: production checks are now warnings, coached practice is separated from assessment, Stop reminders are bounded, the pilot uses parallel allocation, and coverage is no longer called an ordered path. Those improvements should be retained.

The remaining weaknesses lie in interactions between the new mechanisms. A single mode selector cannot clearly deliver the proposed Learning + Typing combination; assessment conditions are described as controlled without a defined isolation protocol; and recovery, state protection, and one-use exceptions still lack consistent operational rules.

This round identifies **2 high-priority and 7 medium-priority findings**. High findings threaten the core enforcement or independent-learning evidence claim. Medium findings require a decision before the affected feature ships. These are findings about the written design, not reproduced implementation defects. “Addressed” below means addressed in the text, not verified in software.

## First-round disposition

| Previous finding | Disposition in the revised plan |
|---|---|
| P-01 Production exclusion | Original absolute claim addressed by the explicit warnings and residual-risk disclosure. The detection scope still needs implementation tests. |
| P-02 Unaided evidence | Partially addressed by evidence categories. Assessment isolation and scorer authority remain unresolved; see P2-02. |
| P-03 Privacy flows | Partially addressed by the inventory and profile split. User-entered global fields and repository persistence need additional controls; see P2-06/P2-07. |
| P-04 Stop workflow gates | Original forced-human-input claim addressed by pending states and bounded reminders. |
| P-05 State and exceptions | Partially addressed by session keys and precedence. Recovery and concurrent exception consumption remain unresolved; see P2-04/P2-05. |
| P-06 Validation and roadmap | Early formative evaluation added. Proceed/revise thresholds remain discretionary; see P2-09. |
| P-07 Pilot comparison | Original order/cohort problem addressed by parallel allocation within one source. Missing-outcome handling still needs specification; see P2-09. |
| P-08 Coverage and maps | Original representation problem addressed by revised labels and explicit unresolved relationships. |
| P-09 Installation trust | Partially addressed by SHA identity and comparison instructions. Verification coverage and independent trust remain incomplete; see P2-08. |
| P-10 Commands | Canonical names and installation tests added. The trusted command execution route remains an explicit prototype question; see P2-03. |

## Findings

| ID | Priority | Finding | Plan evidence |
|---|---|---|---|
| P2-01 | High | Workflow selection and edit enforcement are conflated | §5.1–5.2, §5.4, §8.1 |
| P2-02 | High | Assessed transfer lacks separation from tutoring and scoring | §5.3, §5.5, §6, §9 |
| P2-03 | Medium | The trusted state-writing route conflicts with state protection | §5.1–5.2 |
| P2-04 | Medium | Deleting session state is not a reliable mode-off recovery | §5.2, §5.6 |
| P2-05 | Medium | Atomic file replacement does not make an exception single-use | §5.2, §5.4, §5.6 |
| P2-06 | Medium | Global fields can carry project details across projects | §5.4, §6 |
| P2-07 | Medium | Consent to ignoring files has no safe refusal or tracked-file path | §5.1, §6 |
| P2-08 | Medium | Self-reported verification covers only part of the installed behavior | §5.6, §7.4, §10 |
| P2-09 | Medium | Phase gates and study missing outcomes remain undefined | §8.1, §9 |

### P2-01 — A workflow switch can silently remove edit denial

**Evidence:** `/jrdev:mode` sets one session mode from `learn|type|debug|map|test|off`. The decision table allows edits whenever mode is not `type`. Debug mode describes breakpoints placed by the learner “in Typing mode,” and the efficacy arm is “Learning + Typing.” The plan does not say whether profiles persist independently of mode or whether workflows can coexist with Typing enforcement.

**Failure scenario:** A learner enables Typing, then selects Debug to receive guided debugging. The state becomes `debug`, so the explicit first row now allows an Edit that Typing would deny. Likewise, selecting Learning can remove denial while the UI or study description still implies the combined intervention.

**Impact:** Two central features cannot be composed unambiguously. A workflow change can alter permissions rather than just teaching behavior, and the actual intervention may differ from the one being evaluated.

**Required change:** Separate workflow selection from edit policy, or explicitly specify the supported combined states. Define which learning/profile features run in every workflow, what `off` disables, and when the user is informed that enforcement changes. If combinations are intentionally unsupported, update the Debug instructions and study treatment accordingly.

**Acceptance:** Start with edit denial enabled, enter Learning and Debug, and inspect the next Edit decision. It must match an explicit composition table and the status display. A study participant's recorded configuration must demonstrate the advertised Learning + Typing treatment.

### P2-02 — The assessor can inherit the tutor's answers and instructions

**Evidence:** §5.5 says the agent is restricted to task presentation and scoring, with fresh tasks, tests, and rubric marked as controlled. No assessment execution boundary, submission transition, or task-exposure record is defined. §5.3 stores assessments as pending across sessions. §6 describes provider scoring, while Phase 1 says the task bank is scored by hand.

**Failure scenario:** `/jrdev:assess` runs in a conversation containing the coached solution and a standing instruction to provide pseudocode after denial. The model volunteers a useful hint without an explicit help request. A resumed pending assessment then receives ordinary tutoring. The record qualifies for demonstrated level because the rule only explicitly catches help requests or `/stuck`. Separately, a learner's answer can contain instructions to award a pass if the scorer treats the submission as instructions rather than data.

Claude Code documents that invoked skill instructions remain in conversation context across turns. Loading another skill does not itself establish a clean assessment context. [Skills documentation](https://code.claude.com/docs/en/skills).

**Impact:** The new categories are useful, but the “assessed transfer” category can still certify assisted work or an unreliable grade. A delayed attempt also needs a fresh application of the skill; the delay alone does not establish task novelty.

**Required change:** Define an assessment lifecycle: task selection and exposure tracking, isolated presentation, allowed resources, submission, grading, and finalized evidence. Relabel attempts for unsolicited help as well as requested help. Specify interruption, restart, and abandonment behavior. Keep reference answers and assessor instructions out of learner-facing context. Choose human scoring for Phase 1 consistently across the command and data-flow inventory; require scorer validation before automated grades can raise a demonstrated level.

**Acceptance:** Test a conversation with the worked solution, unsolicited hints, resume after interruption, and an answer containing grading instructions. None may silently produce a qualifying independent record. A blinded human checks the rubric and the evidence category before a Phase 1 level changes. Reports distinguish protocol-controlled conditions from model-requested behavior.

### P2-03 — User-only skills do not define a trusted state-write channel

**Evidence:** §5.1 permits either a dynamic-context command or a UserPromptSubmit hook to mutate state, while denying agent Edit/Write/Bash operations targeting state files. §5.2 separately hard-allows `.jrdev/` scaffolds “written by jrdev scripts.” A path rule does not identify the caller, and the precedence between scaffold allowance and state protection is absent.

**Failure scenario:** Permission settings reject the skill's injected command, so the user invokes mode-off but no state changes. Another implementation broadly allows `jrdev` script calls to make commands work, allowing an ordinary agent Bash call to invoke the same state writer. An overly broad scaffold allowance can also override control-file protection.

Claude Code's documentation says injected commands undergo permission checks and can abort skill rendering. `disable-model-invocation` controls skill invocation; it is not a filesystem privilege for the underlying script. [Skills documentation](https://code.claude.com/docs/en/skills).

**Required change:** Make the Phase 0 prototype resolve the actual invocation route under supported permission settings, rather than just argument passing. Specify trusted hook/script writes separately from model tool calls, normalize protected paths, and give control-file protection explicit precedence over scaffold allowances. Keep the guarantee scoped to tested tool paths rather than claiming the model cannot ever switch enforcement off.

**Acceptance:** A user command changes state once; equivalent model Edit, Write, and direct Bash-script attempts do not receive an unintended exemption. A permission-rejected command reports failure and leaves state unchanged. Cover paths containing spaces, aliases to protected files, and the hard-allowlist conflict.

### P2-04 — Recovery can reinstate the policy that caused the denial

**Evidence:** §5.2 lists deleting `.jrdev/sessions/` as recovery, although project and user defaults apply when session state is absent. Malformed state denies edits. The shell CLI is described as working without the agent, but its runtime independence is not specified.

**Failure scenario:** Project defaults select `type`. Deleting the session override restores that default rather than turning Typing off. If the malformed file is project config, deleting sessions changes nothing. Deleting the whole directory also affects other sessions despite the session-isolation promise. An environment variable set in a separate terminal does not automatically change a running process's inherited environment.

**Required change:** Distinguish resetting session state from disabling policy. Define a recovery command that targets one session and overrides invalid lower layers deliberately, or document a restart-based recovery with exact scope. State how the hook observes a changed disable setting, what happens to pending records, and which recovery path works when Node/Python is unavailable. Never present deleting all sessions as session-local recovery.

**Acceptance:** Recover from project-default Typing, malformed project config, malformed user config, and a missing runtime while a second session remains active. The chosen session permits ordinary work after the documented steps; the second session's state and pending records are preserved.

### P2-05 — A one-edit exception needs transactional consumption

**Evidence:** §5.4 grants a session-and-file exception for one edit or 15 minutes. §5.2 promises atomic file writes through temporary-file rename. §5.6 lists PreToolUse but no successful-write/failure reconciliation mechanism.

**Failure scenario:** Two tool decisions read the same unconsumed exception before either writes the new state, and both edits pass. Alternatively, consuming it at PreToolUse spends the exception on an Edit that later fails or is denied by another policy. One multi-edit call may also change more than the intended scope.

**Impact:** Atomic replacement prevents partial files; it does not define a read-modify-write transaction or whether “one edit” means one approval, one tool call, or one successful mutation.

**Required change:** Define the consumption unit and concurrent reservation semantics. Bind approval to a tool-use identity and canonical file scope. Choose either an explicitly documented one-approved-call policy or reconcile success/failure with a reservation lifecycle. Handle expiry, interruption, and process death without granting a second use or leaving an indefinite exception.

Claude Code exposes separate post-tool success/failure events and documents that permission-denied calls do not fire PostToolUseFailure. Any reconciliation scheme must account for those missing callbacks. [Hooks reference](https://code.claude.com/docs/en/hooks).

**Acceptance:** Simultaneous requests cannot both consume one authorization. Test execution failure, permission denial, interruption, a multi-edit request, and expiry during execution. The log explains whether authorization was used, released, or expired, and the prior policy resumes consistently.

### P2-06 — A global goal title is still an unrestricted disclosure channel

**Evidence:** §6 declares global data free of project details. Setup captures a free-form mission and goals, and Learning injects goal titles and topic names into every project. Assessment summaries are also stored globally without a field-level summary schema.

**Failure scenario:** The learner enters “understand Acme's fraud override for customer X” as a goal title. No project log is read, yet the global title is injected into a different employer's project. The proposed test with a sensitive string only in project A's log passes while this route remains open.

**Required change:** Separate general topic identifiers from private goal descriptions. Preview the exact globally injected fields, offer project-scoped goals, and use an explicit minimal schema for global assessment summaries. Do not assume that fields named “topic” or “title” cannot contain confidential information. Explain what the plugin controls and what remains a user responsibility.

**Acceptance:** Place a sensitive identifier in a mission, goal title, and assessment summary—not only a project log. None reaches another project without an explicit decision to make that field global. The session disclosure shows the actual injected content.

### P2-07 — Declining `.gitignore` changes leaves persistence undefined

**Evidence:** Setup offers to add `.jrdev/` to `.gitignore`; §6 then describes project records as gitignored with consent. It does not specify behavior when consent is declined or when the directory already contains tracked files. That directory also contains project defaults that teams may legitimately want to share.

**Failure scenario:** A learner declines repository modification. The plugin still writes debug notes and assessments there, and a routine `git add .` stages them. In an existing repository, a previously tracked log remains tracked after an ignore rule is added. Git explicitly documents that ignore rules do not affect already tracked files. [Git ignore documentation](https://git-scm.com/docs/gitignore).

**Required change:** Choose a safe persistence fallback outside the worktree when exclusion is declined or cannot be verified. Before writing sensitive records, distinguish tracked configuration from private logs/sessions and check the actual tracking/ignore status. Do not automatically untrack existing files; explain the situation and retain control with the user. Separate shareable defaults from private records in the layout or ignore rules.

**Acceptance:** Test consent refusal, an already tracked `.jrdev` log, and a repository sharing its project config. Ordinary staging does not newly include private records, and team defaults can still be versioned intentionally.

### P2-08 — Verification is incomplete and depends on the installation it verifies

**Evidence:** §7.4 has the bundled verifier print source identity and hashes of hooks/handlers, compared against a release page. §5.6 also contains skills, libraries, output styles, manifests, and vendor dependencies. §10 presents this as a tampering mitigation.

**Failure scenario:** A skill or imported library is changed while hook files remain identical. The stated comparison succeeds even though behavior changes. A modified bundled verifier can also print expected values rather than measure files. A SHA in metadata identifies intended source; it does not by itself prove the contents of a writable installation.

**Required change:** Distinguish version reporting from integrity verification. Publish an authenticated manifest covering every behavior-affecting distributed file, including dependencies, and document an independent comparison procedure using trusted tooling. State how users establish trust in the published manifest and signing identity. Keep the bundled verifier as a convenience with its trust limitation disclosed; do not execute it as the sole check of a suspect installation.

**Acceptance:** Tampering with a skill, library, dependency, and the verifier itself is detected by the independent procedure. A copied source-SHA label cannot make altered contents pass. The documented process works for the installed marketplace cache layout without requiring embedded Git metadata.

### P2-09 — Evaluation gates can be decided after seeing favorable results

**Evidence:** Phase 1 proceeds when “most” finish with “acceptable” burden, revises when mode-off is “high,” and requires no unresolved leakage “pattern.” No thresholds, denominators, or decision owner are specified. The efficacy analysis names intention-to-treat and dropout reporting but does not specify missing delayed assessments or sample-size feasibility.

**Failure scenario:** Completion is calculated among learners who successfully installed rather than everyone enrolled; repeated accidental answer disclosure is dismissed as not a pattern; support burden excludes staff helping through chat. Participants with missing post-tests are simply omitted while the report is called intention-to-treat. The team can pass the phase by changing interpretations after observing results.

**Required change:** Before recruitment, define operational measures, denominators, support-time accounting, leakage severity/action rules, and proceed/revise/stop thresholds. Assign the decision owner. For the efficacy study, specify a primary outcome, recruitment/sample-size or precision rationale, a missing-outcome strategy with sensitivity checks, and how outcome scoring stays separate from the intervention. Retain flexibility for a small formative pilot, but record deviations and reasons instead of retrospectively redefining success.

**Acceptance:** Apply the decision rules to invented results containing setup failures, dropouts, one serious leakage incident, and heavy staff assistance. The outcome is determined without reinterpreting terms. The preregistered study states how all randomized participants are accounted for when delayed outcomes are missing, and confirms that the study fits the capacity budget.

## Recommended next gate

Before expanding the prototype, write a combined workflow/edit-policy table and an assessment lifecycle. Then test command execution, recovery, and exception consumption together from a clean install. These interactions are more consequential than adding another mode or improving the website.

Retain the first-round corrections. Resolve the privacy persistence and verification details before public pilot distribution, and define the formative decision rules before recruitment. Open choices such as business model and first stack do not require closure merely to revise the design, but must be settled where they affect these implementation and study protocols.

Limitations: no plugin implementation was executed, no participant data was collected, and no learning benefit was measured. Technical claims about Claude Code and Git were checked against their primary documentation; version-specific behavior must still be verified against the versions the project elects to support.
