# Adversarial review — round 2

Reviewed **2026-10-01**, against the revised [AI practices report](ai-for-junior-devs.md), revised [skills report](agent-skills-for-junior-devs.md), and [first review](adversarial-review.md).

## Verdict

**The revisions materially improve the reports. Further targeted corrections are needed, especially in the skills report's security claims and the treatment of LACY.** Most of round 1's broad causal overclaims have been addressed. This round identifies remaining issues, regressions introduced during revision, and places where the first review itself was incomplete or overstated.

No source report or previous review was edited in this round. Locations below refer to the revised source files reviewed today.

## Method

Read the current reports and first review; distinguish resolved findings from live ones; inspect additional primary sources and selected full methods; seek counterevidence to the reports' framing. No skills were installed or executed. Repository implementation and cross-agent compatibility remain outside this review's verification scope.

Severity: **High** affects a central conclusion or security boundary; **Medium** changes interpretation or practical application; **Low** affects precision or traceability. Confidence is confidence in the finding, not proof that the contrary proposition is true.

## Findings

### R2-01 — High: Snyk's denominator was incorrectly combined during revision

**Locations:** skills report lines 21, 192–197. **Status:** regression. **Confidence:** high.

The revised text describes 3,984 skills as ClawHub plus a top-100 skills.sh baseline. Snyk assigns the 3,984 denominator to **ClawHub alone** and presents the skills.sh set separately. Consequently, the listed percentages must remain attached to ClawHub, rather than a combined sample. The report also puts exposed secrets in the critical category, while Snyk classifies secret detection as **high** severity. Its critical categories are prompt injection, malicious code, and suspicious downloads. [Snyk taxonomy and results](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/).

**Suggested replacement:** “Snyk scanned 3,984 ClawHub skills and separately compared a curated top-100 skills.sh baseline. In the ClawHub sample, 13.4% had critical findings and 36.8% had findings of any severity; 76 malicious payloads were manually confirmed.” Use the source's category names and keep scanner findings separate from confirmed malicious behavior.

Round 1 correctly named the denominator; the revised report did not preserve that distinction.

### R2-02 — High: Automatic hook execution is confused with policy enforcement

**Locations:** skills report lines 11, 49, 201–207. **Status:** remaining issue and internal inconsistency. **Confidence:** high.

The opening says only hooks, permissions, and sandboxes enforce anything; the mechanism table gives hooks an unqualified “Yes.” Yet the Learning plugin description correctly calls its SessionStart-injected instructions advisory. Automatically running a hook does not make its natural-language output an enforced rule.

Claude Code's official documentation distinguishes blocking and nonblocking events and responses. A properly configured PreToolUse hook can block a tool call. SessionStart does not block the session merely by returning exit code 2. Other events have different decision mechanisms. [Official hooks reference](https://code.claude.com/docs/en/hooks).

**Fix:** Replace the binary hook entry with “depends on event, configuration, and response.” Distinguish automatic execution, advisory context injection, checks that can deny a particular action, and OS-level access restrictions. Explain whether hooks and subprocesses are covered by the chosen sandbox; do not assume all execution paths have identical permissions.

Round 1 identified the need for enforcement distinctions but did not verify this hook-specific detail.

### R2-03 — Medium: LACY's headline comparison needs a sample size and bundled-treatment caveat

**Locations:** AI report lines 27, 128, 165; skills report lines 134, 141. **Status:** incomplete correction. **Confidence:** high.

