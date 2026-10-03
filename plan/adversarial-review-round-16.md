# Adversarial review — round 16

Reviewed 2026-10-02 at commit `c7d2544`. Scope: the round-15 remediation (`de90816` → `c7d2544`), especially its interaction with C6's read-only promise and scope-before-content rule. The current overview and interview guide were also reread for context. This is a focused follow-up on the post-round-12 material, not a repeat certification of the core design.

**Result: two medium findings; no new high-severity finding established.** Both findings concern the revised C6 instructions. Plan and experiment artifacts were not edited.

## Round 15 disposition

P15-01 is **partially addressed**. The skill now explicitly disables fsmonitor, external diff drivers and text conversion, replaces `git status`, and attempts to exclude clean-filtered working files. The README and candidate description disclose these controls and their limits. However, the filter-detection command fails on ordinary configuration (P16-01), so the new default path still permits helper execution. The inspection changes also omit unstaged statistics in the no-filter branch (P16-02).

## P16-01 — Medium: the clean-filter detection regex is overescaped

**Evidence:** [C6 skill, step 2.2](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) prescribes:

```sh
git config --get-regexp '^filter\\..*\\.(clean|process)$'
```

The single-quoted shell argument preserves both backslashes before each dot. This is not the expression that matches a literal dot in an ordinary key such as `filter.fixture.clean`. When the command prints nothing, the instructions explicitly skip attribute checking and continue towards the unstaged content diff.

**Verified counterexample:** a temporary repository had `app.txt filter=fixture` in `.gitattributes` and `filter.fixture.clean` configured to a harmless script that wrote `filter-marker` and copied stdin. After modifying `app.txt`:

| Operation | Observed result |
|---|---|
| Prescribed two-backslash regex | Exit 1, no match |
| Same regex with one backslash before each dot | Exit 0, filter matched |
| Unstaged diff with `core.fsmonitor=false`, `--no-ext-diff` and `--no-textconv` | Marker created |

The existing diff flags do not suppress that clean filter. Thus a normal configured helper can run without the separate user request required by the new contract. This is a concrete failure in the round-15 fix, rather than a hypothetical demand to sandbox Git.

**Required change:** use the shell-ready expression `'^filter\..*\.(clean|process)$'` (one backslash before each dot), or another verified detection method. Distinguish no matches from a command failure instead of treating every empty result as “no filter.” Keep filtered paths out of default unstaged statistics and content reads.

**Acceptance:** execute the exact command copied from the Markdown in a fixture with configured `clean` and `process` entries. Both are detected; matching filtered paths are skipped; default inspection leaves harmless helper markers absent. Include a no-filter fixture. Do not validate only a separately rewritten regex.

## P16-02 — Medium: no-filter repositories skip unstaged statistics before the scope decision

**Evidence:** [C6 skill, steps 2.1–2.4](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) now lists unstaged paths with `git diff-files --name-only`. The only unstaged statistics command is inside the filter-check substep, after the instruction “If this prints nothing, no filter is configured; skip to substep 3.” Step 2.4 still requires a scope question before content reads for roughly 2,000 or more changed lines across all layers.

**Verified counterexample:** in an isolated temporary repository with global/system configuration disabled, no filters, no staged changes and a 2,500-line insertion in ordinary `app.py`, filter detection returned exit 1 and the prescribed unstaged listing returned only `app.py`. A separate protected statistics command confirmed 2,500 insertions. Following the written no-filter branch provides no unstaged line count to the large-change decision. The path is also not a configuration-file trigger, so the instructions permit proceeding directly to its content diff.

**Failure case:** a participant's large uncommitted change is read into the model context before they are offered the promised scope selection. This persists even after correcting P16-01: the no-filter branch still skips statistics.

**Required change:** compute protected unstaged statistics for the eligible path set whether filters are configured or not, before the scope decision. If that set is empty, skip the command explicitly rather than expanding an empty path list into an unrestricted diff. If statistics cannot be obtained safely, disclose the unknown size and ask for scope before content reads.

**Acceptance:** a no-filter fixture with 2,500 unstaged changed lines requires the scope question before any content read; a small change obtains its count and proceeds normally. Repeat with mixed filtered/unfiltered paths and with all paths filtered, confirming that excluded helpers never run.

## Coverage and limits

Read the complete round-15 remediation diff, current C6 skill and README, and current overview/interview guide. Both command-level cases used disposable synthetic repositories; fixtures were removed and no participant repository or configuration was modified. The no-filter case explicitly isolated global/system configuration. No AI-driven skill session, participant collection, hook implementation or privacy lifecycle was executed. The existing participant-readiness hold remains applicable.

## Reviewed snapshot

SHA-256 values below identify the source snapshot; hashes do not imply that unchanged specs received another complete line-by-line audit.

| Source | SHA-256 |
|---|---|
| `plan/jrdev-ai-plan.md` | `e614971cb2f9d507efb53a6cc7084f24392790ed4b9a1c702916283d439f980d` |
| `plan/specs/01-website-and-feedback.md` | `37d0e20509044495eb07edcf8f656ccb3ca1bd0ff7bff5bd571688fb003e41dd` |
| `plan/specs/02-commands-and-policy.md` | `c160baa1eb6cf0964cc2aaf8b51b805983a930d5694504e929c435151c8ef6b1` |
| `plan/specs/03-modes-and-workflows.md` | `bdb7b281aa1c5c76570032fc431ef0b24b1acc6db52cd6957c0453f48ed4fdba` |
| `plan/specs/04-assessment-and-grading.md` | `bd9f8a0a3e1b333e757fea47815c9b303ef3f8680b9986964badbf1024f21ed6` |
| `plan/specs/05-data-flows-and-privacy.md` | `a79db9644cb3f960913db69572a16daf92e44f0cb9f97314f0c8a4b1d446ffd4` |
| `plan/specs/06-repo-and-release-trust.md` | `7fa26a4961f0908161053c73034b044c4be3ed1700eb48e03e4d394758082bc5` |
| `plan/specs/07-evaluation-and-studies.md` | `e2800b93f6907e1300e39fdaf2135f94f1368159ee192411678d586d44187750` |
| `plan/phase0/interview-guide.md` | `28af59c497d3405fd6e5a14e49be28310b29a0e4922d01e86ebb680d3b210b93` |
| `experiments/c6-explain-changes/CLAUDE-snippet.md` | `0d26a58752d2735324a9dcb54f9cf164dd1c1124c8863e22eda57d98f97e86a4` |
| `experiments/c6-explain-changes/README.md` | `e99677026e2b09904000fc41517b39bb307cd74ba972f660b5269b0cf60db401` |
| `experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md` | `8395e2ded6d465033df4bc38f06c6af48aba52f62bbe4022a03a090ae9c060aa` |
