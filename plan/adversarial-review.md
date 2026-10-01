# Adversarial review of the jrdev.ai product plan

Reviewed: 2026-10-01. Scope: [jrdev-ai-plan.md](jrdev-ai-plan.md), draft dated 2026-10-01. This is an execution and product review, separate from the research review rounds. The plan was not modified.

Source SHA-256: `e3081f02759ec8fc50aa92cb371521af3f416a3a27f67023c92e37edc6e1cfa6`.

## Verdict

The plan is suitable for a narrower discovery prototype, but its current guarantees and launch gates are not ready for implementation as written. Its strongest choices are explicit escape hatches, delayed transfer tasks, readable hooks, and distinguishing advisory prompts from tool denial. Several later sections undermine those choices: production exclusion relies on superficial checks; edit denial becomes a claim of unaided performance; and locally stored profiles are injected into model context without an equivalent data-flow disclosure.

There are **3 high-priority and 7 medium-priority findings**. High means a central safety, privacy, or learning claim can fail under the specified design. Medium means a material implementation or evaluation ambiguity must be resolved before the corresponding feature ships. These are design findings, not reproduced vulnerabilities in implemented software.

| ID | Priority | Finding | Plan location |
|---|---|---|---|
| P-01 | High | Environment and URL checks cannot guarantee production exclusion | §5.5, §9 |
| P-02 | High | Denied editing tools do not establish unaided performance | §5.1–5.2, §7 |
| P-03 | High | Local storage is conflated with local processing and safe sharing | Principles 5 and 7, §3, §5.1, §6.2 |
| P-04 | Medium | Stop hooks are treated as durable workflow and human-confirmation gates | §5, §5.1, §5.3, §5.5 |
| P-05 | Medium | Mode state and exception precedence are underspecified | §5, §5.2, §6.1 |
| P-06 | Medium | The roadmap delays core validation while expanding scope | §7–8, §10 |
| P-07 | Medium | The pilot can confuse learning effects with order and cohort effects | §7 |
| P-08 | Medium | Coverage and static maps can be presented as observed execution paths | §5.4–5.5 |
| P-09 | Medium | Signed tags alone do not specify verified, immutable installation | Principle 6, §6.4 |
| P-10 | Medium | The documented commands do not match the proposed plugin skills | §5, §6.1, §6.3 |

## Findings

### P-01 — Production exclusion has no enforceable boundary

**Evidence:** §5.5 says Test mode is “ONLY for local/dev” and is enforced by environment and URL checks, with “NEVER production credentials.” §9 simultaneously says hooks must never be claimed as a security boundary.

**Failure scenario:** A local application runs with a production database credential inherited from the shell. Its browser URL is localhost and its environment label is development. A seed command, queue consumer, or SDK call still changes production data. Checking the browser URL does not constrain downstream destinations; checking environment names does not establish credential scope.

**Consequence:** The strongest safety promise is unsupported by the mechanism. Merely adding more string checks cannot close this gap.

**Required change:** Choose a supported execution model. For a first version, prefer disposable fixtures and explicitly provisioned development credentials. If production exclusion is a guarantee, define credential isolation and network restrictions for every executed process, including setup scripts and background workers. Otherwise describe the checks as warnings and make the residual risk explicit; do not retain the absolute promise.

**Acceptance:** Exercise a fixture that presents localhost/development while attempting a downstream production-like endpoint and another that inherits a forbidden credential. The claimed boundary must reject both before execution. Record which execution paths are outside that boundary.

### P-02 — Typing is being mistaken for unaided evidence

**Evidence:** §5.1 defines evidence as unaided performance and proposes transfer tasks with editing tools denied. §5.2 explicitly permits descriptions of what to write, snippets, pseudocode, and an eventual agent-written solution, then logs evidence after the learner types.

**Failure scenario:** The model prints a complete solution in chat; the learner types it, passes the test, and answers a prompted “why” question. No editing tool was used, but the performance was assisted. A transfer task can suffer the same failure if the tutor remains available to supply the answer.

