# Adversarial review — material added after round 12

Reviewed 2026-10-02. **Scope is the previously unreviewed additions**, not another checkpoint-ledger pass: assigned team/capacity and employer safeguards; early signals; C1–C6; hook performance budget; the Phase 0 interview guide; and all three C6 experiment artifacts.

Baseline: `b1ca69a` (round-12 report). The additions through `f59160b` change eight files: 416 inserted lines and 20 removed lines. Round 12's zero-new-findings result does **not** cover this material. Existing privacy, command, assessment, release, and study contracts were consulted where the additions depend on them.

**Result: 1 high, 6 medium, and 1 low finding.** The experiment should not be treated as participant-ready yet. The candidate ideas remain hypotheses; this review does not establish their learning benefit.

## Findings

| ID | Severity | Finding |
|---|---|---|
| P13-01 | High | The real-participant C6 experiment lacks an explicit readiness and feedback lifecycle |
| P13-02 | Medium | C6's current-base special case is contradicted by the following Git command |
| P13-03 | Medium | C6 lacks a safe content boundary for diffs and untracked files |
| P13-04 | Medium | Interview ordering and the Proceed rule do not establish a top problem |
| P13-05 | Medium | Three different rules can promote candidates into Phase 1 |
| P13-06 | Medium | C2 adds enforcement without extending the all-off contract |
| P13-07 | Medium | Experiment installation can overwrite an existing personal skill |
| P13-08 | Low | Hook resource targets lack a reproducible benchmark contract |

### P13-01 — High: informal experimentation is still real collection

