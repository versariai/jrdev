# Adversarial review — round 17

Reviewed 2026-10-02 at commit `7d0c799`. Scope: the complete round-16 remediation (`c7d2544` → `7d0c799`) and the current C6 skill's inspection, helper exclusion, scope and error contracts. This is a focused follow-up on the experiment, not a fresh audit of every unchanged plan spec.

**Result: two medium findings and a correction to round 16.** No plan, experiment or earlier report was edited.

## Round 16 disposition and correction

**P16-01's overescaped-regex claim is withdrawn.** Extracting the exact expression from the committed `c7d2544` source yields one backslash before each dot. Executing that expression in an isolated fixture detects both `filter.fixture.clean` and `filter.fixture.process` (exit 0, two matches). Round 16 tested an incorrectly reconstructed two-backslash expression rather than the source command. Its claimed default helper execution caused by the regex is therefore unsupported. The overview's correction is accurate. The separate recommendation to distinguish no-match from errors has been adopted.

**P16-02 is addressed in the instructions:** unstaged statistics now run for eligible paths regardless of filter presence, empty sets are skipped explicitly, and unknown sizes trigger scope selection. The new path-selection contract nevertheless has the independently reproduced gap below.

## P17-01 — Medium: an explicitly listed safe filename is still a Git pattern

**Evidence:** [C6 skill, steps 2.2 and 2.5](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) requires passing each safe unstaged path explicitly to statistics/content commands. It does not require literal pathspec interpretation. Quoting a filename for the shell does not disable Git's own pattern matching.

**Verified counterexample:** an isolated temporary repository contained modified `[a].py` and `a.py`. Attribute checking classified `[a].py` as unfiltered and `a.py` as filtered. A harmless clean filter for `a.py` wrote a marker and copied stdin. Only `[a].py` was passed as the safe path, as one argument with no shell expansion:

| Inspection | Observed result |
|---|---|
| `git -c core.fsmonitor=false diff --no-ext-diff --no-textconv --stat -- '[a].py'` | Both files included; filtered file's helper wrote the marker |
| Same command with global `--literal-pathspecs` | Only `[a].py` included; no marker |

**Failure case:** a valid source filename containing pattern syntax broadens the purported safe set. A filtered or user-declined file can consequently be compared, executing its helper or sending its content to the model despite the preceding selection. The same interpretation matters when converting declined filenames into exclusion pathspecs. This finding is independent of the corrected filter-detection regex.

**Required change:** make allowlisted filenames literal in both unstaged statistics and content reads, for example using Git's global `--literal-pathspecs` with a pure allowlist. For commands intentionally using exclusion magic, use a compatible construction that treats each filename literally; do not globally disable magic while expecting `:(exclude)` to work. Preserve the empty-set guard.

**Acceptance:** test exact argument construction with bracket, wildcard and leading-colon filenames, including a filtered sibling and a user-declined sibling. Both statistics and content commands cover only selected files; no helper marker is produced for excluded files. Ordinary filenames still work.

## P17-02 — Medium: the new stop-on-failure rule makes documented fallback paths unreachable

**Evidence:** [C6 skill, section 1](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) now says any Git command failure requires stopping, with only the step-2 no-match exit exempted. Immediately below, base detection explicitly falls back when `origin/HEAD` lookup fails, tries potentially absent refs, and supports a missing upstream. Step 2 also explicitly provides a scoped fallback for filter-check errors. These instructions prescribe incompatible responses to the same failures.

**Verified counterexample:** in an isolated repository with one commit on local `main`, no remote and no upstream:

| Probe | Exit |
|---|---|
| `git rev-parse --abbrev-ref origin/HEAD` | 128 |
| `git rev-parse --verify --quiet origin/main` | 1 |
| `git rev-parse --verify --quiet main` | 0 |
| `git rev-parse --verify --quiet @{u}` | 1 |

A strict execution of the new rule stops at the first probe instead of reaching the valid local base and the documented uncommitted-only summary. A model that follows the local fallbacks instead must disregard the global rule. This is an instruction regression, not an observed model-session failure.

**Required change:** distinguish expected negative probes from inspection failures. Explicitly permit the documented absent-base and absent-upstream outcomes and define precedence for the filter-error fallback. Keep genuine failures in content/statistics commands from being presented as empty changes. Do not broadly ignore every nonzero exit.

**Acceptance:** repositories with no remote HEAD, only a local base, and no upstream reach their documented fallback; a malformed configuration or genuine diff failure is surfaced and never summarized as “no changes.” A filter-check error follows one unambiguous documented policy.

## Coverage and limits

Read the remediation diff and the complete current C6 skill. Command fixtures used disposable repositories with global/system configuration isolated; no participant or project settings were changed. The regex correction was verified against the exact historical source. The filename case reproduced helper execution directly. No AI-driven skill session, participant rollout, privacy lifecycle or hook implementation was executed. Existing readiness and implementation gates remain applicable.

## Reviewed snapshot

SHA-256 identifies the source snapshot; unchanged artifacts are recorded for comparison, not claimed as fully reaudited.

| Source | SHA-256 |
|---|---|
| `plan/jrdev-ai-plan.md` | `4e1ff5498c8f5fe60f4c21b5169f99469d53dabd6215ff2e19824ca6f2f7a55c` |
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
| `experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md` | `cab97279724739889ae2d005284ad822f753a668889b107395fa028559d88238` |
