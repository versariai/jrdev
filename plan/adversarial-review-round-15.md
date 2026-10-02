# Adversarial review — round 15

Reviewed 2026-10-02. Scope: the round-14 remediation (`e38e561` → `de90816`) and its interactions with the post-round-12 material: C6 experiment, C2, interview consent, and supporting specs. This is a focused follow-up, not a new claim that every implementation gate has passed. No plan, experiment, or earlier report was edited.

**Result: one medium finding; no new high-severity finding established.** The four round-14 findings are addressed at the design level. A command-level fixture exposed an additional gap in C6's read-only execution contract.

## Round 14 disposition

| ID | Current design disposition |
|---|---|
| P14-01, index versus working tree | C6 now lists and reads committed, staged, unstaged, and untracked layers separately and explains what is staged for the next ordinary commit. |
| P14-02, scope before reads | Sensitive/large scope selection now precedes content reads and explicitly waits for the user's answer. Filtering remains labelled advisory. |
| P14-03, C2 composition | Tool sets now agree, check-ins occupy row 4a before ordinary Allow exits, and Bash has a separate decision order. The off/status and exception-preservation requirements remain intact. |
| P14-04, public interview outputs | Consent now distinguishes internal paraphrases, optional public paraphrases, optional quotes, and recording; aggregate publication is disclosed and synthesis checks current scopes. |

These dispositions assess written contracts. This review did not execute a tutor session, check-in hook, interview, or collection workflow.

## P15-01 — Medium: ordinary Git diff commands can execute configured helpers

**Evidence:** [C6 skill](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) promises read-only operation and forbids edits or tests unless separately requested. Its content commands use ordinary `git diff` invocations for each layer without disabling external helpers. [README](../experiments/c6-explain-changes/README.md) presents the Git inspection as read-only.

Git documents that text-conversion filters are enabled by default for `git diff` and provides `--no-textconv` and `--no-ext-diff` to suppress conversion filters and external diff drivers. Thus selecting a nominally read-only Git subcommand does not establish that no configured helper runs. [Official git-diff documentation](https://git-scm.com/docs/git-diff).

**Verified counterexample:** in a temporary repository, `.gitattributes` assigned `app.txt` a diff driver, and the local Git configuration assigned that driver a text-conversion script. The script wrote an innocuous `converter-marker` file and printed the input. With a staged change:

| Command path | Marker created? |
|---|---|
| Statistics-only `git diff --cached --stat` | No |
| Prescribed content read, `git diff --cached -- app.txt` | **Yes** |
| `git diff --no-ext-diff --no-textconv --cached -- app.txt` | No |

The final command still reported the source change. The fixture and marker were removed afterwards; no actual project configuration was changed.

**Failure case:** a participant has a configured converter that refreshes generated files, reads other local data, or performs another side effect. C6 invokes it while summarizing an allowed path, despite having neither requested nor disclosed that execution. Filtering secret-like paths does not constrain what a helper can access or print. No malicious repository is required: an ordinary configured tool can violate the skill's stated read-only behavior.

**Required change:** disable external diff drivers and text conversion explicitly on every C6 diff path, including statistics and committed/staged/unstaged content commands. If a useful conversion is wanted, require a separate, disclosed decision rather than running it by default. State that the unconverted view may be less useful for some formats, and keep binary files in the existing “not read” disclosure. Review the inspection command set for other configured helper execution rather than equating “Git command” with “no external execution.” This finding does not claim the two diff flags sandbox the whole Git client.

**Acceptance before participant rollout:** configure harmless converters/external diff helpers that write marker files in a synthetic repository. Exercise all three diff layers and the pre-content statistics phase. Default C6 inspection produces no markers and still reports normal source diffs. Any separately authorized conversion is identifiable in the scope/output disclosure. Keep actual participant configuration out of this test.

## Coverage and limits

Read the remediation diff and compared the revised source hashes with round 14. The checkpoint-ledger design was not the focus of this pass. The post-round-12 team, interview, candidate, and experiment contracts retain their prior dispositions except for the finding above.

The text-conversion side effect was reproduced with Git, not by invoking the skill through an AI model. Official documentation was consulted for the helper behavior and suppression flags. No participant rollout, secret-filtering guarantee, hook resource budget, privacy deletion, or consent operation was tested. The existing experiment readiness hold and implementation acceptance gates remain applicable.

## Reviewed snapshot

SHA-256 identifies the reviewed source artifacts and supporting contracts.

| Source | SHA-256 |
|---|---|
| `plan/jrdev-ai-plan.md` | `d4d007d3156ef01fcfbd68aafab5223c7f948f5534946decb3d0e42fc0193391` |
| `plan/specs/01-website-and-feedback.md` | `37d0e20509044495eb07edcf8f656ccb3ca1bd0ff7bff5bd571688fb003e41dd` |
| `plan/specs/02-commands-and-policy.md` | `c160baa1eb6cf0964cc2aaf8b51b805983a930d5694504e929c435151c8ef6b1` |
| `plan/specs/03-modes-and-workflows.md` | `21e6f3d81d7ff1d4439eab002a3838c3df0a45d557da7727c388d665f0fe5dc1` |
| `plan/specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `plan/specs/05-data-flows-and-privacy.md` | `a79db9644cb3f960913db69572a16daf92e44f0cb9f97314f0c8a4b1d446ffd4` |
| `plan/specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `plan/specs/07-evaluation-and-studies.md` | `e2800b93f6907e1300e39fdaf2135f94f1368159ee192411678d586d44187750` |
| `plan/phase0/interview-guide.md` | `28af59c497d3405fd6e5a14e49be28310b29a0e4922d01e86ebb680d3b210b93` |
| `experiments/c6-explain-changes/CLAUDE-snippet.md` | `0d26a58752d2735324a9dcb54f9cf164dd1c1124c8863e22eda57d98f97e86a4` |
| `experiments/c6-explain-changes/README.md` | `75085002a1ba6eff53b994b80082eddf8da9f04f229b19611e61c8d3975a7fa2` |
| `experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md` | `10345d2f971e28cb4fa663bd86d631c3588a892278a0471b10433aebc4a2a914` |
