# Agent Skills With Possible Value for Junior Developers

*Compiled 2026-10-01 and revised the same day after two adversarial reviews ([round 1](adversarial-review.md), [round 2](adversarial-review-round-2.md)). Companion to [ai-for-junior-devs.md](ai-for-junior-devs.md).*

"Skills" here means **agent skills**: `SKILL.md` folders and plugins for Claude Code, plus similar mechanisms in Copilot, Codex and other tools.

**What this report can and can't tell you**
- **Descriptions come from each project's README or SKILL.md** as published on the date above. **No skill was installed, run or security-audited** for this report.
- **This review did not find published learning-outcome evaluations of these specific packages.** The search was not systematic: web searches, the skills.sh registry and the projects' READMEs, up to 2026-10-01. Some of the *methods* these skills use (e.g. retrieval practice, spaced repetition, tutors with learning safeguards) have research support in other settings. That is evidence for the method, not for a particular SKILL.md. Labels like "intended learning support" describe design intent.
- **Popularity numbers are snapshots.** Install counts come from [skills.sh](https://www.skills.sh/) and stars from GitHub, both on 2026-10-01. Installs are not unique users, active use or a measure of quality. Stars and installs are different units, so don't compare them directly.
- **Instructions are not enforcement.** Most skills are *instructions* the agent may or may not follow. A hook running automatically does not make it a policy either. Only some hook events can block an action (e.g. `PreToolUse`), and only when configured to. Platform permissions and OS-level sandboxing are the main enforced boundaries. See [section 8](#8-installing-and-running-skills-safely).

---

## TL;DR

- **Several skills are *designed* to keep the human thinking.** They include Socratic mentors, "grill me" design interviews, quizzes on your own diffs, and Anthropic's Learning output style. This review found no learning-outcome evaluations of these packages.
- **The registry's most-installed skills are mostly delivery and productivity skills.** Matt Pocock's [`mattpocock/skills`](https://github.com/mattpocock/skills) is a notable exception: process skills like `grill-me` (about 1.3M installs), `tdd` (about 1M), `teach` (about 739K) and `diagnosing-bugs` (about 696K) rank highly.
- **A reasonable first experiment:** Anthropic's **Learning output style**, where the agent leaves 5–10 lines of design or business logic for you to write. Of the tools here, it's the closest to the interaction patterns *associated* with better immediate quiz scores in the [Anthropic study](https://www.anthropic.com/research/AI-assistance-coding-skills). Those patterns were exploratory and not randomized.
- **Start small:** one tutoring workflow plus one check of understanding that doesn't depend on the AI, rather than a large kit.
- **Supply-chain risk is real.** Snyk's ToxicSkills study (corpus of 3,984 skills, Feb 2026, reported mainly as **ClawHub**; see section 8) found **13.4%** with critical-severity findings and **76 manually confirmed malicious payloads**. Its curated top-100 skills.sh baseline had no critical findings. Read third-party skills before installing them and run them with minimal permissions. ([Snyk ToxicSkills](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/))

---

## Design hypothesis: how skills relate to the research

This mapping is **this report's hypothesis**. Nothing here shows that a skill reproduces a pattern, or that the pattern causes learning.

### A. Patterns observed in the Anthropic coding study (the *human* is the actor)
In the [study](ai-for-junior-devs.md#b-interaction-patterns-associated-with-better-immediate-understanding) these were behaviors participants **chose themselves**. They were associated with higher immediate quiz scores in a post hoc analysis.

| Observed human behavior | Tool support that leaves the human in charge | Notes |
|---|---|---|
| **Conceptual inquiry:** *the learner* asks conceptual questions and writes the code themselves | Any assistant used in a "questions only, no code" way; Learning output style (the human writes the key lines) | No special skill is needed. It's a usage habit. |
| **Code + explanation:** *the learner* asks for code together with its reasoning | Explanatory / Learning output styles | The agent adds explanations by default. Whether the learner engages with them isn't guaranteed. |
| **Generate, then comprehend:** *the learner* asks follow-up questions about generated code | Any assistant; `quiz-me` can prompt the follow-up step | `quiz-me` *reverses* the actor: the agent asks. That is related, but it's not what the study observed. |

### B. Additional proposed supports (not among the patterns the study observed)
These come from wider pedagogy and engineering practice. A **Socratic tutor that questions the learner** is a *related* design idea, supported in other domains by the safeguarded-tutor results in math and physics ([see report 1](ai-for-junior-devs.md#1-what-the-evidence-says)). It is not a direct implementation of "conceptual inquiry".

| Proposed support | What the skill is designed to do | Candidate skills |
|---|---|---|
| Socratic tutoring (agent asks, holds back answers, escalates gradually) | Keep the learner doing the reasoning, with a way out when stuck | mentoring-juniors, socratic-skills `guide-me`, socrates-skill |
| Retrieval practice / spaced repetition | Quiz on concepts over time | agent-tutor-skill, `quiz-me` |
| Design interview before coding | Agent interviews the human about the plan | grill-me, grill-with-docs, superpowers `brainstorming` |
| Structured debugging | Hypothesis-driven debugging loop | diagnosing-bugs, superpowers `systematic-debugging` |
| Test-first workflow | Red-green-refactor | tdd, superpowers `test-driven-development` |
| Security checks | Flag insecure defaults and footguns | trailofbits `insecure-defaults`, `sharp-edges` |

**Known failure mode:** an AI-written quiz, explanation or test can repeat the same mistake as the AI-written code. Check against something independent: the original requirement, the docs, a human reviewer, or the code's actual behavior.

---

## 1. Learning and tutoring skills (intended learning support)

### Anthropic Learning and Explanatory output styles
- **What it does:**
  - **Learning:** at decision points the agent leaves a `TODO(human)` for you to write 5–10 lines of design or business logic, and adds "Insights" about trade-offs.
  - **Explanatory:** adds 2–3 short insights about implementation choices and codebase patterns.
- **How it works:** an output style is instructions added to the agent's system prompt. The plugin versions inject the same instructions through a **SessionStart hook**. Either way it is **advisory**: the agent is asked to behave this way, not forced to.
- **Where to get it:** set an output style in Claude Code (`/config → Output style` as of 2026-10; check the [output styles docs](https://docs.anthropic.com/en/docs/claude-code/output-styles)). Or use the plugins: [learning-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) and [explanatory-output-style](https://github.com/anthropics/claude-code/tree/main/plugins/explanatory-output-style).
- **Verification:** descriptions are from Anthropic's repos and docs. A public post by an Anthropic employee recommends it for staying hands-on ([X post](https://x.com/lydiahallie/status/2056420694087594283)). That post was not independently authenticated.

### `mentoring-juniors` ("Sensei") from [github/awesome-copilot](https://github.com/github/awesome-copilot/blob/main/skills/mentoring-juniors/SKILL.md)
- **What it does:** a Socratic mentor aimed at junior devs. It uses the **PEAR loop**: plan (pseudocode first), explore, analyze (read every line), rewrite. Help escalates gradually, from guiding questions through pseudocode and step-by-step help, ending with handing off to a human.
- **How it works:** prompt instructions, available as a SKILL.md or a [Copilot agent](https://github.com/github/awesome-copilot/blob/main/agents/mentoring-juniors.agent.md). Advisory.
- **Note:** the gradual escalation addresses a real risk. Productive struggle needs feedback and a way out, and holding back answers for too long can stall a beginner.
- **Related:** [`mentor.agent.md`](https://github.com/github/awesome-copilot/blob/main/agents/mentor.agent.md), a mentor mode with no edits.

### [`rodbv/socratic-skills`](https://github.com/rodbv/socratic-skills) (about 17★, MIT)
- **`quiz-me`** questions you on your diff, spec or plan, one question at a time.
- **`guide-me`** walks you through implementing a spec with hints instead of copy-paste code.
- **How it works:** prompt instructions. Advisory.
- **Use with care:** one useful check among several. **Don't use it as the only merge gate**, and don't run it on every commit by default. An AI quiz can share the code's blind spots.
- **Install:** the README shows `npx skills add rodbv/socratic-skills/skills/quiz-me`. Not tested here.

### [`Bhala-Srinivash/agent-tutor-skill`](https://github.com/Bhala-Srinivash/agent-tutor-skill) (about 10★, MIT)
- **What it does:** a teaching loop with FSRS spaced repetition, quizzes and per-concept mastery tracking stored in `~/.learn/`.
- **How it works:** prompt instructions plus files written to your home directory. Review what it writes before installing.
- **Note:** spaced repetition and retrieval practice are well supported in learning research in general. This particular implementation has not been evaluated.

### `teach` from [`mattpocock/skills`](https://github.com/mattpocock/skills) (about 739K installs)
- **What it does:** "teach new skills/concepts using the directory as a workspace" (from the README).
- The same repo lists **`scaffold-exercises`** (about 402K installs) under Learning. Its behavior was **not verified**.

### Other Socratic skills (small; not verified beyond their READMEs)
- [`bevibing/socrates-skill`](https://github.com/bevibing/socrates-skill)
- [`malkreide/socratic-method-skill`](https://github.com/malkreide/socratic-method-skill) (written in German)
- [`GitExcited/Socratic`](https://github.com/GitExcited/Socratic), a VS Code extension

---

## 2. Skills for planning before coding

### `grill-me` and `grill-with-docs` ([mattpocock/skills](https://github.com/mattpocock/skills); about 1.3M and 1.1M installs)
- **What it does:** the agent interviews you about your plan before any code is written. `grill-with-docs` also maintains a GLOSSARY.md and ADRs.
- **How it works:** prompt instructions. Advisory.
- **Use with care:** "resolve every branch" can turn into an endless interview. **Limit it** to the decisions that matter, especially for beginners and small tasks.

### `brainstorming` and `writing-plans` from [`obra/superpowers`](https://github.com/obra/superpowers)
- **What it does:** turns a rough idea into a design doc and a plan, with an emphasis on TDD and YAGNI.
- **Use with care:** Superpowers can also run long autonomous multi-agent work. For a learner, that can drift into handing the whole task over.

### [`andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills) (about 216K★; formerly `forrestchang/…`, now redirects to `multica-ai/…`)
- **What it does:** four guidelines for the agent: think before coding, simplicity first, surgical changes, and goal-driven execution.
- **Note:** these shape the *agent's* behavior, not the human's learning. Smaller, simpler diffs may be easier for juniors to review, but that hasn't been measured.

---

## 3. Engineering-discipline skills

| Skill | Source | Installs (2026-10-01) | Designed to | How it works |
|---|---|---|---|---|
| `tdd` | mattpocock/skills | about 1M | Red-green-refactor workflow | Prompt instructions |
| `diagnosing-bugs` / `diagnose` | mattpocock/skills | about 696K / 240K | Structured bug diagnosis | Prompt instructions |
| `systematic-debugging` | obra/superpowers | n/a | Find the root cause before fixing | Prompt instructions |
| `test-driven-development` | obra/superpowers | n/a | Watch the test fail, then make it pass | Prompt instructions |
| `code-review` | mattpocock/skills | about 648K | Review against coding standards and the spec | Prompt instructions |
| `improve-codebase-architecture` | mattpocock/skills | about 1M | Suggest refactoring opportunities (HTML report) | Prompt instructions |
| `/code-review`, `/debug` | Claude Code (bundled) | n/a | Built-in review and debugging workflows | Bundled skills ([docs](https://code.claude.com/docs/en/skills)) |

**Suggested use:** form your own view first, then run the skill and compare. Write down your bug hypothesis, or do your own review of the PR. AI-generated tests check the AI's *interpretation* of the requirement. Make sure the acceptance criteria come from the requirement itself.

---

## 4. Safety and security tooling

**These tools reduce risk; they don't guarantee safety.** Scanners and checklists have blind spots, and security-sensitive changes still need competent human review.

- **`git-guardrails-claude-code`** (mattpocock, about 418K installs) is described as blocking dangerous git operations. **Whether it uses a hook (enforced) or prompt instructions (advisory) was not verified.** Check its source before relying on it, and keep your platform's own permission settings as the real safety boundary.
- **[`trailofbits/skills`](https://github.com/trailofbits/skills)** (about 7.3K★) includes:
  - `insecure-defaults` (fail-open configs)
  - `sharp-edges` (risky APIs and footguns)
  - `static-analysis` (CodeQL, Semgrep, SARIF)
  - `supply-chain-risk-auditor`

  **Installing takes two steps.** `/plugin marketplace add trailofbits/skills` only *registers* the marketplace; you then install a specific plugin from it (e.g. via `/plugin`). Check the repo for exact plugin names.

---

## 5. Onboarding mode: new hires and new codebases

See the [Onboarding mode playbook](ai-for-junior-devs.md#onboarding-mode-playbook).

**Related evidence (small study):** in [LACY](https://arxiv.org/html/2603.25391v1), five learners on a legacy finance system scored 83% vs. 57% on a quiz using expert-prepared, AI-assisted tours with podcasts vs. AI-only tours they generated themselves. Expert ratings of their explanations were closer (79% vs. 77%). The conditions differ in several ways, so this is *consistent with* the value of expert-guided onboarding but doesn't isolate expert review. Having a maintainer review AI-generated onboarding material is a sensible practice where time allows.

| Tool | Type | Use in onboarding | Notes |
|---|---|---|---|
| **Plan mode** | Claude Code built-in | Exploration without code edits | Blocks *edits*. If you also allow commands, scripts and tests can still have side effects, so run them in a disposable environment. |
| **`/init`** | Claude Code built-in | Generates a starter CLAUDE.md | Compare it with your notes and with a teammate's understanding. |
| **Subagents** | Claude Code built-in | Broad investigations | Keeps your main session focused. |
| **`codebase-onboarding`** skills | Community ([affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code/blob/main/skills/codebase-onboarding/SKILL.md), [borghei/Claude-Skills](https://github.com/borghei/Claude-Skills/blob/main/engineering/codebase-onboarding/SKILL.md)) | Architecture overview, file map, setup steps | **AI-only output.** Have a maintainer review it where practical. |
| **`grill-with-docs`** | mattpocock/skills | Builds a glossary and ADRs | Have a maintainer check the domain definitions. |
| **Explanatory output style** | Anthropic official | Explains codebase patterns during first changes | Explanations can be wrong. Verify them in the code. |
| **`quiz-me`** | rodbv/socratic-skills | Checks your understanding | One check among several. Not a gate on its own. |
| **`mentor.agent.md`** | github/awesome-copilot | Challenges your assumptions | Advisory prompt. |

**Team-side idea:** write a project-specific `onboarding` skill in `.claude/skills/` that holds **human-curated** content: example PRs, good first issues, owners and unwritten rules. Keep it under code review like any other team documentation.

---

## 6. Popular skills with possible relevance to juniors

No registry reports installs by experience level, so **this report can't say which skills juniors actually use.** The table lists popular skills (by skills.sh installs on 2026-10-01) with a judgment of how they *might* fit a learner's workflow.

| Skill | skills.sh installs | Intended learning support / plausible use |
|---|---|---|
| `find-skills` (vercel-labs) | about 3.7M | Discovery tool. Neutral. |
| `grill-me` (mattpocock) | about 1.3M | Designed to make the human articulate a design. Plausible support for planning. |
| `tdd` (mattpocock) | about 1M | Workflow structure. Learning depends on how it's used. |
| `frontend-design` (anthropics) | about 943K | Mainly delivery. It can support learning if you critique and change its output yourself. |
| `vercel-react-best-practices` | about 762K | The rule set is readable reference material. |
| `teach` (mattpocock) | about 739K | Designed for teaching. Not evaluated. |
| `caveman` (token reduction) | about 225K | Shorter output. Whether that helps or hurts learning is unknown. Targeted hints can beat long explanations. |

Star counts for small tutoring repos (for example, socratic-skills at about 17★) are a **different unit** and can't be compared with these install counts.

---

## 7. Example starter setups (adapt; start small)

Begin with **one tutoring workflow and one independent check**. Add more only if a gap shows up. A large kit of tools is a learning burden in itself. To judge whether a setup helps, track *aided delivery*, *immediate understanding* and *later unaided performance* separately, on comparable tasks rather than repeats of the same one. See habit 13 in [report 1](ai-for-junior-devs.md#c-habits-worth-trying). A few tasks give you a personal observation, not proof.

**Learning mode** (new library or concept)
- *Minimal:* Learning output style, plus solving a related task without AI a few days later
- *Optional additions:* `mentoring-juniors` or `guide-me` when stuck; `agent-tutor-skill` for concepts that keep slipping

**Onboarding mode** (new job or codebase)
- *Minimal:* plan mode and subagents for exploration, plus a weekly session with a human buddy to review your architecture sketch
- *Optional additions:* `/init` or a codebase-onboarding skill (review its output with a maintainer); `grill-with-docs` for the glossary

**Delivery mode** (familiar work)
- *Minimal:* your normal agent, plus your own review of the diff before any AI review
- *Optional additions:* `tdd` or `diagnosing-bugs`; `code-review` after your own review; Trail of Bits scanners for security-sensitive areas

**For teams:** keep an internal list of approved skills, pinned to reviewed versions. Consider making the Learning output style the *default* (not mandatory) for people new to an area. Treat skill output as material for mentoring conversations, not as a substitute for them.

---

## 8. Installing and running skills safely

**What Snyk ToxicSkills found** ([Snyk](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)):
- **Sample:** Snyk describes the 3,984-skill corpus (scanned 2026-02-05) both as "from ClawHub and skills.sh" and as "3,984 skills from ClawHub". Its per-policy table reports **ClawHub (all)** separately from a curated **top-100 skills.sh** baseline. Read the percentages below as describing that corpus, which is overwhelmingly ClawHub. They don't describe skills.sh in general.
- **Scanner findings (automated):**
  - 36.8% (1,467) had a finding of any severity.
  - 13.4% (534) had a **critical** finding. Snyk's critical categories are **prompt injection, malicious code and suspicious downloads**. Secret detection and credential handling are rated **high**, not critical.
  - In the skills.sh top-100 baseline, the critical detectors flagged 0%.
- **Confirmed malicious (manual review):** **76 payloads**, aimed at credential theft, backdoors and data exfiltration.
- **Scanner findings are not the same as confirmed attacks.** Most were vulnerabilities or risky patterns, not deliberate malware.

**How a skill or plugin can act on your machine:**

| Mechanism | Enforced? | Risk |
|---|---|---|
| SKILL.md instructions | No. Advisory text for the agent. | Prompt injection: instructions can tell the agent to do harmful things with *your* permissions. |
| Bundled scripts | They run when the agent calls them. | Arbitrary code execution. |
| Hooks | **Depends on the event, configuration and response.** Hooks *run* automatically. In Claude Code, `PreToolUse` can deny a tool call (exit code 2 or a `deny` decision). `SessionStart` cannot block anything; its output only adds context, which is advisory. See the [hooks reference](https://code.claude.com/docs/en/hooks). | Arbitrary code runs on every matching event. A hook that injects instructions is no stronger than any other instruction. |
| MCP servers or remote fetches | Depends. | Data exfiltration; behavior can change after you install. |
| Platform permissions | Yes, for the tool calls they govern. | Only as strong as how you configure them. |
| OS-level sandbox | Yes, for the processes it covers. | **Check which execution paths it covers** (agent shell commands, hooks, MCP servers, scripts). Don't assume they all share the same restrictions. |

**Practices:**
- **Read everything a skill contains** before installing: SKILL.md, scripts, hooks and remote URLs. Reading reduces risk but **doesn't guarantee** safety against obfuscated or future changes.
- **Pin to a reviewed revision** and review updates before applying them. Prefer known publishers, but don't treat that as proof of safety.
- **Try new skills in a disposable environment** with minimal credentials and no access to production, `~/.ssh` or real `.env` files.
- **Never treat a skill's name as a guarantee.** A "guardrails" or "learning" label doesn't create a sandbox. Configure platform permissions and sandboxing separately.
- **Scanners such as [Snyk Agent Scan](https://labs.snyk.io/experiments/skill-scan/) help but have limited coverage.**

---

## Remaining verification gaps

- No skill was installed or run. Behavior, enforcement mechanisms (especially `git-guardrails-claude-code`), install commands and compatibility across Claude Code, Copilot and Codex were not tested.
- Repository revisions were not recorded. Descriptions may change.
- The behavior of `scaffold-exercises` and the source of the X post were not verified.
- This review's non-systematic search found no learning-outcome evaluations for the specific skills listed. A systematic search (defined terms, databases and inclusion criteria) could find some.

---

## Sources

- [skills.sh leaderboard](https://www.skills.sh/) (snapshot 2026-10-01)
- [mattpocock/skills](https://github.com/mattpocock/skills/) · [grill-me explainer (AI Hero)](https://www.aihero.dev/skills-grill-me)
- [anthropics/claude-plugins-official: learning-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) · [explanatory-output-style](https://github.com/anthropics/claude-code/tree/main/plugins/explanatory-output-style) · [Claude Code output styles docs](https://docs.anthropic.com/en/docs/claude-code/output-styles)
- [github/awesome-copilot: mentoring-juniors skill](https://github.com/github/awesome-copilot/blob/main/skills/mentoring-juniors/SKILL.md) · [agent](https://github.com/github/awesome-copilot/blob/main/agents/mentoring-juniors.agent.md) · [mentor agent](https://github.com/github/awesome-copilot/blob/main/agents/mentor.agent.md)
- [rodbv/socratic-skills](https://github.com/rodbv/socratic-skills)
- [Bhala-Srinivash/agent-tutor-skill](https://github.com/Bhala-Srinivash/agent-tutor-skill)
- [bevibing/socrates-skill](https://github.com/bevibing/socrates-skill) · [malkreide/socratic-method-skill](https://github.com/malkreide/socratic-method-skill) · [GitExcited/Socratic](https://github.com/GitExcited/Socratic)
- [obra/superpowers](https://github.com/obra/superpowers)
- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
- [trailofbits/skills](https://github.com/trailofbits/skills)
- [codebase-onboarding (everything-claude-code)](https://github.com/affaan-m/everything-claude-code/blob/main/skills/codebase-onboarding/SKILL.md) · [borghei/Claude-Skills](https://github.com/borghei/Claude-Skills/blob/main/engineering/codebase-onboarding/SKILL.md)
- [LACY (FSE 2026)](https://arxiv.org/abs/2603.25391)
- [Snyk ToxicSkills study](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)
- [Claude Code skills docs](https://code.claude.com/docs/en/skills) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Firecrawl: 14 best Claude Code skills](https://www.firecrawl.dev/blog/best-claude-code-skills)