**Consequence:** Topic levels can become a record of copied or coached solutions while being presented as evidence of independent competence. Correct FSRS scheduling does not validate that the queued items or ratings measure transferable programming ability.

**Required change:** Separate coached practice, learner-reported independent work, and assessed transfer. Persist the assistance level and task conditions with every evidence record. Define allowed resources, fresh tasks, scoring rubrics, and how help requests invalidate or relabel an assessment. Avoid invasive monitoring; disclose the limits of self-report. Only the explicitly defined assessment category should advance a level claimed to represent independent performance.

**Acceptance:** A complete chat solution, partial snippet, or stuck escalation cannot produce an “unaided” evidence record. A learner who declines assessment can continue practicing without an inferred mastery upgrade. Assessment reports state which independence conditions were controlled and which were self-reported.

### P-03 — The privacy promise omits processing and disclosure paths

**Evidence:** Principle 5 says shared data excludes code. §5.1 stores a global profile and injects goals and reviews into sessions. §3 routes free-text feedback through LLM clustering, interviews, and public themes. §6.2 also allows consented conversation samples. A telemetry hard-off does not govern all these flows.

**Failure scenario:** A learner's log or review contains an employer-specific incident, code excerpt, or internal identifier. That information enters a later model session through profile injection, possibly in another project. Separately, a feedback submission containing sensitive material is sent to an LLM processor or exposed through a public issue link despite “no code required.”

**Consequence:** Local persistence is compatible with external model processing. Consent to telemetry does not imply consent to transcript review, feedback processing, or public publication. The current language obscures these distinctions.

**Required change:** Specify a data-flow inventory: each field, storage location, model/provider destination, human recipient, retention/deletion rule, and publication rule. Keep global goals distinct from project-specific evidence, and minimize injected content. Resolve whether code-bearing conversation review is prohibited or separately consented. Add explicit ignore behavior for project logs/maps and clear controls for feedback processing and publication. Redaction should be reviewed before disclosure, rather than assumed to work perfectly.

**Acceptance:** A seeded sensitive project log is not injected into an unrelated project. Installation and session setup disclose what reaches the model. A hard-off setting has a documented scope. Private feedback and conversation samples cannot become public themes or linked issues without the appropriate review and consent.

### P-04 — Stop hooks cannot guarantee completion of a human workflow

**Evidence:** §5 labels Stop blocking “Enforced”; §5.1 uses it for learner confirmation of a recap; §5.5 prevents “done” until a checklist is complete. §5.3 calls the debugging loop “enforced by the skill workflow.”

**Failure scenario:** The agent asks the learner to confirm a recap, then a Stop hook blocks because confirmation is absent. Continuing the model turn cannot manufacture a legitimate human response. The agent loops, invents completion, or eventually stops with an incomplete checklist.

Claude Code documents that Stop does not run on user interruption, exposes `stop_hook_active` to avoid unresolved loops, and currently caps consecutive Stop continuations. A skill's instructions also do not make every step mechanically compulsory. [Hooks reference](https://code.claude.com/docs/en/hooks), [Skills documentation](https://code.claude.com/docs/en/skills).

**Required change:** Treat human confirmation as a pending state across turns. Specify a bounded reminder policy, interrupt/error behavior, skipped/not-applicable checklist items, and a reachable escape hatch. Reserve “enforced” for the specific tool decision or transition actually controlled, and label the broader tutoring workflow advisory.

**Acceptance:** A missing human confirmation yields control without repeated continuation or fabricated approval. Interrupting a session preserves an incomplete record. A skipped check cannot silently become a passed check. Test these against the declared supported Claude Code versions.

### P-05 — The policy can disagree with itself or change across sessions

