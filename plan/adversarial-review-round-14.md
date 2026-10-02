# Adversarial review — round 14

Reviewed 2026-10-02. Scope: the post-round-13 revisions to the team/early-signal plan, candidates, interview guide, hook budget, and C6 experiment. Reviewed the changes from `f59160b` through `e38e561`, including the renamed experimental skill at `skills/jrdev-explain-changes/SKILL.md`. Supporting specs were checked for cross-document consistency. The plan, experiment, and previous reports were not edited.

**Result: four medium findings; no new high-severity finding established.** The rollout hold, channel-readiness requirements, candidate promotion procedure, interview ordering, and benchmark definition are substantive improvements. Remaining findings concern scope correctness and consent/policy sequencing.

## Round 13 disposition

| ID | Current design disposition |
|---|---|
| P13-01 | Participant rollout is explicitly on hold; experiment readiness, consent, storage, and deletion are specified and registered. These checks have not been executed by this review. |
| P13-02 | The current-base branch uses one upstream range consistently. P14-01 identifies a different omission in uncommitted scope. |
| P13-03 | Provider disclosure and path exclusions are present. The large/sensitive scope question still occurs after content reads: P14-02. |
| P13-04 | Spontaneous ranking now precedes theme probes, importance is distinguished from observation, and minimum sample/gate rules are defined. |
| P13-05 | One owner applies eligibility, earliest phase, design acceptance, and a two-candidate capacity limit. The two-person experiment cannot schedule C6 alone. |
| P13-06 | All-off/status, trusted configuration, atomic counters, and final-decision exception consumption are specified. Registry and placement ambiguities remain: P14-03. |
| P13-07 | The experiment has a distinct skill name, collision check, snippet markers, and backup guidance. |
| P13-08 | Whole-invocation timing, aggregate latency, memory scope, samples, contention, baseline review, and reference-machine recording are specified. No benchmark was run here. |

## Findings

| ID | Severity | Finding |
|---|---|---|
| P14-01 | Medium | Net working-tree diffs can hide changes in the next commit |
| P14-02 | Medium | The sensitive/large scope question occurs after the content read |
| P14-03 | Medium | C2's composition requirements and registered enforcement paths still differ |
| P14-04 | Medium | Interview consent covers internal paraphrases, but synthesis is published |

### P14-01 — C6 must distinguish the index from the working tree

**Evidence:** [C6 skill, step 2](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) uses `git diff --stat HEAD` and `git diff HEAD` for staged and unstaged changes together. The skill promises review of staged, unstaged, untracked, and branch-committed work before a commit or PR.

**Verified counterexample:** in an isolated temporary repository, a tracked file started with `base`. A change to `staged change` was staged; the working file was then restored to `base` without staging that reversal. `git diff HEAD` was empty, while `git diff --cached` contained the change that the next ordinary commit would include, and `git diff` contained its unstaged reversal. C6's prescribed net diff therefore misses the exact change awaiting commit.

**Required change:** enumerate and summarize staged and unstaged layers separately, applying the same path exclusions to each. Clearly identify which version is in the index and which is only in the working tree; avoid merging opposing changes into a claim of no changes. Keep branch-committed scope separate as well.

**Acceptance:** the staged-change/unstaged-reversal fixture reports both layers and explains what would be committed. Also cover a staged deletion with a recreated working file, ordinary staged-only work, and ordinary unstaged-only work. These checks need only synthetic repositories.

### P14-02 — Ask for scope before reading large or sensitive content

