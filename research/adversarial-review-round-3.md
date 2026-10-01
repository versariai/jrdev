# Adversarial review — round 3

Reviewed **2026-10-01**. Scope: the current [AI practices report](ai-for-junior-devs.md), [skills report](agent-skills-for-junior-devs.md), and the conclusions of [round 1](adversarial-review.md) and [round 2](adversarial-review-round-2.md).

## Verdict

**The reports now handle most of the earlier evidence problems responsibly. This pass found seven narrower issues, with no new high-severity finding.** The most useful next corrections concern actual skill implementation, statistical interpretation, and precise measurement. Further general caveats would add less value than correcting these specific claims.

This round also corrects round 2's overly categorical treatment of Snyk's sample description. An adversarial review must apply the same evidentiary standards to its own conclusions.

## Method and boundaries

Read the revised reports, reassess the previous findings, and inspect selected primary publications, official documentation, and repository files. Unlike earlier passes, this review records exact revisions for the skill files inspected. No third-party skill, script, hook, or installation command was executed. Code observations below are static analysis, not runtime test results or a complete security audit.

Severity: **Medium** materially affects interpretation or application; **Low** affects precision or traceability. Confidence concerns the identified problem, not whether the opposite claim is true. Locations refer to the current source files, identified by hashes at the end.

## Findings

### R3-01 — Medium: The git guardrail's mechanism is now verified, but its coverage is limited

**Location:** skills report line 130 and remaining verification gaps. **Confidence:** high for mechanism and static limitations.

The actual skill configures a **PreToolUse hook matching Bash**, copies a bundled shell script, and uses exit code 2 to deny matching commands. It is not merely an advisory prompt after setup. [Pinned skill instructions](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/git-guardrails-claude-code/SKILL.md).

However, the script is a list of regular-expression matches against command text, not a parser or comprehensive Git access control. For example, `git -C some-repo push` does not match its `git push` expression. Aliases, other execution tools, and alternate argument forms also require separate consideration. It depends on `jq`; extraction errors are not explicitly made fatal before the final success exit. These are static limitations, not demonstrated live bypass tests. [Pinned script](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/git-guardrails-claude-code/scripts/block-dangerous-git.sh).

**Fix:** Replace “mechanism not verified” with this verified mechanism and bounded coverage. Describe it as blocking selected command strings through one tool path, contingent on setup and dependencies. Avoid implying that its name or installation prevents every dangerous Git operation.

### R3-02 — Medium: “FSRS” overstates what the tutor's scheduling instructions implement

**Locations:** skills report lines 78–80 and the spaced-repetition mapping. **Confidence:** high for the implementation mismatch; effectiveness remains unknown.

The tutor's `references/fsrs.md` uses rating-based growth multipliers with a difficulty adjustment. These are explicitly described as approximate practical multipliers. The published FSRS algorithm uses parameterized state-update formulas involving memory state and retrievability; sharing the names difficulty and stability is not enough to reproduce it. [Pinned tutor reference](https://github.com/Bhala-Srinivash/agent-tutor-skill/blob/e273585b542d1d77b72c87eaa605d384ea4479f1/references/fsrs.md), [upstream algorithm](https://github.com/open-spaced-repetition/awesome-fsrs/wiki/The-Algorithm).

The reference also labels concepts mastered when estimated stability reaches 90 days and stops actively scheduling them. That badge is a rule derived from the heuristic, not an independent demonstration of durable understanding or engineering transfer.

**Fix:** Describe the package as requesting “FSRS-inspired heuristic scheduling in prompt instructions,” unless an actual FSRS implementation is separately verified. Treat mastery labels as scheduling metadata. Retain independent practice checks. Also narrow “this implementation has not been evaluated” to “this review found no evaluation,” consistently with the report's revised opening.

### R3-03 — Medium: Nonsignificant completion-time results still become evidence of little benefit

**Location:** AI report line 37, final column. **Confidence:** high.