**Evidence:** §5 stores the current mode in both project and user state. §5.2 says explicit Typing mode denies all edits but also says boilerplate, configuration, generated files, and scaffolds are “always allowed.” Smart mode depends on topic and proposed change size. The final stuck step requires a write that the current mode may still deny. §6.1 proposes fail-closed enforcement without defining recovery.

**Failure scenario:** Two sessions in one repository want different modes; one switches off while the other expects denial. Alternatively, configuration containing the learning target is allowlisted, or the user requests the final stuck step and its edit remains blocked. A missing runtime or malformed state can leave ordinary work blocked without a functioning mode-off command.

**Required change:** Define a policy decision table, project/user/session precedence, schema and atomic writes, and the authority allowed to change enforcement state. State whether explicit mode overrides allowlists. Make smart classification advisory until its inputs and failure behavior are specified. Scope write exceptions to a session, task, and expiry; restore the prior policy afterward. Provide a recovery path independent of the failing hook runtime.

**Acceptance:** Cover simultaneous sessions, contradictory settings, malformed state, absent runtime, an allowlisted target-topic file, unknown classification, and stuck escalation. Each has a documented decision and a usable recovery path; no session silently changes another's policy.

### P-06 — The roadmap invests in breadth before resolving the core uncertainties

**Evidence:** Phase 1 includes profiles, learning, typing, a website/catalog, marketplace, newsletter, signed releases, and 50 active users. Debug and Map follow before the delayed-transfer pilot in Phase 3. The tutor-behavior review process is also a Phase 3 exit criterion, although Principle 7 requires review after every model/prompt change. Capacity is only described as 1–2 people part-time. Discovery requires 100 survey responses without a recruitment plan.

**Failure scenario:** Months of feature and content work create a substantial maintenance surface before establishing that learners accept the friction or can use the tutoring workflow successfully. Fifty users encounter repeated prompt revisions before the required behavior review process is operating.

**Required change:** Move a small formative pilot and tutor-behavior regression checks into Phase 1, with the initial evaluation protocol prepared during discovery. Start with one tool, one workflow, one stack, manual assessment, and a working escape hatch. Defer additional modes and sophisticated profile scheduling until those observations justify them. Assign owners, hours/week, recruitment channels, recurring costs, and explicit proceed/revise/stop gates. Response count alone is not a demand validation gate.

**Acceptance:** The first public release has a tested tutoring and escape workflow, a repeatable behavior review, and recruited learners from the intended audience. Its exit gate includes completion, frustration, support burden, and an initial delayed assessment. Scope fits an explicit capacity budget; numerical targets have a recruitment and measurement basis.

### P-07 — The pilot's comparison is not yet interpretable

**Evidence:** §7 proposes two cohorts of about 20 juniors, Learning + Typing versus normal AI, with randomized order where possible, a pre-test, and delayed post-test.

**Failure scenario:** Learners first use the intervention and carry its debugging habits or knowledge into the normal-AI period. Reversing order does not remove learned knowledge. If one cohort is a bootcamp and the other a company, cohort differences may also dominate the comparison. Learners who find Typing frustrating can drop out before the delayed assessment.

**Consequence:** Honest publication alone does not make the result an estimate of intervention benefit. A small pilot may establish feasibility while remaining too uncertain for a learning-effect claim.

**Required change:** Declare feasibility versus efficacy as the primary objective. Specify allocation unit, comparator access, comparable fresh tasks, assessment rubric, assessor blinding where practical, follow-up timing, attrition handling, and uncertainty reporting. Prefer parallel allocation for durable learning outcomes; if using crossover, explain topic separation and carryover assumptions. Treat mentor explanations and debug predictions as distinct outcomes from delayed independent transfer.

**Acceptance:** Register the protocol before outcomes are observed. Publish assignment, baseline differences, completion/dropout counts, task conditions, and uncertainty. State which causal conclusions the design cannot support; do not convert an inconclusive comparison into an equivalence claim.

### P-08 — Executed lines and static relationships are not execution traces

