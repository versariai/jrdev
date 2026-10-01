# Adversarial review — round 7

Reviewed **2026-10-01**. Scope: the current [AI practices report](ai-for-junior-devs.md), [skills report](agent-skills-for-junior-devs.md), and [round 6](adversarial-review-round-6.md), with earlier findings checked to avoid duplication.

## Verdict

**Two low-severity follow-ups; no new medium- or high-severity finding.** Round six's corrections are substantially incorporated. Both follow-ups concern the wording of those additions. This pass found no new evidence requiring a change to the reports' main learning conclusions.

The reports now make their causal limits, package-level evidence, static inspection boundaries and proposed practices reasonably clear. Remaining uncertainty about effectiveness is not itself an error.

## Method and limits

Read both revised reports and reassess the latest corrections. Cross-check official Git ignore documentation, Claude Code hook documentation, and the pinned Learning plugin's configuration and handler. No third-party skill, hook, installer or script was executed. No external application or repository was modified.

This is a targeted document review and static source check, not a systematic literature search or runtime compatibility test. **Low** findings affect precision or usability. Confidence concerns the identified ambiguity, not whether the opposite behavior is impossible.

## Findings

### R7-01 — Low: Plugin links are placed where they imply a subagent-inheritance workaround

**Location:** skills report line 67, the bullet about ordinary subagents. **Confidence:** high for the wording ambiguity.

The revised bullet correctly distinguishes main-session output styles from ordinary subagent prompts and recommends supplying tutoring intent explicitly. It then ends with “Or use the plugins,” followed by Learning and Explanatory plugin links. In this position, the sentence reads as an alternative way to make ordinary subagents inherit tutoring behavior.

The inspected Learning plugin registers a `SessionStart` command and emits learning instructions as additional context. Its configuration does not register a `SubagentStart` handler or explicitly configure subagent prompts. That establishes its session-context mechanism, not a verified solution to ordinary-subagent inheritance. [Pinned hook configuration](https://github.com/anthropics/claude-plugins-official/blob/ab024cdcfa7ca80be204acd4907656ba5a968589/plugins/learning-output-style/hooks/hooks.json), [pinned handler](https://github.com/anthropics/claude-plugins-official/blob/ab024cdcfa7ca80be204acd4907656ba5a968589/plugins/learning-output-style/hooks-handlers/session-start.sh).

This does not show that tutoring instructions can never reach a delegated task. An explicit task prompt could carry them. It shows that the linked configuration does not substantiate the implied workaround, and no runtime inheritance test has been performed.

**Fix:** Move the plugin links back into the “Where to get it” bullet as an alternative installation route. End the subagent-scope bullet with the existing advice to specify the desired interaction and inspect the result. Keep the distinction between built-in styles and plugin context injection.

### R7-02 — Low: The ignore-check example needs its tracked-file boundary stated

**Location:** skills report line 178, the Claude-Skills onboarding row. **Confidence:** high for documented command behavior.

The tracking check is appropriate, and `git check-ignore -v .env` is useful for an untracked path. However, Git excludes tracked files from this command's output by default. If a reader first discovers that `.env` is tracked and then runs the suggested ignore check, no output does not establish that there is no matching ignore pattern. [Official Git check-ignore documentation](https://git-scm.com/docs/git-check-ignore).

**Fix:** Explain the two separate questions: use `git ls-files --error-unmatch -- .env` for current index tracking; use `git check-ignore -v -- .env` for an untracked path. To inspect which pattern would match regardless of tracking, use `git check-ignore -v --no-index -- .env`. Matching an ignore rule does not untrack a file. Run these in the intended repository and distinguish a Git error from a negative result.

The existing command is not generally incorrect. This is a clarification for the tracked-file case discussed in the same row, rather than another finding against the already-described validator defect.

## Status of round 6

| Finding | Current status |
|---|---|
| R6-01: Learning style scope | Incorporated in both reports. R7-01 concerns the placement of plugin links in the new explanation. |
| R6-02: Onboarding mechanisms and writes | Incorporated: separate rows describe a `CLAUDE.md` write and bundled Python tools, with exploration and review guidance. |
| R6-03: Validator's `.env` tracking inference | Incorporated as a known static defect, with a recommendation to check Git directly and avoid using the score as a gate. R7-02 refines the diagnostic explanation. |

The fifth-round fixes also remain present: committed-diff scope, the selected TDD implementation's deferred refactoring, and the teaching workspace are explicit.

## Checks that did not become findings

- The hook reference's event-specific decision table describes `SessionStart` as context-only with no blocking decision control. A general JSON field elsewhere in the reference is not sufficient reason to reverse that event-specific description. No new hook-enforcement finding is justified here. [Official hooks reference](https://code.claude.com/docs/en/hooks#decision-control).
- The reports retain the distinctions between immediate performance and delayed retention, throughput and learning, and tutoring-package effects and individual technique effects. This pass does not supply new causal evidence against those qualifications.
- Unverified installation commands, runtime behavior and learning effects remain declared gaps. This review does not convert those gaps into factual failures or a claim of comprehensive validation.

## Reviewed snapshot

The plugin files were inspected at `anthropics/claude-plugins-official@ab024cdcfa7ca80be204acd4907656ba5a968589`. Official documentation was checked on the review date and may change.

Source report SHA-256:

```text
d7f8054cfea66c0a953cee5c5d74dc4a534ed3bbd3ac2fa5e6e67b678830d3f8  ai-for-junior-devs.md
06cec51751dcf3ad79ba5222806e1bf297e9b947e1711d055ed6eb9f2252c229  agent-skills-for-junior-devs.md
```

Only this seventh-round review was added. Source reports and previous reviews were left unchanged. No minimum finding count was imposed.
