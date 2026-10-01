# Adversarial review — round 8

Reviewed **2026-10-01**. Scope: the current [AI practices report](ai-for-junior-devs.md), [skills report](agent-skills-for-junior-devs.md), and [round 7](adversarial-review-round-7.md), with the earlier review history used to distinguish unresolved issues from repetitions.

## Verdict

**No new actionable finding in this pass.** The two round-seven clarifications are incorporated. This does not certify every citation or package, but further wording objections from the checks below would add little value. No minimum finding count was imposed.

The reports now distinguish proposed practices from validated interventions, immediate assessment from retention, productivity from understanding, and static inspection from runtime validation. This pass found no new evidence requiring those conclusions to be strengthened or reversed.

## Method and boundaries

Read both current reports, check the latest corrections, and cross-check selected official documentation and pinned plugin configuration. Revisit the outstanding GitClear number-verification gap using its publisher page and targeted searches. Check local Markdown file links and source hashes.

No third-party skill, hook, installer or downloaded script was executed. No skill was installed, no application tests were run, and no external repository was modified. The command examples were checked against documentation rather than rerun against the user's repository. This was a targeted review, not a systematic literature search, complete citation audit, security audit or runtime compatibility test.

## Round-seven status

| Previous finding | Result |
|---|---|
| R7-01: Plugin links implied a subagent-inheritance workaround | Addressed. Links now appear under installation routes. The subagent note describes the pinned Learning plugin's session hook and explicitly says inheritance was not tested. |
| R7-02: Ignore-check example omitted tracked-file behavior | Addressed. The onboarding row distinguishes tracking from ignore-pattern checks, explains the default tracked-file exclusion, supplies `--no-index`, and distinguishes errors from negative results. |

The Git explanation agrees with the documented defaults: `git ls-files` normally lists index-tracked files, while `git check-ignore` omits tracked files unless `--no-index` is used. A matching ignore rule does not establish that a file is untracked. [Git ls-files documentation](https://git-scm.com/docs/git-ls-files), [Git check-ignore documentation](https://git-scm.com/docs/git-check-ignore).

The pinned plugin configuration registers a `SessionStart` command. It does not establish an automatic ordinary-subagent tutoring mechanism. The report now presents this as a configuration observation with an untested inheritance boundary. [Pinned Learning plugin configuration](https://github.com/anthropics/claude-plugins-official/blob/ab024cdcfa7ca80be204acd4907656ba5a968589/plugins/learning-output-style/hooks/hooks.json), [official output-style scope](https://code.claude.com/docs/en/output-styles#how-output-styles-work).

## Other conclusions reassessed

- **Evidence interpretation:** small self-selected interaction groups remain associations; tutoring trials remain evidence for packages in their studied settings; the Copilot estimate remains scoped to assignment-induced adoption and autocomplete-era throughput.
- **Workflow scope:** committed-branch review is distinguished from working-tree review. The selected TDD implementation and teaching workspace are described separately. The onboarding packages' file writes and scripts remain explicit.
- **Practical advice:** the reports label schedules and habits as adaptable proposals. Independent checks and human mentoring remain part of the suggested workflows, without a claim that any listed skill has demonstrated durable learning gains.
- **Security claims:** selected hook coverage, dependency limits, scanner findings and confirmed attacks remain distinguished. The onboarding validator defect is identified as a static observation, not an executed security test.

These are document-consistency observations. They do not mean every underlying study or external implementation was independently revalidated during this round.

## Gaps that remain open

| Gap | What this round establishes |
|---|---|
| Exact GitClear 3.1%–5.7% churn figures | The accessible publisher page supports the dataset size and broad duplication/refactoring trends but does not expose those exact figures. Search results repeating them do not independently verify the primary calculation. No finding that the numbers are false is justified. [Publisher page](https://www.gitclear.com/ai_assistant_code_quality_2025_research). |
| Learning outcomes for the listed skill packages | No new evaluation was identified in this targeted pass. That is not proof that none exists. |
| Runtime behavior, installation and cross-agent compatibility | Still untested here. Static configuration and instructions do not establish successful operation. |
| Source selection and long-term transfer | The reports remain a non-systematic synthesis; this pass adds neither a systematic search nor longitudinal coding evidence. |

No additional source edits are required by this round's findings. These gaps would need different verification work rather than repeated editorial review of the same claims.

## Reviewed snapshot

Official documentation was checked on the review date. The plugin configuration was read at `anthropics/claude-plugins-official@ab024cdcfa7ca80be204acd4907656ba5a968589`.

Source report SHA-256:

```text
a803192458dc8ddba92929790d6ede7ea083ba67418dfed44d62a76a859905f9  ai-for-junior-devs.md
93666315b42f9e7b16974d601edae00c68c28167cdce466309d6133e6ce17126  agent-skills-for-junior-devs.md
```

Only this eighth-round review was added. Source reports and previous reviews were left unchanged.