**Evidence:** §5.5 calls coverage output “Code path reached” and the roadmap calls it “coverage path.” §5.4 creates maps from imports, LSP, and framework route extraction.

**Failure scenario:** Two requests execute the same lines in different orders or interleave background work. Their coverage reports look alike, but their actual paths differ. A static map omits dynamically registered routes, dependency injection, reflection, or queue dispatch, yet a learner treats it as the complete runtime flow.

Coverage.py describes executed statements and optional branch coverage, rather than an ordered request trace. [Coverage.py documentation](https://coverage.readthedocs.io/en/latest/).

**Required change:** Label coverage as executed lines/branches for a defined run. Record run boundaries and aggregation rules. Label static edges by their extraction source and unsupported or unresolved relationships explicitly. Use request-scoped tracing when promising order or causality, and explain incomplete instrumentation. Do not let a diagram's polished presentation imply completeness.

**Acceptance:** Demonstrate two runs with the same coverage but different order, and a repository with dynamic dispatch. Reports retain the distinction and show unresolved edges instead of invented connections.

### P-09 — The trust model ends at signing rather than installation verification

**Evidence:** §6.4 says signed tags/releases and a marketplace pointing to tags give users pinned versions. It does not define the trust root, verification point, immutable artifact, or update/rollback behavior.

**Failure scenario:** A tag is moved, or a marketplace refresh changes a relative-path package. A signed release exists, but the installed artifact is never checked against that signature. The user cannot demonstrate that the executable hooks match the reviewed version.

Claude Code documents source-specific pinning through git `ref` or `sha`; this must be designed for the actual marketplace/package layout. [Marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces).

**Required change:** Specify immutable commit/artifact identifiers, dependency inclusion, signer trust, and the installation verification mechanism. Separate publisher signing from verified client installation. Document how marketplace changes, plugin updates, and runtime dependencies interact; define rollback and explicitly disclosed update behavior. If the client does not verify signatures, say what users must verify themselves.

**Acceptance:** Install a release in a clean environment and show the exact source/artifact identity. A moved tag cannot silently alter an installation described as immutable. Demonstrate both update and rollback, including the bundled hook runtime/dependencies.

### P-10 — The command contract needs a router or corrected names

**Evidence:** §5 documents `/jrdev mode ...`, `/jrdev status`, `/jrdev stuck`, and `/jrdev learn setup`; elsewhere it mentions `/jrdev-debug`. The proposed tree instead has skills named `learn`, `type`, `debug`, `map`, `test`, `stuck`, and `feedback`, with no root router or status implementation.

Plugin skills are normally invoked as `/plugin-name:skill-name`, so that tree does not by itself supply the advertised command family. [Skills documentation](https://code.claude.com/docs/en/skills).

**Required change:** Choose the supported command syntax and align the tree, onboarding instructions, tests, and escape hatches. If presenting a single router, specify how it dispatches commands and changes state deterministically rather than relying only on the model to interpret text.

**Acceptance:** From a clean install, execute every documented command, including status, setup, stuck escalation, and mode off. Confirm the state change and resulting hook decision. Manifest validation and recorded hook-input tests alone are insufficient; marketplace validation can pass despite a missing source directory that fails installation. [Marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces).

## Recommended revision sequence

1. Resolve the three high-priority claims: supported test execution boundary, evidence categories, and privacy/data flows.
2. Define the mode/state/escape contract and correct the command surface. Prove it with a small installed prototype, including failure recovery and human-waiting behavior.
3. Recut Phase 1 around that prototype, early behavior review, and a feasibility pilot. Write the assessment protocol before recruiting for any learning-effect comparison.
4. Add maps, coverage, and release-trust features only with the narrower claims and feature-specific acceptance checks above.

Review limitation: this is a review of the written plan and current primary documentation, not an audit of implemented hooks, a security penetration test, or a completed pilot. No learning effectiveness result is implied.
