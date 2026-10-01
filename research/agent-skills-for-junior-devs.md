# Agent Skills That Help Junior Developers

*Research compiled 2026-10-01. Companion to [ai-for-junior-devs.md](ai-for-junior-devs.md).*

"Skills" here means **agent skills**: `SKILL.md` folders and plugins for Claude Code, plus the equivalents for Copilot, Codex and similar tools. Install counts come from [skills.sh](https://www.skills.sh/) and stars from GitHub, both read on the date above. Both change quickly.

---

## TL;DR

- **Skills that help juniors learn are rare and niche.** Skills that make an agent *write more code faster* dominate the most-installed lists. The useful ones for juniors make the human **think, explain or write**: Socratic tutors, "grill me" interviews, quizzes on your own diffs, and Anthropic's Learning output style.
- **Matt Pocock's [`mattpocock/skills`](https://github.com/mattpocock/skills) is the best-known source** of process skills that suit juniors well. `grill-me` (about 1.3M installs), `tdd` (about 1M), `teach`, `diagnosing-bugs` and `code-review` all rank highly on skills.sh.
- **The one skill to start with:** Anthropic's official **`learning-output-style`** plugin, or the Learning output style itself. It has the agent leave 5–10 meaningful lines for *you* to write. That puts the high-scoring "conceptual inquiry" and "code + explanation" patterns from the [Anthropic RCT](https://www.anthropic.com/research/AI-assistance-coding-skills) into practice.
- **No public data shows which skills juniors in particular install.** skills.sh reports totals, not who installs them. The section on "skills heavily used by juniors" below is inferred from category and audience, not measured.
- **Security warning:** Snyk scanned about 4K marketplace skills and found **13.4% with critical issues** and **76 confirmed malicious payloads**. Juniors are the group most likely to install skills without reading them. ([Snyk ToxicSkills](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/))

---

## How skills map to the research

The first report found that **how** a junior uses AI decides whether they learn. The skills below are grouped by which healthy pattern they enforce:

| Pattern from research | What a skill should do | Best skills |
|---|---|---|
| **Conceptual inquiry / think first** | Ask the human questions and withhold answers | mentoring-juniors, socratic-skills `guide-me`, socrates-skill |
| **Code + explanation** | Explain the *why* next to every change | explanatory-output-style, learning-output-style |
| **Generate, then comprehend** | Test the human's understanding of code they just shipped | socratic-skills `quiz-me`, agent-tutor-skill |
| **Plan before you prompt** | Interview the human about the design first | grill-me, grill-with-docs, superpowers `brainstorming` |
| **Debug with a hypothesis** | Force a disciplined debugging loop | diagnosing-bugs, superpowers `systematic-debugging` |
| **Tests and verification** | Red-green-refactor workflow | tdd, superpowers `test-driven-development` |
| **Security care** | Flag insecure defaults and footguns | trailofbits `insecure-defaults`, `sharp-edges` |

---

## 1. Learning and tutoring skills: the human writes the code

### ⭐ Anthropic `learning-output-style` and `explanatory-output-style` (official plugins)
- **Learning:** at decision points, the agent prepares the context and leaves a `TODO(human)` for you to write **5–10 lines of business logic or design code**. It then explains trade-offs and gives "Insights" before and after.
- **Explanatory:** adds 2–3 educational insights about implementation choices and codebase patterns each time it writes code.
- Built in as an output style: `/config → Output style → Learning`. Also available as plugins in [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) and [anthropics/claude-code/plugins](https://github.com/anthropics/claude-code/tree/main/plugins/explanatory-output-style).
- **Why it matters:** this is the closest tool match to the RCT's high-scoring patterns. Lydia Hallie (Anthropic) [uses it for side projects](https://x.com/lydiahallie/status/2056420694087594283) "to stay hands-on".

### ⭐ `mentoring-juniors` ("Sensei") from [github/awesome-copilot](https://github.com/github/awesome-copilot/blob/main/skills/mentoring-juniors/SKILL.md)
- Socratic mentor written *specifically for junior devs*. Available as both a SKILL.md and a [Copilot agent](https://github.com/github/awesome-copilot/blob/main/agents/mentoring-juniors.agent.md).
- **PEAR loop:** **P**lan (write pseudocode before asking) → **E**xplore (use AI suggestions as a starting point) → **A**nalyze (read every line) → **R**ewrite (rebuild it yourself).
- **Progressive clues:** guiding questions and docs → pseudocode and partial snippets → step-by-step help → escalate to a human.
- Golden rules: no unexplained solutions, no blind copy-paste, never condescending, never rushed.
- Triggers on phrases like "I'm stuck", "explain this", "teach me", "why doesn't this work", "ELI5".
- Related: [`mentor.agent.md`](https://github.com/github/awesome-copilot/blob/main/agents/mentor.agent.md) is a mentor mode with no edits that challenges assumptions.

### [`rodbv/socratic-skills`](https://github.com/rodbv/socratic-skills) (about 17★, MIT)
- **`quiz-me`** quizzes you on *your own* diff, spec or plan one question at a time, probing before it corrects you. It works well as a gate before committing: "if you can't explain it, don't merge."
- **`guide-me`** walks you through implementing a spec step by step. It reviews your diffs and gives hints but **never gives copy-paste code**. Test modes are TDD-first, test-after, mixed or none.
- `npx skills add rodbv/socratic-skills/skills/quiz-me`

### [`Bhala-Srinivash/agent-tutor-skill`](https://github.com/Bhala-Srinivash/agent-tutor-skill) (about 10★, MIT)
- Tutor built on cognitive science: explain → example → check → evaluate → practice.
- **FSRS spaced repetition**, zero-hint quizzes with plausible wrong answers, and per-concept mastery tracking stored in `~/.learn/`.
- Teaches from your own PDFs, docs, URLs and **code**. Good for learning a new library the way the RCT participants learned Trio.

### `teach` from [`mattpocock/skills`](https://github.com/mattpocock/skills) (about 739K installs)
- "Teach new skills/concepts using the directory as a workspace." It is the most-installed general teaching skill on skills.sh.
- The same repo has **`scaffold-exercises`** (about 402K installs, listed under Learning). Judging by its name and category it generates practice exercises; I didn't verify that.

### Other Socratic skills (small and lightly used)
- [`bevibing/socrates-skill`](https://github.com/bevibing/socrates-skill): Socratic questioning over any asset (code, docs, configs).
- [`malkreide/socratic-method-skill`](https://github.com/malkreide/socratic-method-skill): written in German.
- [`GitExcited/Socratic`](https://github.com/GitExcited/Socratic): a VS Code extension rather than a skill. It gives hints instead of solutions.

---

## 2. "Think before coding" skills

### ⭐ `grill-me` and `grill-with-docs` ([mattpocock/skills](https://github.com/mattpocock/skills), about 1.3M and 1.1M installs)
- Before any code is written, the agent **interviews you relentlessly** about your plan until every branch of the decision tree is resolved. `grill-with-docs` also updates a GLOSSARY.md and ADRs as it goes.
- Juniors often skip design. This reverses the usual flow: the AI asks and the human answers. It is the #2 most-installed skill on skills.sh.

### `brainstorming` and `writing-plans` from [`obra/superpowers`](https://github.com/obra/superpowers)
- Brainstorming refines a rough idea through questions, explores alternatives and saves a design doc *before* coding. Superpowers then produces a plan built around TDD, YAGNI and DRY.

### [`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) (about 216K★)
- Four guardrails for the agent: **think before coding** (state assumptions and push back), **simplicity first**, **surgical changes** and **goal-driven execution** with verifiable success criteria.
- It targets the agent, not the human, but it reduces the over-engineered "house of cards" code that juniors struggle to review.

---

## 3. Engineering-discipline skills (debugging, testing, review)

| Skill | Source | Installs | What it does for a junior |
|---|---|---|---|
| `tdd` | mattpocock/skills | about 1M | Red-green-refactor. Tests are written first, so you define "done" before the code exists. |
| `diagnosing-bugs` / `diagnose` | mattpocock/skills | about 696K / 240K | Disciplined bug diagnosis with a red-green loop instead of pasting errors into the AI and hoping. |
| `systematic-debugging` | obra/superpowers | n/a | Root-cause debugging process, used before any fix. |
| `test-driven-development` | obra/superpowers | n/a | Enforces "watch it fail, then make it pass". |
| `code-review` | mattpocock/skills | about 648K | Reviews against both coding standards and the spec. Useful to *learn* review by comparing its findings with yours. |
| `improve-codebase-architecture` | mattpocock/skills | about 1M | Finds deepening and refactoring opportunities and produces an HTML report. Works against the duplication GitClear found. |
| `/code-review`, `/debug` | Claude Code (bundled) | — | Built into Claude Code, nothing to install. |

**How juniors should use these:** run the skill *after* you form your own view. For example, write down your bug hypothesis and then run `diagnosing-bugs`, or review the PR yourself and then compare with `code-review`. The gap between your view and the skill's is where you learn.

---

## 4. Safety and security guardrails

- **`git-guardrails-claude-code`** (mattpocock, about 418K installs) blocks dangerous git operations by the agent. It is a safety net for people still learning git.
- **[`trailofbits/skills`](https://github.com/trailofbits/skills)** (about 7.3K★): `insecure-defaults` finds fail-open configs, `sharp-edges` flags risky APIs and footguns, `static-analysis` runs CodeQL and Semgrep, and `supply-chain-risk-auditor` checks dependencies. This addresses Veracode's finding that **45%** of AI-generated code samples had security flaws. Install with `/plugin marketplace add trailofbits/skills`.

---

## 5. Onboarding mode: new hires and new codebases

See the [Onboarding mode playbook](ai-for-junior-devs.md#onboarding-mode-playbook) for the full 4-week structure. These are the tools for it:

| Tool | Type | Use in onboarding |
|---|---|---|
| **Plan mode** (`Shift+Tab`) | Claude Code built-in | Read-only exploration. Ask questions without the agent changing anything. |
| **`/init`** | Claude Code built-in | Generates a starter CLAUDE.md from the repo. As a new hire, compare it with your own notes and fix what's wrong. |
| **Subagents** ("use subagents to investigate…") | Claude Code built-in | Broad "how does X work" investigations without filling your main context. |
| **`codebase-onboarding`** skills | Community ([affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code/blob/main/skills/codebase-onboarding/SKILL.md), [borghei/Claude-Skills](https://github.com/borghei/Claude-Skills/blob/main/engineering/codebase-onboarding/SKILL.md)) | Scan the repo and produce an architecture overview, annotated file map, setup steps, task runbooks and a CLAUDE.md. |
| **`grill-with-docs`** | mattpocock/skills | Turns Q&A into a GLOSSARY.md and ADRs, so the domain language you learn gets written down. |
| **`explanatory-output-style`** | Anthropic official | Adds codebase-pattern "Insights" while you make your first changes. |
| **`quiz-me`** | rodbv/socratic-skills | Checks your understanding of the system or of your first PRs. |
| **`agent-tutor-skill`** | Bhala-Srinivash | Spaced repetition for domain concepts and internal terminology. |
| **`mentor.agent.md`** | github/awesome-copilot | Mentor mode with no edits that challenges your assumptions about the codebase. |

**Caution:** a generated onboarding guide is passive reading, and AI explanations of *your* codebase can be wrong about conventions and history. Draw the architecture yourself first, trace one request by hand, check claims against the code, and ask humans the *why* questions.

**Team-side skill idea:** write a project-specific `onboarding` skill in `.claude/skills/` that holds the curated example PRs, good first issues, who owns what, and the "unwritten rules". It is reusable for every new hire, and new hires keep it up to date.

---

## 6. "Skills heavily used by juniors" (inferred)

No registry publishes installs by experience level. Based on audience, marketing and the most-installed lists, the skills juniors and newcomers most likely install are:

| Skill | Installs | Helps learning? |
|---|---|---|
| `find-skills` (vercel-labs) | about 3.7M | Neutral. It's how most people discover skills. |
| `frontend-design` (anthropics) | about 943K | ⚠️ Delivery. It produces UI but doesn't teach design. |
| `vercel-react-best-practices` | about 762K | ➕ Moderate. The 57 rules are readable, so learn them rather than just applying them. |
| `caveman` (token reduction) | about 225K | ➖ Shorter output means *less* explanation, which is the opposite of what learners need. |
| Superpowers (whole framework) | popular | ➕ Good process, but autonomous multi-agent runs can turn into the "delegation" pattern. |
| `grill-me` | about 1.3M | ✅ Yes. |
| `learning-output-style` | official | ✅ Yes. |

**Pattern:** the skills juniors find first are mostly about **producing output**. Learning-focused skills have far fewer installs. Socratic tutors have stars in the double digits, while "do it for me" skills reach six or seven figures. This matches the "Silent Silo" risk from the first report.

---

## 7. A recommended starter kit for juniors

**Learning mode** (a new library, concept or codebase):
1. Learning output style (`/config → Output style → Learning`), or the `learning-output-style` plugin
2. `mentoring-juniors` or `guide-me` when you're stuck
3. `quiz-me` before every commit
4. `agent-tutor-skill` for spaced repetition on concepts you keep forgetting

**Onboarding mode** (new job or new codebase):
1. Plan mode and subagents for read-only exploration. `/init` for a starter CLAUDE.md.
2. A `codebase-onboarding` skill for a first map, then check it yourself
3. `grill-with-docs` to build the glossary and domain model
4. `quiz-me` on the architecture and your first PRs
5. Your team's own `onboarding` skill, if there is one

**Delivery mode** (work you already understand):
1. `grill-me` before starting a feature
2. `tdd` and `diagnosing-bugs` for the implementation loop
3. `code-review`, run *after* your own review
4. `git-guardrails-claude-code` and `trailofbits/insecure-defaults` as safety nets

**For teams:** fork these into a shared team marketplace and add a team `CLAUDE.md` rule such as "for junior engineers, default to the Learning output style". Use `quiz-me` output as material for the "walk me through it" conversation in code review.

---

## 8. Installing skills safely (especially for juniors)

From [Snyk ToxicSkills](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) (3,984 skills scanned, Feb 2026): **36.8%** had at least one security flaw, **13.4%** had critical issues, and **76** malicious payloads were found for credential theft, backdoors and data exfiltration. In one documented attack, three lines of markdown were enough to exfiltrate SSH keys.

- **Read every SKILL.md and script before installing.** A skill is code plus a prompt with the agent's permissions.
- Prefer official or well-known sources: anthropics, github/awesome-copilot, vercel-labs, trailofbits, mattpocock, obra.
- Be careful with skills that run shell scripts, read `~/.ssh` or `.env`, or fetch remote URLs.
- Consider [Snyk Agent Scan](https://labs.snyk.io/experiments/skill-scan/) for third-party skills.

---

## Sources

- [skills.sh leaderboard](https://www.skills.sh/)
- [mattpocock/skills](https://github.com/mattpocock/skills/) · [grill-me explainer (AI Hero)](https://www.aihero.dev/skills-grill-me)
- [anthropics/claude-plugins-official: learning-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) · [explanatory-output-style](https://github.com/anthropics/claude-code/tree/main/plugins/explanatory-output-style) · [Claude Code output styles docs](https://docs.anthropic.com/en/docs/claude-code/output-styles)
- [github/awesome-copilot: mentoring-juniors skill](https://github.com/github/awesome-copilot/blob/main/skills/mentoring-juniors/SKILL.md) · [agent](https://github.com/github/awesome-copilot/blob/main/agents/mentoring-juniors.agent.md) · [mentor agent](https://github.com/github/awesome-copilot/blob/main/agents/mentor.agent.md)
- [rodbv/socratic-skills](https://github.com/rodbv/socratic-skills)
- [Bhala-Srinivash/agent-tutor-skill](https://github.com/Bhala-Srinivash/agent-tutor-skill)
- [bevibing/socrates-skill](https://github.com/bevibing/socrates-skill) · [malkreide/socratic-method-skill](https://github.com/malkreide/socratic-method-skill) · [GitExcited/Socratic](https://github.com/GitExcited/Socratic)
- [obra/superpowers](https://github.com/obra/superpowers)
- [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)
- [trailofbits/skills](https://github.com/trailofbits/skills)
- [Firecrawl: 14 best Claude Code skills](https://www.firecrawl.dev/blog/best-claude-code-skills)
- [codebase-onboarding (everything-claude-code)](https://github.com/affaan-m/everything-claude-code/blob/main/skills/codebase-onboarding/SKILL.md) · [borghei/Claude-Skills](https://github.com/borghei/Claude-Skills/blob/main/engineering/codebase-onboarding/SKILL.md)
- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Snyk ToxicSkills study](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)
- [Claude Code skills docs](https://code.claude.com/docs/en/skills)