The row correctly says the roughly two-minute time difference was not statistically significant, but its takeaway says AI reduced understanding “without saving much time.” That turns a descriptive sample estimate into a stronger conclusion about the underlying effect. Failure to detect a difference is not proof that the effect is negligible, nor an equivalence test. The significant comprehension result and uncertain time result need separate interpretations. [Anthropic summary](https://www.anthropic.com/research/AI-assistance-coding-skills).

**Fix:** “AI access lowered average immediate quiz performance in this task; the study did not establish a completion-time benefit.” Keep the observed difference as a descriptive estimate and report interval uncertainty if obtainable. Do not conclude that AI has no useful speed effect from this nonsignificant result.

### R3-04 — Medium: The new math-study headline leaves the comparison and units implicit

**Locations:** AI report lines 25, 39 and the short-term/long-term discussion. **Confidence:** high for the missing comparison.

“Students did 17% worse once access was removed” can be read as a within-student decline from assisted practice to an unaided exam. The published abstract describes the result as **17% lower grades than students who never had AI access**. The 48% and 127% practice improvements and the 17% exam reduction are relative changes, not percentage-point changes. [Bastani et al. abstract](https://pubmed.ncbi.nlm.nih.gov/40560616/).

The article supports a distinction between assisted practice and subsequent unaided assessment. That alone does not establish years-long retention or career competence; the exact assessment interval is still absent from the report's time-horizon column.

**Fix:** Name the no-AI comparison group, identify these as relative changes, and supply the actual assessment timing from the methods before characterizing it as evidence about durable learning. “Largely mitigated” is supported wording; avoid upgrading it to proof of zero harm.

### R3-05 — Medium: Copilot's estimate needs its instrumental-variable assumptions and tool vintage

**Location:** AI report line 42. **Confidence:** high for scope; no claim that the estimate is invalid.

The revision correctly distinguishes usage from offered access. However, “for those who adopted it” may imply an effect for all observed users. An instrumental-variable local effect pertains to developers whose adoption changed because of assignment, under the identification assumptions, rather than necessarily every adopter. The paper calls its estimate a local average treatment effect. [Author-hosted paper, introduction and footnote 4](https://www.mertdemirer.com/Papers/Demirer_AI_productivity.pdf).

The paper also explicitly studies **2022–2023 Copilot autocomplete**. The report supplies a study duration but omits this tool period. Publication in 2025 does not make it an evaluation of current autonomous coding agents.

**Fix:** Describe the number as the authors' pooled IV estimate for adoption induced by assignment, subject to their assumptions. Write the standard error as **10.3 percentage points** to avoid confusion with a relative error. Add the 2022–2023 autocomplete scope. Keep the verified 4,867-person sample and throughput/learning distinction.

**Refinement to round 2:** Its recommended “for adopters” wording was still too broad, though consistent with the paper's informal summary.

### R3-06 — Low: Overall survey participation is substituted for item denominators

**Location:** AI report line 46. **Confidence:** high.

The row lists approximately 49K developers, but the cited frustrations item has **31,476 responses** and the accuracy/trust item has **33,244**. The overall survey size is legitimate context, but is not the denominator for these percentages. [Original AI survey items](https://survey.stackoverflow.co/2025/ai).

**Fix:** List the item-specific response counts alongside the figures. Describe the proportions as answers among respondents to those questions, not population prevalence among developers or juniors. The headline percentages themselves remain supported.

### R3-07 — Low: Built-in styles and plugin instructions are treated as identical

**Location:** skills report line 60. **Confidence:** high for the unsupported equivalence.

The official Learning plugin's hook emits **additional context at SessionStart**. Its own comment describes combining an earlier Learning style with explanatory functionality. The current built-in Learning style is documented separately and includes its own contribution-and-resume behavior. These mechanisms support similar goals, but this review has not established that their instructions are identical. [Pinned plugin hook](https://github.com/anthropics/claude-plugins-official/blob/ab024cdcfa7ca80be204acd4907656ba5a968589/plugins/learning-output-style/hooks-handlers/session-start.sh), [current official output-style documentation](https://code.claude.com/docs/en/output-styles).

**Fix:** Replace “the same instructions” with “similar learning and explanatory instructions.” Separate built-in style configuration from plugin setup. Both are advisory, and choosing one is not evidence that installing both adds learning value. The terminal `/config` route and the existence of built-in Learning are supported by current docs.

## Correction to the earlier review: Snyk's article is internally inconsistent

Round 2's R2-01 asserted unambiguously that the denominator belonged to ClawHub alone. The source's introduction actually describes 3,984 skills as coming from both ClawHub and skills.sh, while its results section calls them ClawHub skills. Its opening also includes exposed secrets among critical issues, while its taxonomy rates secret detection high. [Snyk article, introduction and taxonomy/results](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/).

The source reports inconsistent descriptions. The revised skills report now acknowledges that ambiguity and follows the explicit severity taxonomy. That is a defensible correction. **Do not count the denominator ambiguity again as a proven error in the revised report.** Prefer the detailed results' attribution, but qualify it rather than silently reconciling the article. The top-100 skills.sh baseline and its zero critical detections do not establish that all skills.sh packages are safe.

## Additional verification completed

- **`scaffold-exercises` is inspectable.** Its instructions create sections with problem, solution, and explainer directories, require `pnpm ai-hero-cli internal lint`, and direct a Git commit. It is a workflow for a particular exercise-authoring environment, rather than proof of a general adaptive tutoring capability. Its earlier “behavior not verified” gap can now be replaced with a source-level description and prerequisites. [Pinned skill](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/scaffold-exercises/SKILL.md).
- **The `quiz-me` install example matches its README.** This confirms provenance of the command, not successful installation or cross-agent compatibility. [Repository](https://github.com/rodbv/socratic-skills/tree/dda051c2b4390d19d32b33edc222abbfa39927ac).
- **The current EEG abstract supports the reported group sizes and switch directions.** This pass does not add a full methods audit or establish applicability to coding. [Versioned preprint abstract](https://arxiv.org/abs/2506.08872v2).

## Status after the second revision

| Round 2 issue | Current disposition |
|---|---|
| Snyk denominator and categories | Revised report acknowledges source ambiguity; taxonomy corrected. Round 2's certainty needs qualification. |
| Hook execution versus enforcement | Main table improved. Actual Git guardrail mechanism now verified, with limitations (R3-01). |
| LACY sample and bundled treatment | Addressed with five learners, podcasts, counterbalancing, and multiple outcomes. |
| Copilot primary source and uncertainty | Addressed; causal population, units, and tool vintage need refinement (R3-05). |
| Missing educational counterevidence | Addressed with math and physics trials, explicitly outside coding. |
| Human inquiry versus Socratic tutor | Addressed by separate mappings and explicit actors. |
| Universal claim that evaluations do not exist | Opening corrected; one local sentence remains too absolute (R3-02). |
| Repeat-task bias and incomplete effort measurement | Addressed with comparable tasks and broader effort tracking. |
| Stanford endpoint | Addressed: September 2025. |
| Evidence classification | Improved; design and provenance are separated in the main table. |

## Verification record

Repository revisions inspected, **not installed**:

| Repository | Revision |
|---|---|
| mattpocock/skills | `d81f3a183412e71a5b1e84ca21bc1a35eea03a60` |
| Bhala-Srinivash/agent-tutor-skill | `e273585b542d1d77b72c87eaa605d384ea4479f1` |
| anthropics/claude-plugins-official | `ab024cdcfa7ca80be204acd4907656ba5a968589` |
| rodbv/socratic-skills | `dda051c2b4390d19d32b33edc222abbfa39927ac` |

SHA-256 of source reports reviewed:

```text
ai-for-junior-devs.md
4825ffea35ab7749c3df33561ca595480aa50bf4f748ecef0742db293f04f4be

agent-skills-for-junior-devs.md
b07526f288d00a491a9e0b69974bf6f0cd78606dd007158ef819a44ebcb9064e
```

Open limits: no runtime verification, no comprehensive repository security audit, no exhaustive search for learning evaluations, no authentication of the employee X post, and no full methods review of every cited study. Static descriptions should not be promoted into tested behavior.

## Recommended next changes

Correct the verified Git guardrail and heuristic scheduler descriptions first. Then clarify statistical uncertainty, math-study comparisons, Copilot scope, and survey denominators. Preserve the revisions already made to LACY, causal language, human mentoring, and adaptable practice. Keep previous reviews as historical records and use this round's explicit corrections when deriving the next edit list.