**Evidence:** [Early signals](jrdev-ai-plan.md#early-signals-2026-10-02) explicitly says the initial feedback was collected before channel checks. [C6 README](../experiments/c6-explain-changes/README.md) describes an experiment started on 2026-10-02, gives immediate installation instructions for two employees, and collects feedback through messages to Ivan. It says participants agreed to take part, but does not route the experiment through collection-channel readiness, employer sign-off, or a defined message-feedback lifecycle. [Interview guide](phase0/interview-guide.md), by contrast, explicitly gates interviews before the first participant.

**Failure case:** the two employees follow the README while storage/deletion readiness or the employer's written safeguards are still unconfirmed. Feedback messages acquire another identifiable copy outside the defined interview store. Answers are then paraphrased into the public plan without verifying the separate publication scope or whether a distinctive workplace incident remains identifiable. Calling this informal and excluding it from pilot data does not resolve these handling questions.

**Required change:** add an experiment-specific preflight before inviting or enrolling real participants: voluntary participation and employer safeguards, approved AI use on work repositories, scoped research/processing/publication permission, feedback storage/recipients, retention, withdrawal, and deletion of message and derived copies. Register the experiment feedback channel in the inventory/readiness checks. Reconcile the already collected early feedback with those rules; until verified, keep its raw material and distinctive derived incidents out of further external processing/publication. This does not presume the participants lacked consent; the required scopes and operations are not demonstrated by “agreed to take part.”

**Acceptance:** exercise a synthetic experiment message through storage, paraphrased output, withdrawal, and deletion. Confirm the employer safeguards privately before recruitment. The README names the readiness gate and distinguishes team-only artifact testing from participant rollout. Record the disposition of existing early feedback without publishing identities.

### P13-02 — Medium: the current-base branch loses committed changes

**Evidence:** [C6 skill, step 1](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) correctly says that, when the current branch is the base, committed scope is the commits ahead of its upstream. Step 2 unconditionally recomputes `git merge-base HEAD <base>` and diffs that result against HEAD.

**Verified counterexample:** a temporary repository on `main`, with an upstream at its earlier commit and one local commit, invoked with explicit base `main`. `@{u}..HEAD` contains **one commit**. The prescribed step-2 merge base is HEAD, producing **zero commits and an empty diff**. The fixture was removed afterwards. This is command-level verification, not a model execution test.

**Required change:** make the scope branches mutually exclusive. Resolve the upstream range for the current-base case and carry it into both log and diff steps; use the merge-base range only for a distinct comparison branch. Define how local and remote forms of the same branch are recognized. Keep the documented no-upstream disclosure.

**Acceptance:** current `main` with one unpublished commit includes that commit; a feature branch includes its branch work; no-upstream current-base use reports the stated limitation. Cover staged, unstaged, and untracked changes independently.

### P13-03 — Medium: read-only does not define what may reach the model

**Evidence:** [C6 skill](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) asks for all changes, full committed/uncommitted diffs, and reading relevant untracked files. Its exclusions address binaries, lockfiles, and generated files. The [README](../experiments/c6-explain-changes/README.md) warns against sending company details in feedback, but does not give an equivalent content boundary or AI-provider disclosure for summarization itself.

**Failure case:** a tracked configuration change contains a credential, or a non-ignored untracked environment file appears in the candidate file list. Reading diffs/files makes their content available to the AI session even though no file is changed. Avoiding sensitive material in the later feedback message does not prevent this earlier disclosure.

**Required change:** disclose the summarization data flow and require the company's approved AI policy. Enumerate paths before content reads, exclude credential/secret-bearing paths by default, provide a clear scope choice for sensitive or unusually large changes, and avoid treating untracked files as automatically relevant. State the limits: path filtering is not guaranteed secret redaction, and secrets can occur in ordinary source files. Keep the feature advisory and read-only.

**Acceptance:** synthetic tracked and untracked secret fixtures are excluded before their contents enter a model-visible tool result; ordinary source changes remain covered. Show excluded paths in the scope disclosure without printing their contents. No participant repository or real credential is needed.

### P13-04 — Medium: the interview gate can pass without a top difficulty

**Evidence:** [Interview guide](phase0/interview-guide.md) places targeted probes about loops, approval, focus, comprehension, and learning before the “unprompted” difficulties question. Its Proceed rule counts any specific T1/T2/T5 instance **or** a top-three difficulty. [Overview Phase 0 gate](jrdev-ai-plan.md#5-roadmap) requires the difficulty to be a top problem.

**Failure case:** five juniors describe an occasional prompted approval or fix-loop instance, but all rank other problems above it and say it causes little difficulty. The guide yields Proceed; the overview's top-problem claim has not been established. Earlier theme probes also mean the later ranking cannot be described as wholly unprompted.

**Required change:** collect spontaneous difficulties/ranking before named theme probes or preserve a clearly separate spontaneous portion. Separate “pattern observed” from “important problem with a concrete cost.” Align the Proceed formula with the overview's top-problem requirement, or explicitly change the overview to the weaker discovery criterion. Define proportional thresholds and a minimum useful interview sample before collection.

**Acceptance:** classify a synthetic set with five prompted, low-importance instances and zero top-ranked difficulties. It must not silently prove the top-problem gate. A second set with concrete, independently ranked difficulties produces the prespecified result. The Portuguese script follows the same ordering and definitions.

### P13-05 — Medium: candidate promotion has conflicting authorities

**Evidence:** [Candidates spec](specs/03-modes-and-workflows.md#feature-candidates-from-early-signals) requires interview support and design review before scheduling, with C3 earliest Phase 2 and C4 earliest Phase 3+. The [interview synthesis rule](phase0/interview-guide.md#synthesis-and-decision-rules-set-before-the-first-interview) says C1–C6 move into Phase 1 if three juniors describe the problem and fewer than half would turn the candidate off. Only Typing/C1/C2/C6 have concept descriptions and reaction fields. [Experiment results rule](../experiments/c6-explain-changes/README.md#how-results-are-used) separately promotes C6 if it helps the two early respondents.

**Failure case:** C4 passes the interview rule and enters Phase 1 despite its stated phase restriction; unasked C3/C5 reactions are treated as zero objections; or two favorable C6 users bypass the three-junior and design-review requirements. Several candidates can also qualify simultaneously without a selection step constrained by Ivan's ten-hour week.

**Required change:** make these evidence inputs to one promotion decision. Distinguish eligibility, priority, earliest phase, design acceptance, and actual capacity allocation. Specify denominators for concepts actually presented; unasked reactions are unknown. Keep the two-person experiment formative, and require the same design/capacity gate for C6. State the reviewer/decision owner and maximum narrow Phase 1 scope.

**Acceptance:** two favorable C6 experiment users do not automatically authorize shipment. C4 cannot jump its earliest phase through the generic interview rule. Unpresented concepts remain unevaluated. A fixture where every candidate qualifies still yields a capacity-bounded, documented selection.

### P13-06 — Medium: check-ins must participate in recovery and all-off

**Evidence:** [C2](specs/03-modes-and-workflows.md#feature-candidates-from-early-signals) adds a fourth independent setting with tool denial. [Existing command contract](specs/02-commands-and-policy.md) still says `/jrdev:off` sets all three settings to off/none/open. C2's design prerequisites exempt recovery commands and name policy precedence, but do not extend all-off, status, defaults, or exception-consumption rules.

**Failure case:** after enabling check-ins, a learner invokes `/jrdev:off` and sees the existing all-off notice, while a later burst of tool calls is still denied by C2. Alternatively, an implementation consumes a stuck edit exception before the check-in policy denies the overall call. Candidate status appropriately defers implementation; these cases must be included in its required design review rather than inherited from the three-setting contract.

**Required change before adopting C2:** extend all-off and status to the fourth setting; specify trusted configuration, default scope, reset events, and counter/concurrency behavior. Evaluate the combined final decision before consuming a stuck exception. Keep kill-switch and recovery behavior reachable, and state which tool paths remain outside check-in enforcement.

**Acceptance:** check-in threshold reached → `/jrdev:off` → a new tool burst remains unrestricted by C2. Status shows effective check-in state. A check-in-denied edit does not spend an exception through a prematurely approved policy sub-decision; concurrent calls and repeated denials do not cause a continuation loop.

### P13-07 — Medium: installation must preserve an existing skill

**Evidence:** [Experiment install/uninstall commands](../experiments/c6-explain-changes/README.md#install-about-2-minutes) copy into `~/.claude/skills/explain-changes` without checking for an existing directory, and uninstall removes that directory recursively. The snippet also asks users to edit persistent project/global instructions without a restoration record.

**Failure case:** a participant already has a personal `explain-changes` skill. Copying merges or overwrites files; uninstall deletes the participant's original files as well. Removing the snippet by hand can leave ambiguous boundaries when similar instructions already exist.

**Required change:** check for a collision and stop or back up the existing installation before replacement. Prefer an experiment-specific name if possible. Mark snippet boundaries, document the original-file backup/restoration procedure, and ensure uninstall removes only experiment-owned content.

**Acceptance:** install and uninstall with an existing same-name skill and existing similar CLAUDE.md text. Original content is preserved byte-for-byte; a fresh install also uninstalls cleanly. Do not run destructive commands on participants' actual configurations to test this.

### P13-08 — Low: hook targets need a benchmark definition

**Evidence:** [Performance budget](specs/02-commands-and-policy.md#hook-performance-budget) specifies p95 latency, resident memory, a reference laptop, Linux/macOS CI, and a 20% regression threshold. It does not identify the reference environment, timing boundaries, sample count, cold-start treatment, memory measurement, or baseline policy.

**Failure case:** CI times only handler logic on a fast runner, while the learner pays Node startup, dispatch, and lock contention for each invocation. Different runs can both claim compliance without measuring the same cost. This is a validation gap, not evidence that TypeScript cannot meet the target.

**Required change:** define the runtime/hardware, whole-invocation timing boundaries, cold/warm samples, memory scope, sample size, contention fixture, and approved baseline. Check absolute budgets as well as relative regressions. Include combined enabled hooks so per-handler success does not hide aggregate tool-call overhead.

**Acceptance:** the same fixture produces comparable results on the declared reference and CI environments, and an artificial startup delay or memory increase is detected. No benchmark was executed in this review.

## Coverage and conclusions

The assigned coordinator/scorer, explicit ten-hour total, and recognition that schedules need lengthening improve the old capacity assumptions. Names can stay private. This review does not assume Ivan is a participant's manager; the readiness check must establish independence for whoever recruits/interviews employees. “Assigned” alone is not evidence that privacy, grading-host access, or employer sign-off is complete.

C1's advisory label, C3's staged proposal, C4's later phase, and C5's advisory limit are honest starting points. They remain hypotheses, not demonstrated interventions. The early signals correctly disclose n = 2, backend context, and non-pilot status. Their usefulness does not exempt the later experiment from the existing collection safeguards. Research claims were not re-audited in this focused pass.

Complete the experiment readiness and C6 scope fixes before participant rollout. Reconcile discovery/promotion rules before interviews, and complete C2's composition design before scheduling it. The hook-budget benchmark is a later prototype acceptance task. The earlier zero-finding report applies only to its recorded snapshot.

## Verification and snapshot

Read the post-round-12 diff and all listed new artifacts. Reproduced P13-02 in an isolated temporary Git repository; no workspace or participant configuration was modified. No interview, experiment rollout, model invocation, or hook benchmark was performed. The experiment's SKILL.md was reviewed as an artifact, not invoked.

SHA-256 hashes below identify the material reviewed and supporting contracts.

| Source | SHA-256 |
|---|---|
| `plan/jrdev-ai-plan.md` | `28e0f6ca6ef07090909491984e88dc6e9684647512caa406fa853f9bc92d8867` |
| `plan/specs/01-website-and-feedback.md` | `35a667925bf41b597abdf5fbdde66008bc721e8372c6f9b9045d5bfd9ccd0a80` |
| `plan/specs/02-commands-and-policy.md` | `ddfbd11585bac42b8464c71b36ac86ae3095c1377f003f51218b5e162e3dc034` |
| `plan/specs/03-modes-and-workflows.md` | `4a39433d323425efabee7982b17d1fec5c27cf8c777a5c9d692a50ed36db5ffe` |
| `plan/specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `plan/specs/05-data-flows-and-privacy.md` | `6b448ef75accb2e2cf676f6be9367b5a81601f0ea775800577ea8f1cbf230e5c` |
| `plan/specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `plan/specs/07-evaluation-and-studies.md` | `4aedb0c614619d68ba83aa61de6e41c12157e29bb76ec2c2c0b8ea6c504bc70f` |
| `plan/phase0/interview-guide.md` | `35f845a3978119e5ccb3fc9912e671c8fc60a3c3102e0e38afe8c668458b1a57` |
| `experiments/c6-explain-changes/CLAUDE-snippet.md` | `70f260a220e5987bae31369e376d9f5e66780491adb1d0bc20dc0d92102ad669` |
| `experiments/c6-explain-changes/README.md` | `0b826edf619df5e6cf640831a212ea0ce58a9501e1dd557d8a732f8b4155f786` |
| `experiments/c6-explain-changes/skills/explain-changes/SKILL.md` | `276400e568e9bc249f0dce5368a5164231b2714910c150374414d37d0d1aa196` |
