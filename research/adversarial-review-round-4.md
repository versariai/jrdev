# Adversarial review — round 4

Reviewed **2026-10-01**. Scope: the revised [AI practices report](ai-for-junior-devs.md), [skills report](agent-skills-for-junior-devs.md), and [previous review](adversarial-review-round-3.md), with earlier rounds used to avoid duplicating findings.

## Verdict

**Three remaining medium-severity findings; no new high-severity finding.** The previous round's corrections are substantially incorporated. This pass concentrates on the interpretation of positive tutoring evidence and an overlooked primary publication about a programming tutor. It does not establish that the recommended workflows are ineffective.

The useful next revision is small: qualify the physics comparison, distinguish a successful tutoring package from evidence for an individual teaching technique, and use CS50's published evaluations to make the tutor recommendation more concrete.

## Method and boundaries

Read the current reports and reassess round 3. Check selected primary sources, including the published physics trial and the author-hosted CS50 conference paper. The physics publisher/PMC page intermittently presented access challenges; the indexed primary article supplied the study details used below. No third-party skills, hooks, installers, or scripts were executed. This is an evidence and document review, not a runtime security audit or systematic literature review.

**Medium** means a problem materially changes interpretation or application. Confidence refers to the identified mismatch or omission. Source locations and hashes describe the reviewed snapshot; recommendations below have not been applied to the source reports.

## Findings

### R4-01 — Medium: The physics row omits the comparison conditions and assessment horizon

**Locations:** AI report lines 25, 40, 52 and 213. **Confidence:** high.

The headline is consistent with the paper, but its population/time-horizon column gives neither sample size nor assessment timing. The trial involved **194 Harvard physics students**, two lessons in consecutive weeks, and a crossover between at-home AI tutoring and classroom active learning. Post-tests followed each lesson. The AI condition also allowed self-pacing. [Published trial](https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/).

The result supports this instructional package in that setting. It does not isolate AI from pacing/location, establish delayed retention, or test junior programmers.

**Fix:** Add the sample, crossover design, two-lesson exposure and post-lesson assessment to the table. Describe the comparison as a custom, self-paced at-home tutor versus classroom active learning. Preserve the positive result while naming its boundaries, as the report already does for the math trial.

### R4-02 — Medium: Evidence for tutoring packages is used to support a specific Socratic mechanism

**Location:** skills report line 39 and the following Socratic-tutoring row. **Confidence:** high for the attribution problem; effectiveness of the proposed skills remains unknown.

The sentence explicitly distinguishes tutor questioning from learner conceptual inquiry, which fixes the previous actor mismatch. However, it says the Socratic design idea is supported by the math and physics results. Those comparisons evaluate tutoring packages, not random assignment to Socratic questioning versus another teaching technique. A successful package does not identify which component caused its benefit.

The math trial is useful evidence about restricted versus unrestricted assistance, including subsequent unaided assessment; it is not a component-level test of the listed skills. [Math trial](https://doi.org/10.1073/pnas.2422633122). The physics result likewise does not independently establish the benefit of agent-led questioning.

**Fix:** Replace the sentence with: “Structured tutoring has promising evidence in math and physics. Socratic questioning is one proposed implementation; these studies do not isolate its contribution or evaluate the skills listed here.” Keep the candidate skills as hypotheses to pilot.

### R4-03 — Medium: The CS50 recommendation stops at intent despite available behavioral evaluation

**Location:** AI report line 206; skills report's Learning/Socratic recommendations and verification gaps. **Confidence:** high.

The distinction between intent and learning outcomes is appropriate, but the Gazette reference links to primary CS50 publications. The 2025 paper reports operational shortcomings and measured tutor-response evaluations. This is relevant programming-education evidence that the current recommendation leaves unused.

The authors report Markdown code blocks in about **22% of responses** and **48% of conversations**, despite instructions against direct solutions. These are code-block indicators, not verified rates of complete homework answers. They also describe a **50-query evaluation with 29 teaching fellows**: preferences between original and more question-led prompts varied by task. These measurements concern behavior and reviewer preference, not independent student retention. [Improving AI in CS50, sections 2.1 and 5.1](https://cs.harvard.edu/malan/publications/fp0627-liu.pdf).

**Fix:** Add this primary source alongside the interview. Say that learning-oriented tutors have measured behavioral evaluations, while causal effects on durable learning remain unestablished by this paper. Recommend checking representative tutor conversations for answer leakage, useful hints and appropriate help when stuck, and repeating the check after model or prompt changes. Define acceptable code examples before treating any code block as a failure. This directly supports a practical evaluation step without claiming that prompts or filtering guarantee the intended behavior.

## Status of round 3

| Previous finding | Current status |
|---|---|
| R3-01: Git hook coverage and dependency limits | Incorporated: selected Bash command strings, static limitations and setup requirements are explicit. Runtime behavior remains untested. |
| R3-02: FSRS implementation and mastery labels | Incorporated: heuristic scheduling and metadata are distinguished from a verified algorithm or demonstrated mastery. |
| R3-03: Nonsignificant completion time | Incorporated: the report now says the study did not establish a completion-time benefit. |
| R3-04: Math comparison, units and timing | Incorporated: relative changes versus control and the same-session exam are explicit. |
| R3-05: Copilot population and vintage | Incorporated: assignment-induced adoption, IV assumptions, standard-error units and 2022–2023 autocomplete scope are supplied. |
| R3-06: Survey denominators | Incorporated: item-level denominators replace the overall participation count. |
| R3-07: Built-in output style versus plugin | Incorporated: the mechanisms are described separately and choosing one is recommended. |

The revised Snyk discussion also retains the source's denominator and severity-label ambiguities. There is no basis here to restore round 2's categorical correction. The reports responsibly distinguish static inspection from runtime verification and missing evaluations from proof that no evaluation exists.

## Revision priority

1. Complete the physics row's scope and time-horizon fields.
2. Narrow the Socratic evidence attribution.
3. Add CS50's primary behavioral evidence and a small, repeatable conversation-review step.

No arbitrary minimum finding count was used. Earlier issues that are now qualified appropriately are not reopened merely because uncertainty remains. These changes should improve precision without expanding the reports into a comprehensive education or security survey.

## Reviewed snapshot

SHA-256 of the source reports:

```text
ae65f09ef7dfd369dd084382575d25c292105cecceb75a87bb213d753b6e16a1  ai-for-junior-devs.md
0c8d8ff03948276607ba85d42b4f36c5699b873d32d9c52a83a1b7a7171eaf5c  agent-skills-for-junior-devs.md
```

Only this review file was added. The source reports and earlier reviews were left unchanged.