The full paper identifies **five learners**, averaging **4.6 years of programming experience**, and two experts. The within-subject comparison used expert-prepared AI-assisted tours **with podcasts** versus learner-generated AI-only tours. It counterbalanced order and features, but did not isolate expert review as the sole difference or include a traditional-documentation baseline. [LACY methods and results](https://arxiv.org/html/2603.25391v1).

The 83% versus 57% quiz result is accurately quoted. However, “expert curation matters” should not imply a precisely estimated, isolated effect of a maintainer checking an arbitrary generated guide. Domain context, selection, structure, and bundled presentation also differ. The reported expert comprehension ratings were much closer, so the quiz is not the only relevant outcome.

**Fix:** Add learner count and treatment details wherever the result is prominent. Describe it as evidence consistent with the value of expert-guided onboarding. Retain maintainer review as a sensible practice, with effectiveness and cost dependent on context.

**Correction to round 1:** Finding 4 should be medium priority. Its abstract-only “High” classification overstated the omission's importance.

### R2-04 — Medium: Copilot's 26% estimate can now be verified, but it is not simply an access effect

**Locations:** AI report line 38; round 1's remaining verification gaps. **Status:** gap closed; new precision issue. **Confidence:** high.

An author-hosted June 2025 paper is accessible. It reports three company experiments, **4,867 developers**, and a pooled **26.08% estimate with a 10.3 percentage-point standard error**. Its preferred instrumental-variable analysis uses randomized assignment to estimate the local average treatment effect of usage for adopters under imperfect compliance. This differs from the average effect of merely offering access to all assigned developers. [Author-hosted paper, introduction and footnote 4](https://demirermert.github.io/Papers/Demirer_AI_productivity.pdf).

**Fix:** Cite the accessible paper, include sample size and uncertainty, and describe the estimate as a pooled effect of usage for adopters under the paper's assumptions. Avoid applying 26% to every junior, agent, or task. Preserve the distinction between throughput and learning.

**Correction to round 1:** Its blocked-SSRN caveat was accurate for that attempt, but should no longer be treated as an unresolved verification barrier. A publisher's inaccessible URL need not make the underlying research inaccessible.

### R2-05 — Medium: The admitted evidence imbalance still shapes the practical framework

**Locations:** AI report lines 40, 48, 52–59, 94, 201–203. **Status:** remaining synthesis weakness. **Confidence:** high that relevant counterevidence exists; transfer to junior coding is uncertain.

The report now admits that positive tutoring results are underrepresented. Disclosure improves honesty, but does not balance a synthesis that uses an essay-writing study to motivate attempting work first while omitting other educational trials relevant to tutor design.

Two primary publications worth incorporating:

- Kestin and colleagues report greater learning in less time with a deliberately designed AI tutor than an active-learning classroom in college physics. This is evidence about a specific educational intervention, not ordinary coding agents. [Published study abstract](https://pubmed.ncbi.nlm.nih.gov/40537565/).
- Bastani and colleagues compare standard AI assistance, a tutor with learning safeguards, and a control in high-school mathematics. They distinguish assisted practice performance from subsequent unaided performance and report that tutor safeguards mitigate the harmful effects found with unrestricted assistance. [Published study](https://doi.org/10.1073/pnas.2422633122), [primary abstract](https://pubmed.ncbi.nlm.nih.gov/40560616/).

**Fix:** Include these as explicitly noncoding evidence, using the same transfer caveat applied to the EEG study. Compare intervention design, outcomes, and assessment timing. Do not conclude that “think first” or withholding answers is always superior; evaluate supported practice and later independent transfer.

This does not demonstrate efficacy for any named SKILL.md package.

### R2-06 — Medium: Observed human prompting patterns are mapped to different tutor behaviors

**Locations:** skills report lines 27–37, especially line 31 and the column heading. **Status:** new issue identified. **Confidence:** high.

In the coding study, conceptual inquiry means the **human asks conceptual questions and independently writes code**. The skills table maps it to the **agent asking the human questions and withholding answers**. Both may promote engagement, but they are different interactions. Plan interviews, TDD, and security checks in the same table were not the six observed interaction categories in that study. [Anthropic's interaction categories](https://www.anthropic.com/research/AI-assistance-coding-skills).

The revised design-hypothesis disclaimer is useful, but the table still gives all rows the appearance of study-derived patterns.

**Fix:** Separate “patterns observed in the coding study” from “additional proposed learning and engineering supports.” State the actor for each behavior. Describe Socratic tutoring as a related design hypothesis rather than a direct implementation of conceptual inquiry.

### R2-07 — Medium: Lack of discovered evaluations becomes a universal absence claim

**Locations:** skills report lines 9, 17, 223. **Status:** remaining overclaim. **Confidence:** high.

“No skill here has published evidence” and “none has been evaluated” are stronger than “no evaluations were found.” The report supplies no reproducible search method that establishes exhaustive coverage of papers, repositories, unpublished internal testing, and renamed implementations. The final verification-gap wording is appropriately narrower than the opening.

**Fix:** Use “this review did not find published learning-outcome evaluations of these specific packages.” Record search terms, sources, cutoff date, and inclusion criteria if a comprehensive absence claim is important. Distinguish evidence supporting a pedagogical method from evaluation of a particular implementation.

### R2-08 — Medium: Outcome measurement advice does not account for repeat-task learning or total costs

**Locations:** AI report lines 25, 101, 106, 168; skills report lines 174, 178, 182. **Status:** practical gap. **Confidence:** high as a methodological concern; optimal protocol is context-dependent.

“Time some tasks with and without AI” can mislead if the same task is repeated: the second run benefits from what was learned in the first. Comparing unrelated tasks introduces difficulty differences. Measuring coding time alone omits prompting, verification, reviewer work, rework, tool costs, and later maintenance. The reports name several of these costs in onboarding, but do not carry them into their personal measurement advice.

Likewise, calling alternative understanding checks “better” is an untested ranking even with an opinion tag. Predicting behavior and transferring skills measure different things; no single score resolves all of them.

**Fix:** Suggest comparable tasks with varied order, record difficulty and prior familiarity, include end-to-end effort and quality, and treat small samples as personal observations. Track immediate comprehension, delayed transfer, and aided delivery separately. Say “complementary checks,” rather than claiming an unsupported ranking.

### R2-09 — Low: Stanford's endpoint does not match the linked paper version

**Locations:** AI report lines 26, 45. **Status:** factual precision issue. **Confidence:** high.

The report gives mid-2025 or July 2025 as the endpoint, but the linked November paper uses payroll data through **September 2025**, including in its discussion of the nearly 20% developer decline. [Linked Stanford paper, sections 3–4.1](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf).

**Fix:** Align the endpoint with the cited version, or cite the older version that supports the July claim. Preserve the existing distinction between software developers' unadjusted trend and broader adjusted AI-exposure estimates.

### R2-10 — Low: Evidence labels mix design, provenance, and publication status

**Locations:** AI report lines 7–14, 33–46, 128–130. **Status:** usability issue. **Confidence:** high.

“Vendor” describes who produced research; “RCT” describes design; “Paper” describes format. A vendor can run an RCT, and a paper can report speculation or an experiment. The current labels are not a consistent classification and omit later-used labels such as controlled study and docs. The RCT badge on post hoc subgroups can also create an impression that those particular behaviors were randomized despite the nearby caveat.

**Fix:** Use separate columns or fields for design, population and outcome, provenance/conflicts, publication status, and verification level. Label interaction-pattern findings “post hoc observational analysis within an RCT.” Keep practical advice explicitly marked as proposed.

## Status of the first review

| Round 1 concern | Current status |
|---|---|
| Randomized access versus self-selected usage patterns | Mostly addressed; the design table still conflates behaviors (R2-06). |
| Immediate comprehension versus long-term learning | Addressed in the principal claims. |
| Onboarding superlative and causal interpretation of DX | Addressed; current wording acknowledges confounding and delivery proxies. |
| Omitted LACY result | Added, but needs fuller qualification; round 1's severity was excessive (R2-03). |
| Snyk ecosystem generalization | Partly addressed, with a new denominator/category error (R2-01). |
| Proven efficacy inferred from skill intent and popularity | Mostly addressed; absence-of-evidence language remains too absolute (R2-07). |
| Stack Overflow's 29% trust figure | Addressed with total and component categories. |
| Missing METR follow-up | Addressed. |
| EEG domain and subgroup limitations | Addressed in scope; broader evidence selection still needs work (R2-05). |
| Survey and observational causality overclaims | Mostly addressed. |
| Individual career-protection inference | Addressed; Stanford date needs alignment (R2-09). |
| Junior adoption inferred from registry totals | Addressed. |
| Rigid timers, read-only week, per-commit quizzes | Addressed through adaptable examples. |
| Permissions versus advisory instructions | Improved, but hook enforcement remains misstated (R2-02). |
| AI quizzes as sole proof of understanding | Addressed through independent and complementary checks. |
| Untraceable rankings and mentoring percentage | Removed. Primary-source verification of the Copilot result is now available (R2-04). |

## Further verification and limits

The SSRN 6409098 title, author, dates, and 28% claim are now visible in its indexed publisher abstract. This supports bibliographic identification and what the author claims; it does **not** validate the job-posting analysis or its methods. The revised report appropriately keeps the result provisional. [SSRN abstract](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098).

Round 1's verification statements are historical records of checks performed then, not permanent certifications. Live install counts, repository paths, tool availability, and UI instructions can change. This second pass did not re-audit every tool description, verify the X post, test install commands, or assess the full EEG, Veracode, and GitClear methods.

## Recommended next revision

1. Correct the Snyk denominator and severity taxonomy, and replace the binary hook-enforcement claim.
2. Add LACY's five-learner sample and bundled conditions; correct round 1's overly strong endorsement when using it as editorial guidance.
3. Cite the accessible Copilot paper and its adoption-based estimate, uncertainty, and sample size.
4. Add balanced tutoring evidence with explicit noncoding scope; distinguish study-observed prompting from proposed tutoring designs.
5. Narrow evaluation-absence claims, improve the measurement suggestions, and align source versions and evidence labels.

The reports are substantially more defensible after their revisions. Remaining corrections can be made without abandoning their practical framework, provided the framework stays a proposal whose usefulness depends on the learner, task, tool, and independent verification.