**Evidence:** [C6 skill, step 2](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) lists paths, applies exclusions, **reads diffs in substep 3**, then asks which parts to cover for large or potentially credential-bearing configuration changes in substep 5. The [README disclosure](../experiments/c6-explain-changes/README.md#what-reaches-the-ai-provider) says it asks first on large or sensitive changes.

**Failure case:** `settings.json` is not excluded by name but may contain credentials, or a source diff exceeds 2,000 changed lines. Following the written order reads it into model-visible output before asking the user to choose a narrower scope. The later question cannot undo that read. This is distinct from the honestly disclosed limit that ordinary source files can contain undetected secrets.

**Required change:** put the large/sensitive assessment and any required scope question before content reads. Use path lists and statistics to make the preliminary decision. Build all committed, staged, unstaged, and approved untracked content commands from the selected scope. Describe the filtering as advisory instructions rather than a guaranteed technical privacy boundary.

**Acceptance:** with a synthetic credential-bearing configuration file and a large source diff, the first output contains paths/statistics only. No content command runs until scope is selected; declining a path keeps its bytes out of subsequent results. A routine small source diff still proceeds under the stated rules.

### P14-03 — C2 needs one exact combined decision path and tool set

**Evidence:** [C2 composition requirements](specs/03-modes-and-workflows.md#c2-composition-requirements-p13-06) promise enforcement for `Edit`, `Write`, **`MultiEdit`**, and `Bash`. The single-source [interface registry](specs/02-commands-and-policy.md#interface-registry-p6-04) still registers the C2 matcher as `Edit|Write|Bash`. The composition text places check-ins after rows 1–4 and before row 7, while rows 5 and 6 of the edit table already terminate with Allow for open policy and allowlisted paths.

**Failure case:** an implementation follows the C2 registry and does not run its dedicated check for `MultiEdit`. Another implementation inserts the check just before exception row 7, retaining the earlier open-policy and allowlist returns; it never applies check-ins to those allowed edits. Both differ from the proposed independent check-in behavior. The stated final-decision-before-consumption rule addresses spent exceptions, but does not by itself resolve these paths.

**Required change before adopting C2:** reconcile the candidate description, registry, matcher, and tests to one tool set. Place the check-in decision explicitly before ordinary Allow exits if it is intended to constrain open edits and allowlisted files, preserving the named recovery/control exceptions. Define the equivalent Bash path separately; Bash is not in the file-edit decision table. If any of these paths are intentionally exempt, label that limit consistently.

**Acceptance:** at the threshold, exercise open-policy Edit, an allowlisted Write, MultiEdit, and Bash, plus Typing-denied and exception-covered edits. Each follows its declared combined decision. Recovery remains reachable, `/jrdev:off` disables C2, and a C2-denied edit does not spend a stuck exception. This is an extension of the revised composition check, not a reversal of the all-off fix.

### P14-04 — Interview publication needs a matching consent scope

**Evidence:** [Interview readiness](phase0/interview-guide.md#before-the-first-interview-gate) defines scope (b) as anonymized paraphrases **in internal docs**, with product-research use, optional verbatim quotes, and optional recording as the other scopes. Its synthesis step publishes paraphrased themes and concept reactions into the plan's early-signals section. The [C6 readiness checklist](../experiments/c6-explain-changes/README.md#readiness-before-inviting-anyone-p13-01) correctly distinguishes paraphrased themes **in the public plan**, showing that the two collection channels currently promise different uses.

**Failure case:** an interviewee agrees to research and internal paraphrases, declines optional quotes, and reasonably expects their account to stay internal. The interviewer publishes a paraphrased incident or reaction in the plan under the synthesis instructions. Removing the name or raw notes does not make an internal-use permission into an explicit public-use permission, especially for a distinctive workplace story.

**Required change:** give interviews an explicit public-paraphrase scope, optional and separate from research participation, or keep their outputs internal unless they meet a clearly disclosed aggregate-only rule. Apply current consent and the existing identifiability check before publishing. Align the consent form, note template, synthesis instructions, inventory, and Portuguese opening; do not require public reuse to participate.

**Acceptance:** synthetic interviews consenting to internal use only do not generate public individual-derived paraphrases or distinctive incidents. A separate consenting record can support an approved public theme. Withdrawal before synthesis changes the output, and a permitted aggregate-only publication is classified and disclosed explicitly.

## Verification and limits

Read the round-13 remediation diff and the current C6 skill, README, snippet, and relevant cross-spec rules. The renamed skill was treated as a review artifact, not invoked. An initial snapshot comparison encountered its removed old pathname; the review then used the replacement path and did not treat the rename as a missing implementation.

P14-01 was reproduced using Git in a temporary repository, which was removed afterwards. The other counterexamples are static traces of the written contracts. No participant was enrolled, no private feedback was read, and no actual configuration was installed or uninstalled. Hook behavior, resource budgets, secret filtering, and consent workflows were not executed.

The earlier checkpoint-ledger review is not the focus of this pass. The scoped rollout hold remains appropriate until its readiness checks pass; this report does not claim those checks failed in a live system. No new external factual claim was needed for these findings.

## Reviewed snapshot

SHA-256 identifies the current source artifacts and supporting contracts, including the replacement skill pathname.

| Source | SHA-256 |
|---|---|
| `plan/jrdev-ai-plan.md` | `2064fb7e7138805fbfc98dae66a5254b8a6851b2ffd73126b363f46edbbe3a42` |
| `plan/specs/01-website-and-feedback.md` | `0b8b76585977e79c834b2c0a0b25b788ea86c1c08c5c97f25366808d9ae292e6` |
| `plan/specs/02-commands-and-policy.md` | `0c51f5c60b8bfe1a0ba99eac49defe53e3c72119ab3edd096d89f465fa1b9374` |
| `plan/specs/03-modes-and-workflows.md` | `34010c2a65cc22e6f61d4964fbe5082fa2528d061b40aaf1d4203464291eb1b0` |
| `plan/specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `plan/specs/05-data-flows-and-privacy.md` | `e60169c65ea1cec7a7e3163e535139be539151a8cc0be3975385232eb35c741d` |
| `plan/specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `plan/specs/07-evaluation-and-studies.md` | `e2800b93f6907e1300e39fdaf2135f94f1368159ee192411678d586d44187750` |
| `plan/phase0/interview-guide.md` | `69c415987a234dfce93d5b9cd4255849b98e79a06c487174ffcfab0b79d24350` |
| `experiments/c6-explain-changes/CLAUDE-snippet.md` | `0d26a58752d2735324a9dcb54f9cf164dd1c1124c8863e22eda57d98f97e86a4` |
| `experiments/c6-explain-changes/README.md` | `0073325997dbce6d752be282411432fbe331c34374cacab5f76f476b903e9831` |
| `experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md` | `9c22b29072d2c0a1d404f8d7e67bedd2f2285dc6285490bae232f9079e3dc458` |
