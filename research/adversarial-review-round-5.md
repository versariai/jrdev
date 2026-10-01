# Adversarial review — round 5

Reviewed **2026-10-01**. Scope: the current [AI practices report](ai-for-junior-devs.md), [skills report](agent-skills-for-junior-devs.md), and [round 4](adversarial-review-round-4.md), with earlier reviews checked for duplicate findings.

## Verdict

**Two new medium-severity findings and one low-severity clarification; no new high-severity finding.** The fourth-round corrections are incorporated. The remaining actionable problems concern matching the proposed workflow to the actual skill instructions, rather than another broad objection to the learning evidence.

The main correction is to distinguish review of committed branch changes from review of the working tree. The report should also describe the selected TDD implementation accurately and explain where the teaching skill writes its material.

## Method and limits

Read both revised reports. Statically inspect additional skill files at `mattpocock/skills@d81f3a183412e71a5b1e84ca21bc1a35eea03a60`, the same revision used by earlier reviews: `skills/engineering/tdd/SKILL.md`, `skills/engineering/tdd/tests.md`, `skills/engineering/code-review/SKILL.md`, `skills/productivity/teach/SKILL.md`, and its learning-record format. Cross-check selected GitHub mentoring instructions and the DORA and Veracode publisher summaries. Consult official Git documentation for diff scope.

These external skill files were read as evidence, not invoked as instructions for this review. No package was installed, no third-party script or hook was executed, and no external repository was modified. Findings about a prescribed command or workflow are static observations, not runtime tests of whether an agent follows the instructions. This remains a targeted review, not a systematic search or comprehensive security audit.

**Medium** materially affects application or interpretation; **Low** improves precision or usability. Confidence refers to the mismatch identified. Locations refer to the source snapshot recorded below.

## Findings

### R5-01 — Medium: The recommended review skill's prescribed diff excludes uncommitted work

**Locations:** skills report lines 124, 128 and 203–205; companion AI report's before-committing checklist. **Confidence:** high for command scope.

The report recommends `code-review` after the learner's own diff review, without explaining its input boundary. Matt Pocock's inspected implementation requires a fixed point and prescribes `git diff <fixed-point>...HEAD`. It checks that this diff is nonempty before continuing. It also depends on issue-tracker configuration and runs separate standards and specification reviews. [Pinned skill](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/engineering/code-review/SKILL.md).

That three-dot command compares the merge base with the committed `HEAD` tree. It does **not** include staged changes, unstaged changes or untracked files. [Git diff documentation](https://git-scm.com/docs/git-diff).

**Concrete consequence:** a developer edits code and invokes this skill before committing. If the branch already has commits, the prescribed command reviews those committed changes while omitting the new edit. If there are no committed changes since the selected base, its nonempty-diff check stops the workflow. The frontmatter mentions work in progress, but its prescribed process does not implement working-tree coverage.

This is a mismatch between an easy reading of the recommendation and the implementation, not proof that every agent would omit the edit: an agent might depart from the prescribed command. Nor does the report explicitly promise a pre-commit gate. It nevertheless needs the boundary stated so readers can choose the appropriate review.

**Fix:** Label this package as a committed-branch review workflow requiring a base reference and configuration. For pre-commit review, use or adapt a workflow that explicitly includes staged, unstaged and relevant untracked content, and verify the reviewed file list. Do not advise committing merely to make this particular skill work. Keep Claude Code's bundled `/code-review` separate; this finding concerns Matt Pocock's package.

### R5-02 — Medium: The selected `tdd` implementation is inaccurately summarized as a refactoring loop

**Locations:** skills report lines 49 and 120. **Confidence:** high for the inspected revision.

The table calls Matt Pocock's skill a red-green-refactor workflow. Its instructions instead define a red/green loop, require agreed public test boundaries before writing tests, and explicitly move refactoring to the review stage. [Pinned TDD skill](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/engineering/tdd/SKILL.md).

The intended effect is not merely terminological: the package asks the agent to defer a phase that the report says is part of its loop. The generic teaching-method row also groups this implementation with another package under one description, hiding possible implementation differences.

**Fix:** Describe this revision as “one failing behavior test, then minimal implementation, at agreed public boundaries; refactoring deferred to review.” Keep red-green-refactor as a general practice if desired, but do not attribute that exact loop to this implementation. No claim about whether one sequence teaches better is justified by this inspection.

### R5-03 — Low: `teach` needs a concrete workspace description

**Locations:** skills report lines 88–90 and 184. **Confidence:** high for file-writing intent; learning effectiveness remains unknown.

The one-line README summary is broadly accurate, but it leaves out what using the current directory as a teaching workspace means. The actual skill plans learning across sessions and writes mission/resource notes, learning records, HTML lessons, reference documents and shared assets into that directory. It requests retrieval, spacing and interactive feedback, rather than being only a chat persona. [Pinned teaching skill](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/productivity/teach/SKILL.md).

**Fix:** Add a short static description and recommend a dedicated learning directory when these artifacts do not belong in the application repository. Retain “this review found no learning-outcome evaluation” rather than the table's categorical “Not evaluated.” Workspace state and lesson production do not demonstrate retained skill.

## Status of round 4

| Finding | Current status |
|---|---|
| R4-01: Physics comparison and assessment horizon | Incorporated: sample, crossover, two lessons, post-lesson tests and package-level interpretation are explicit. |
| R4-02: Socratic mechanism attribution | Incorporated: the reports distinguish package evidence from the unisolated questioning component and untested skills. |
| R4-03: CS50 behavioral evidence | Incorporated: the primary publication, response evaluations and a conversation-review step are included. The AI report distinguishes code blocks from verified full solutions. |

Selected checks produced no additional finding: the DORA summary supports the reported directions of throughput and stability associations; Veracode's summary supports the 45% and 86% benchmark headlines; the mentoring skill includes the described PEAR loop and escalation, and the mentor agent requests no code edits. These checks do not validate learning effects or runtime enforcement. [DORA publisher summary](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report), [Veracode publisher summary](https://www.veracode.com/blog/genai-code-security-report/), [mentoring instructions](https://github.com/github/awesome-copilot/blob/main/skills/mentoring-juniors/SKILL.md), [mentor agent](https://github.com/github/awesome-copilot/blob/main/agents/mentor.agent.md).

## Revision priority

1. State and verify which changes the chosen review workflow includes.
2. Correct the TDD implementation description.
3. Explain the teaching workspace and extend the static-verification inventory with the files inspected this round.

No fixed number of findings was required. Earlier issues are not reopened merely because the reports still contain uncertainty. This pass found no new basis to strengthen or reverse their causal learning conclusions.

## Reviewed snapshot

SHA-256:

```text
c812640b34b949451b09c2302453b21d370765f503d957170c32fda8c13a9436  ai-for-junior-devs.md
c3d50a7596c0c78fb7e78adc9ff9e9842900b25b2b3974469e1fc0ed77709a19  agent-skills-for-junior-devs.md
```

Only this fifth-round review was added. Source reports and previous reviews were left unchanged.
