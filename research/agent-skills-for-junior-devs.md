# Agent Skills With Possible Value for Junior Developers

*Compiled 2026-10-01 and revised the same day after six adversarial reviews ([round 1](adversarial-review.md), [round 2](adversarial-review-round-2.md), [round 3](adversarial-review-round-3.md), [round 4](adversarial-review-round-4.md), [round 5](adversarial-review-round-5.md), [round 6](adversarial-review-round-6.md)). Companion to [ai-for-junior-devs.md](ai-for-junior-devs.md).*

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
These come from wider pedagogy and engineering practice. Structured tutoring has promising evidence in math and physics ([see report 1](ai-for-junior-devs.md#1-what-the-evidence-says)). A **Socratic tutor that questions the learner** is one proposed way to implement it. Those studies evaluated whole tutoring packages: they **don't isolate** what Socratic questioning contributes, and they didn't test the skills listed here. It is also not a direct implementation of "conceptual inquiry". Treat these candidates as hypotheses to pilot.

In programming education specifically, CS50's tutor is instructed not to give solutions, yet about 22% of its responses and 48% of conversations contained code blocks ([Liu et al., SIGCSE 2025](https://cs.harvard.edu/malan/publications/fp0627-liu.pdf)). **Prompts alone don't guarantee tutoring behavior.** Whatever you pilot, check it with conversation review (see [section 7](#7-example-starter-setups-adapt-start-small)).

| Proposed support | What the skill is designed to do | Candidate skills |
|---|---|---|
| Socratic tutoring (agent asks, holds back answers, escalates gradually) | Keep the learner doing the reasoning, with a way out when stuck | mentoring-juniors, socratic-skills `guide-me`, socrates-skill |
| Retrieval practice / spaced repetition | Quiz on concepts over time | agent-tutor-skill (heuristic scheduling; see below), `quiz-me` |
| Design interview before coding | Agent interviews the human about the plan | grill-me, grill-with-docs, superpowers `brainstorming` |
| Structured debugging | Hypothesis-driven debugging loop | diagnosing-bugs, superpowers `systematic-debugging` |
| Test-first workflow | Write a failing test, then make it pass (implementations differ; see section 3) | tdd, superpowers `test-driven-development` |
| Security checks | Flag insecure defaults and footguns | trailofbits `insecure-defaults`, `sharp-edges` |

**Known failure mode:** an AI-written quiz, explanation or test can repeat the same mistake as the AI-written code. Check against something independent: the original requirement, the docs, a human reviewer, or the code's actual behavior.

---

## 1. Learning and tutoring skills (intended learning support)

### Anthropic Learning and Explanatory output styles
- **What it does:**
  - **Learning:** at decision points the agent leaves a `TODO(human)` for you to write 5–10 lines of design or business logic, and adds "Insights" about trade-offs.
  - **Explanatory:** adds 2–3 short insights about implementation choices and codebase patterns.
- **How it works:** these are two separate mechanisms with **similar, not identical** instructions. Both are **advisory**: the agent is asked to behave this way, not forced to.
  - **Built-in output style:** configured in Claude Code itself. Per current docs, the built-in Learning style has its own contribution-and-resume behavior.
  - **Plugin:** a **SessionStart hook** that adds context at session start. The [pinned hook script](https://github.com/anthropics/claude-plugins-official/blob/ab024cdcfa7ca80be204acd4907656ba5a968589/plugins/learning-output-style/hooks-handlers/session-start.sh) says it combines an earlier, unshipped Learning style with Explanatory-style "Insights".
  - **Choose one.** There's no evidence that running both adds learning value.
- **Where to get it:** set an output style in Claude Code with `/output-style learning` or `/config → Output style`. In the desktop app, set `"outputStyle": "Learning"` in a settings file. These are the routes as of 2026-10; check the [output styles docs](https://code.claude.com/docs/en/output-styles).
- **Doesn't reach ordinary subagents.** Per the [docs](https://code.claude.com/docs/en/output-styles#how-output-styles-work), an output style applies to the main conversation and to *forks*, but other subagents run their own system prompt. Work you delegate to a subagent may come back fully done rather than with a `TODO(human)` for you. If a delegated task should keep the tutoring interaction, say so in the subagent's instructions and check what comes back. Or use the plugins: [learning-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) and [explanatory-output-style](https://github.com/anthropics/claude-code/tree/main/plugins/explanatory-output-style).
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
- **Install:** the README (at `dda051c`) shows `npx skills add rodbv/socratic-skills/skills/quiz-me`. That confirms where the command comes from, not that installing works.

### [`Bhala-Srinivash/agent-tutor-skill`](https://github.com/Bhala-Srinivash/agent-tutor-skill) (about 10★, MIT)
- **What it does:** a teaching loop with quizzes, review scheduling and per-concept mastery tracking stored in `~/.learn/`.
- **Scheduling is FSRS-*inspired*, not FSRS.** Its [`references/fsrs.md`](https://github.com/Bhala-Srinivash/agent-tutor-skill/blob/e273585b542d1d77b72c87eaa605d384ea4479f1/references/fsrs.md) tells the agent to use a retrievability formula plus approximate, rating-based "practical growth multipliers" with a difficulty adjustment. The [published FSRS algorithm](https://github.com/open-spaced-repetition/awesome-fsrs/wiki/The-Algorithm) uses parameterized state-update formulas; sharing the terms *difficulty* and *stability* doesn't reproduce it. The calculation is carried out by the agent following prompt instructions, not by verified code. Its "mastered" label is triggered when estimated stability reaches 90 days. Treat it as scheduling metadata, not proof of durable understanding.
- **How it works:** prompt instructions plus files written to your home directory. Review what it writes before installing.
- **Note:** spaced repetition and retrieval practice are well supported in learning research in general. This review found no evaluation of this particular implementation. Keep independent practice checks.

### `teach` from [`mattpocock/skills`](https://github.com/mattpocock/skills) (about 739K installs)
- **What it does (static read of the [pinned SKILL.md](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/productivity/teach/SKILL.md)):** a **multi-session, stateful** teaching workflow that treats the *current directory* as a learning workspace. It writes:
  - `MISSION.md` (why you want to learn this)
  - `RESOURCES.md`
  - numbered `learning-records/*.md`
  - short self-contained HTML `lessons/` and `reference/` sheets
  - shared `assets/` such as stylesheets and quiz widgets
  - `NOTES.md`

  It asks the agent to ground teaching in high-quality sources, distinguish fluency from long-term retention, and use retrieval practice, spacing and interleaving.
- **Invocation:** user-invoked only (`disable-model-invocation: true`). Run it with `/teach`.
- **Practical note:** run it in a **dedicated learning directory**, not in your application repo, unless you want those files committed alongside your code.
- **Evidence:** this review found no learning-outcome evaluation. Producing lessons and records doesn't show that a skill was retained.
- The same repo lists **`scaffold-exercises`** (about 402K installs) under Learning. Its [SKILL.md](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/scaffold-exercises/SKILL.md) (static read) is an **exercise-authoring workflow** for a particular course environment. It creates section and exercise folders with `problem/`, `solution/` and `explainer/` readmes, requires `pnpm ai-hero-cli internal lint` to pass, and then makes a git commit. It's a tool for *course authors*, not an adaptive tutor for learners.

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
| `tdd` | mattpocock/skills | about 1M | **Red → green only** ([pinned](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/engineering/tdd/SKILL.md)). It agrees the public test boundaries ("seams") with the user before writing any test, then works one failing behavior test and minimal code at a time. It explicitly says **refactoring is not part of the loop**; that belongs to the review stage (its `code-review` skill). | Prompt instructions |
| `diagnosing-bugs` / `diagnose` | mattpocock/skills | about 696K / 240K | Structured bug diagnosis | Prompt instructions |
| `systematic-debugging` | obra/superpowers | n/a | Find the root cause before fixing | Prompt instructions |
| `test-driven-development` | obra/superpowers | n/a | Watch the test fail, then make it pass | Prompt instructions |
| `code-review` | mattpocock/skills | about 648K | Parallel reviews against documented standards (plus a code-smell baseline) and against the originating spec ([pinned](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/engineering/code-review/SKILL.md)). **Reviews committed branch changes only** (see note below). Needs a base reference, and its issue-tracker setup (`/setup-matt-pocock-skills`). | Prompt instructions |
| `improve-codebase-architecture` | mattpocock/skills | about 1M | Suggest refactoring opportunities (HTML report) | Prompt instructions |
| `/code-review`, `/debug` | Claude Code (bundled) | n/a | Built-in review and debugging workflows | Bundled skills ([docs](https://code.claude.com/docs/en/skills)) |

**Scope of `code-review` (mattpocock).** The skill runs `git diff <fixed-point>...HEAD`. A three-dot diff compares the merge base with the **committed** `HEAD` ([git docs](https://git-scm.com/docs/git-diff)). It **does not include staged, unstaged or untracked changes**, and it stops if the committed diff is empty. So if you run it before committing, it reviews your earlier commits and **misses your current edits**, or stops entirely. Its description mentions "work-in-progress changes", but the prescribed command doesn't cover them. An agent *might* deviate from it; don't count on that. Use it as a **committed-branch / PR review**. For pre-commit review, use a workflow that explicitly includes the working tree, such as `git diff HEAD` plus any untracked files you mean to add, and **check the list of files it actually reviewed**. Don't commit just to make this skill work. Claude Code's bundled `/code-review` is a separate tool, and this note doesn't apply to it.

**Suggested use:** form your own view first, then run the skill and compare. Write down your bug hypothesis, or do your own review of the PR. AI-generated tests check the AI's *interpretation* of the requirement. Make sure the acceptance criteria come from the requirement itself.

---

## 4. Safety and security tooling

**These tools reduce risk; they don't guarantee safety.** Scanners and checklists have blind spots, and security-sensitive changes still need competent human review.

- **`git-guardrails-claude-code`** (mattpocock, about 418K installs). **Mechanism (static read):** the [SKILL.md](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/git-guardrails-claude-code/SKILL.md) instructs the agent to install a **`PreToolUse` hook matching the `Bash` tool**. The hook runs a [bundled script](https://github.com/mattpocock/skills/blob/d81f3a183412e71a5b1e84ca21bc1a35eea03a60/skills/misc/git-guardrails-claude-code/scripts/block-dangerous-git.sh) that exits with code 2 to block matching commands. Once set up, it is **enforced for that path**, not just advisory.
  - **Coverage is limited.** The script checks the command text against a fixed list of regular expressions: `git push`, `git reset --hard`, `git clean -f(d)`, `git branch -D`, `git checkout .`, `git restore .`, `push --force` and `reset --hard`. That is not a parser or full git access control.
  - **Forms the list doesn't match:** `git -C repo push` doesn't contain the string `git push`. Git aliases, scripts that call git indirectly, and tools other than `Bash` aren't covered either.
  - **Depends on `jq`.** If `jq` is missing or extraction fails, the script doesn't explicitly fail closed. It reaches `exit 0`, which allows the command.
  - These limits come from reading the code, not from live bypass tests.
  - **Treat it as a seatbelt against common mistakes, not a security boundary.** Keep platform permissions and branch protection on the remote as the real controls.
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
| **`codebase-onboarding`** ([everything-claude-code @ `c70874f`](https://github.com/affaan-m/everything-claude-code/blob/c70874fae9eb0e5ad0365beb7e2955899fd1d30f/skills/codebase-onboarding/SKILL.md)) | Community; prompt instructions | Onboarding guide printed in the conversation; **creates or updates a `CLAUDE.md` in the project root** (instructions say to preserve existing content) | **Writes a file**, so it isn't pure exploration. Ask for findings in the conversation first, and review the proposed `CLAUDE.md` diff before keeping it. AI-only output: have a maintainer review it where practical. |
| **`codebase-onboarding`** ([borghei/Claude-Skills @ `5318eda`](https://github.com/borghei/Claude-Skills/blob/5318eda16c134c500c237425404975545c0eedd2/engineering/codebase-onboarding/SKILL.md)) | Community; prompt **plus three bundled Python scripts** (`architecture_mapper.py`, `onboarding_generator.py`, `setup_validator.py`) | Architecture map, onboarding guide, setup-completeness score | **Runs code.** Inspect the scripts, not just the SKILL.md. **Known issue (static read):** `setup_validator.py` reports "`.env` file is committed!" whenever a local `.env` exists and `.gitignore` lacks one of a few literal lines. It never asks git. So it can flag an untracked, globally ignored `.env`, and it misses a tracked `.env` that's absent from disk. Check tracking yourself with `git ls-files --error-unmatch -- .env` and ignore status with `git check-ignore -v .env`. Don't use its score as a gate. |
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
| `teach` (mattpocock) | about 739K | Designed as a multi-session teaching workspace. This review found no learning-outcome evaluation. |
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
- *Optional additions:* `/init` or a codebase-onboarding skill. Review its output with a maintainer, and check which files it writes or scripts it runs; neither community workflow is guaranteed to stay read-only or work in plan mode. Plus `grill-with-docs` for the glossary.

**Delivery mode** (familiar work)
- *Minimal:* your normal agent, plus your own review of the diff before any AI review
- *Optional additions:* `tdd` or `diagnosing-bugs`; mattpocock `code-review` for **committed** branch or PR review, after your own review; Trail of Bits scanners for security-sensitive areas

**For teams:** keep an internal list of approved skills, pinned to reviewed versions. Consider making the Learning output style the *default* (not mandatory) for people new to an area. Treat skill output as material for mentoring conversations, not as a substitute for them.

**Check tutoring behavior, not just intent.** For any tutoring skill or Learning-style setup you roll out, periodically review a small sample of real conversations (with consent and in line with privacy policy). Look for:
- **Answer leakage:** full solutions where hints were intended. Define in advance which code examples are acceptable.
- **Useful hints:** do they move the learner forward?
- **Escalation:** does the learner get enough help, or a human, when truly stuck?

Repeat after any model, prompt or skill-version change. CS50 found code-block rates *rose* after a model switch. This checks behavior. It does not measure learning outcomes.

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

- No skill was installed or run. Runtime behavior, install commands and compatibility across Claude Code, Copilot and Codex were not tested. The `git-guardrails-claude-code`, `scaffold-exercises`, `agent-tutor-skill` and Learning plugin descriptions are based on **static reads** of the pinned revisions below.
- Revisions statically read (in review rounds 3, 5 and 6; not installed): `mattpocock/skills@d81f3a1` (`git-guardrails-claude-code`, `scaffold-exercises`, `tdd`, `code-review`, `teach`, `diagnosing-bugs`, `improve-codebase-architecture`), `Bhala-Srinivash/agent-tutor-skill@e273585`, `anthropics/claude-plugins-official@ab024cd`, `rodbv/socratic-skills@dda051c`, `affaan-m/everything-claude-code@c70874f` (codebase-onboarding), `borghei/Claude-Skills@5318eda` (codebase-onboarding and `setup_validator.py`). Other skills' descriptions come from their current READMEs and may change.
- The X post was not authenticated.
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
- [Liu et al.: Improving AI in CS50 (SIGCSE 2025)](https://cs.harvard.edu/malan/publications/fp0627-liu.pdf)
- [Snyk ToxicSkills study](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)
- [Claude Code skills docs](https://code.claude.com/docs/en/skills) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Firecrawl: 14 best Claude Code skills](https://www.firecrawl.dev/blog/best-claude-code-skills)
