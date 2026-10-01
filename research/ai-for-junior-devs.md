# Good Practices for Using AI as a Junior Developer

*Compiled 2026-10-01 and revised the same day after an [adversarial review](adversarial-review.md). It draws on the 5 links you supplied plus about 15 more sources: trials, surveys, observational and labor data, vendor reports and practitioner writing.*

**How to read this report.** Each claim is tagged with the kind of evidence behind it:

| Tag | Meaning |
|---|---|
| **[RCT]** | Randomized experiment. Can support causal claims, but only for the population, task and time horizon studied. |
| **[Obs]** | Observational or correlational data. Shows associations, not causes. |
| **[Survey]** | Self-reported data. |
| **[Vendor]** | Analysis by a company with a commercial interest in the topic. |
| **[Opinion]** | Practitioner advice. Plausible, but not tested. |
| **[Extrapolation]** | Evidence from a different domain applied to coding by this report. |

The practices in sections 3 to 6 are **proposed habits** built on that evidence. They are not validated interventions.

---

## TL;DR

1. **AI access lowered immediate comprehension in one learning experiment.** In Anthropic's randomized study (52 mostly junior Python devs learning an unfamiliar library), the AI group averaged **50% vs. 67%** on a quiz taken right after the task, with no significant time saving. [RCT] Long-term effects were not measured.
2. **How people used AI was *associated* with different scores.** Participants who asked conceptual questions or asked for explanations scored higher on average than those who delegated. Those groups were small and self-selected, so this suggests habits to try. It does not prove a recipe. [RCT, exploratory subgroup]
3. **Debugging showed the largest gap** in that study. Practising independent debugging is a reasonable priority. [RCT]
4. **Use more than one check of your own understanding.** Explaining the code is one. Predicting behavior, finding a bug that was planted on purpose, and later solving a related task without help are better. [Opinion]
5. **The entry-level market has tightened.** Employment of US software developers aged 22–25 fell nearly 20% from late 2022 to mid-2025 (payroll data). [Obs] That doesn't show which individual habits protect a career.
6. **Daily AI use was *associated* with faster onboarding.** In six enterprises, daily users reached their 10th PR in 49 days vs. 91 for non-users. [Obs, Vendor] Expert-curated AI code tours beat AI-only tours on comprehension (83% vs. 57%) in one industry study. [Controlled study] See the [Onboarding mode playbook](#onboarding-mode-playbook).

---

## 1. What the evidence says

| Finding | Type | Source | Scope and limits | Reasonable takeaway |
|---|---|---|---|---|
| AI group scored **50% vs. 67%** on an immediate quiz. The time difference (about 2 min) was **not significant**. Largest gap on **debugging** questions. | RCT | [Anthropic](https://www.anthropic.com/research/AI-assistance-coding-skills) | n=52, mostly junior, Python users new to the Trio library; short task; quiz right afterward. Lasting skill effects were **not** measured. | In this kind of learning task, AI access can reduce immediate understanding without saving much time. |
| Delegation-style use averaged **<40%**. Conceptual-inquiry and code-plus-explanation use averaged **≥65%**. | RCT (exploratory subgroups) | same | Usage patterns were **not randomized**. Groups had n=2–7. Prior ability or motivation could explain both behavior and score. Anthropic says this analysis does not establish causality. | Habits worth trying, not a proven recipe. |
| Experienced devs had **19% longer completion times** with AI, while *believing* they were about 20% faster. | RCT | [METR paper (2025)](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study-paper.pdf) | 16 experienced devs on their own large OSS repos, with early-2025 tools. METR's [Feb 2026 update](https://metr.org/blog/2026-02-24-uplift-update/) says later tools likely speed devs up more, but its follow-up estimates are unreliable because of selection effects. | Perceived speedups can differ greatly from measured ones. Measure, don't assume. |
| Copilot access was associated with **+26% completed tasks**, with larger gains for less experienced devs. | Field RCTs | [Cui et al. (SSRN)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) | Measures output, not learning. Exact figure taken from secondary summaries; SSRN blocked direct access. | AI can raise juniors' output. Whether skill grows with it is a separate question. |
| Higher confidence in AI was **associated** with less *self-reported* critical thinking. Higher self-confidence was associated with more. | Survey | [Microsoft Research + CMU, CHI 2025](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/) | n=319 knowledge workers (not only developers). Self-report. Does not show AI *causes* reasoning to decline. | Watch for over-trust. |
| Participants who wrote **unaided first, then with an LLM** showed better recall and broader brain activity than those who switched the other way. | Experiment (EEG) | [Kosmyna et al., "Your Brain on ChatGPT" (arXiv)](https://arxiv.org/abs/2506.08872) | **Essay writing, not coding.** 54 participants in sessions 1–3, only **18** in the switched session 4. Brain connectivity is not a measure of engineering skill. | Applying this to coding is **[Extrapolation]**. "Attempt first" is still a reasonable habit to try. |
| **45%** of AI-generated samples failed security tests. XSS defenses failed in **86%** of relevant samples. | Vendor | [Veracode 2025](https://www.veracode.com/blog/genai-code-security-report/) | Vendor's own test set of about 80 tasks across 100+ models. These are not population-wide failure rates. | Security-sensitive AI code needs competent review. |
| **66%** report AI solutions that are "almost right, but not quite". **45%** report that debugging AI code takes more time. About **33%** trust AI accuracy (3.1% highly + 29.6% somewhat), about **46%** distrust it. | Survey | [Stack Overflow 2025](https://survey.stackoverflow.co/2025/ai) | Reported frustration and trust, not measured time lost. | Expect near-misses and verify against primary sources. |
| Copy-pasted and duplicated lines **overtook "moved" (refactored) lines** in 2024. Code revised within 2 weeks rose from 3.1% to 5.7%. | Obs, Vendor | [GitClear 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research) | Repo-wide trends over the period when AI was adopted. Does not show AI caused them for comparable work. "Moved lines" is only a proxy for refactoring. | Refactoring on purpose is sound advice regardless of the cause. |
| AI adoption is **associated** with higher throughput and also with more delivery instability. | Survey/Obs | [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report) | Relationships across organizations, not controlled effects. | Team practices matter as much as the tool. |
| Employment of devs aged 22–25 is down **nearly 20%** from its late-2022 peak to July 2025. Older age groups are flat. | Obs | [Stanford "Canaries in the Coal Mine" (Nov 2025)](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf) | US payroll data. Age is a rough proxy for "junior". This is separate from the paper's broader adjusted figure for AI-exposed jobs. | The entry-level market is tighter. This says nothing about which individual habits help. |
| Entry-level SWE postings reportedly down **28%** since 2022. Describes a "vibe coding" paradox. | Paper | [Adam, SSRN 6409098](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098) | **Provisional.** Seen only through a search-engine abstract snippet; SSRN blocked direct access. Methods not reviewed. | Treat as unverified. |

**Studies of positive learning outcomes are under-represented here.** This report found more evidence about dependence than about well-designed AI tutoring. That gap reflects where the search went, not proof that AI can't support learning.

---

## 2. The core model: "Supervise, don't delegate"

Several practitioner sources independently frame the junior's job as supervising AI rather than handing work to it. **[Opinion]**

- thoughtgears describes AI as a fast colleague who is sometimes careless, not an oracle. ([thoughtgears](https://thoughtgears.co.uk/blog/how-to-use-ai-as-a-junior-developer/))
- Addy Osmani compares AI to an eager junior on your team who needs supervision. ([Osmani](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your))
- daily.dev argues that juniors who guide and check AI will outpace juniors who just lean on it. ([daily.dev](https://daily.dev/blog/how-junior-developers-adapt-to-ai/))
- Osmani's **"knowledge paradox"**: seniors use AI to speed up what they already know, while juniors use it to learn what to do, which is the riskier use. ([The 70% problem](https://addyo.substack.com/p/the-70-problem-hard-truths-about))
- A Reddit commenter argues an engineer's value lies in being responsible for the code, not just delivering it. (r/ExperiencedDevs, paraphrased)

The sources differ on details. One example: whether juniors should type code by hand until fluent.

---

## 3. Practices for junior developers

*Everything below is a **proposed** practice. Adapt the timings and frequencies to your situation. None of the schedules have been compared in studies.*

### A. Three modes: learning, onboarding, delivery
It helps to know which mode you're in before you open the AI.

| | **Learning mode** (new concept, language or library) | **Onboarding mode** (new job, team or codebase) | **Delivery mode** (things you already understand) |
|---|---|---|---|
| Goal | Understanding a *concept* | Building a *mental model of a system* and its people | Throughput with quality |
| AI role | Tutor: conceptual questions, explanations, hints | Guide: answers "where/how" questions; exploration that doesn't change code | Pair programmer: boilerplate, scaffolding, refactors |
| Default habit | Try first or form a hypothesis. Ask for explanations alongside code. | Explore without making changes. Check AI claims against the code and against the people who own it. | Read the diff, check behavior, and make sure you could debug it. |
| Tools (as of 2026-10; check current docs) | Claude **Learning** output style, which leaves `TODO(human)` sections for you; ChatGPT **Study Mode** | Plan mode, subagents, `/init` → CLAUDE.md, codebase-onboarding skills | Normal agent/editor modes |
| Ready to move on when | You can use the concept on a related problem without help | You can sketch the architecture, trace a request and ship small PRs with light review | n/a |

### B. Interaction patterns *associated* with better immediate understanding
These come from exploratory subgroups of the Anthropic study (n=2–7 per group; not randomized). Treat them as **habits to try**, not proven techniques.

| Pattern | Group size | Average quiz score |
|---|---|---|
| ➕ **Conceptual inquiry:** ask only conceptual questions, then write the code yourself | n=7 | ≥65% average; also the fastest of the high-scoring groups |
| ➕ **Code + explanation:** ask for code *and* the reasoning behind each decision | n=3 | ≥65% average |
| ➕ **Generate then comprehend:** if AI writes it, ask follow-up questions until you understand it | n=2 | ≥65% average |
| ➖ **Delegation:** "just write it" | n=4 | <40% average |
| ➖ **Progressive reliance:** starts with questions, then hands everything over | n=4 | <40% average |
| ➖ **Iterative AI debugging:** pastes errors back in a loop without forming a hypothesis | n=4 | <40% average |

### C. Habits worth trying
1. **Attempt first, when it's practical.** A short sketch or hypothesis before prompting. **[Opinion; Extrapolation from MIT]** If you're blocked on setup, tooling or an urgent incident, ask right away.
2. **Get help when progress stops.** daily.dev suggests 20–30 minutes of solo debugging first. Treat that as an example, not a rule. A good signal to ask is that you've stopped making progress. When you ask, bring a hypothesis: *"I think X because Y. What am I missing?"* **[Opinion]**
3. **Read the diff, every line,** and make sure you could debug it if it broke. **[Opinion]**
4. **Ground tests in the requirement, not the implementation.** Write acceptance criteria *before* or *apart from* the AI's code. If the same AI writes the code and the tests, they can share one mistake. **[Opinion]**
5. **Get security-sensitive changes reviewed by someone competent.** For auth, crypto, input handling and permissions, check against official docs and OWASP and get a human review. Tests and scanners don't catch everything. **[Vendor evidence + Opinion]**
6. **Practise fundamentals without AI some of the time.** Data structures, recursion, SQL, HTTP. Some seniors advise typing code by hand until fluent; others disagree. **[Opinion]**
7. **Check API usage against primary sources.** "Almost right" is the most reported frustration. **[Survey]**
8. **Show understanding in more than one way.** Predict what the code will do before running it. Find a bug a mentor plants. Adapt the code to a changed requirement. Solve a related task later without AI. Explaining out loud is useful, but paraphrasing an AI explanation doesn't prove you can reason on your own. **[Opinion]**
9. **Refactor on purpose.** Remove duplication and simplify. **[Opinion; GitClear is context]**
10. **Keep a decision log** of what AI suggested, what you rejected and why. **[Opinion]**
11. **Keep talking to humans.** Don't let the privacy of AI turn you into a "silent silo". ([notthecode](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/)) **[Opinion]**
12. **Start with a small toolset.** One main tool and one way to check your understanding, rather than many. **[Opinion]**
13. **Measure, don't guess.** METR found a large gap between perceived and measured speed. Time some tasks with and without AI. **[RCT, narrow population]**

### D. Skills to invest in
Debugging and reading code; code review; fundamentals (memory, databases, networks, concurrency); system design; written and spoken communication. **[Opinion, broad agreement across practitioner sources]**

### E. An example 90-day plan (adapted from thoughtgears; adjust freely)
- **Month 1:** Learn one AI tool well and check its suggestions. Do occasional fundamentals practice without AI.
- **Month 2:** Ship a small project end to end and keep a decision log.
- **Month 3:** Build judgment: review PRs, write an ADR, and debug real issues where AI is one input, not the first resort.

---

## Onboarding mode playbook

Onboarding has some of the more specific evidence in this report. It is also where a new hire is most tempted to ask AI instead of colleagues.

### Evidence
- **[Obs, Vendor] Daily AI use was associated with a shorter time to 10th PR.** Across six enterprises: 49 days for daily users vs. 91 for non-users. ([DX](https://getdx.com/blog/ai-cuts-developer-onboarding-time-in-half/))
  - Observational, with no published sample sizes.
  - Other factors could explain it: prior experience, task assignment, PR size, review culture, and AI enthusiasts possibly being faster learners anyway.
  - New hires are not all juniors.
  - Reaching a 10th PR shows *delivery*, not *understanding of the system*.
- **[Controlled study] Expert curation matters.** In LACY, at Beko on a 30K+ line legacy finance system, learners using **expert-guided code tours scored 83% vs. 57% with AI-only tours**. ([LACY, FSE 2026](https://arxiv.org/abs/2603.25391)) This is one study in one setting, but it supports having a maintainer check AI-generated onboarding material before newcomers rely on it.
- **[Docs] Anthropic's guidance** suggests asking Claude Code "the questions you'd ask a senior engineer" when you join a new codebase. ([Claude Code best practices](https://code.claude.com/docs/en/best-practices))
- **[Paper]** LLMs may help reduce information overload in onboarding docs. ([ASE 2024](https://dl.acm.org/doi/abs/10.1145/3691620.3695286))
- **[Opinion] The risk:** knowledge about *why* things are the way they are gets passed on in conversation, and AI can't see most of it. Practitioner writing warns that AI can quietly replace those conversations. ([notthecode](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/))

### For the new hire: an example 4-week arc (adapt to your team)

**Early days: Map**
- Start with exploration that doesn't change code, e.g. Claude Code's plan mode. **Note:** "read-only" covers code edits. Running tests, scripts or setup commands can still have side effects, such as touching databases, calling external services or migrating data. Run them only in a disposable local or dev environment with minimal credentials.
- Ask the questions you'd ask a senior engineer: what the project does and for whom; the top-level architecture; how logging, config, auth and errors work; how to add an endpoint, with a good example to copy; and *"summarize the git history of Y"*.
- Use subagents for broad questions so your main session stays focused.
- Sketch the architecture yourself, then compare it with the AI's version and, more importantly, with a teammate's.
- **Don't wait a full week to contribute.** A small, well-scoped fix on day one or two can teach a lot.

**Trace and verify**
- Trace one real request end to end, using AI to answer questions along the way.
- Check AI claims by opening the cited files or running code in a safe environment. AI explanations of *your* codebase can be wrong about conventions and history.
- To see how things fail, break them on purpose **only in a disposable local environment**.
- Keep an onboarding log: questions, answers, and which ones AI got wrong.

**Small contributions**
- Good first issues, small fixes and tests. Use the learning style when the area is new to you.
- Use more than one way to check your understanding (see habit 8). A quiz skill can help, but **don't make an AI-generated quiz the only gate**: it can repeat the same mistake that's in the code.

**Give back**
- Fix gaps in the README, onboarding docs or CLAUDE.md. New hires notice what's missing.
- Move into delivery mode for areas you now understand.

**Throughout:**
- **AI is good for *what/where* questions. People are essential for *why*:** history, past incidents, unwritten rules.
- Meet regularly with a buddy and bring your onboarding log.
- Follow the company's AI and data policy from day one.

### For the team
- **Keep a short, maintained `CLAUDE.md` / `AGENTS.md`** with build and test commands, non-default conventions and gotchas. It helps both new humans and agents. **[Opinion; Anthropic docs]**
- **Keep ADRs and a glossary** so the reasons behind decisions are written down.
- **Have a maintainer review any AI-generated onboarding guide or code tour** before newcomers rely on it. **[Supported by LACY]**
- **Provide a curated starter set:** example PRs, one reference implementation per pattern, and good first issues.
- **Pair AI exploration with a human buddy** who reviews the newcomer's architecture sketch and log.
- **Measure onboarding broadly.** Time-to-10th-PR can be one *team-level* signal, alongside understanding, task difficulty, rework, review burden and independent troubleshooting. **Don't use PR counts as individual performance targets.**

---

## 4. Practices for teams, mentors and managers

*Practitioner recommendations. **[Opinion]***

**From the [Reddit thread](https://www.reddit.com/r/ExperiencedDevs/comments/1po21hq/ai_is_a_death_trap_for_many_junior_devs_how_do_i/) (paraphrased):**
- Several commenters set the review bar at "you must be able to explain your code". Consider offering written or practical ways to show understanding as well as spoken ones.
- Low-quality code doesn't merge, however it was produced.
- Pair program occasionally, include juniors in design discussions, and review code together.
- AI raises velocity expectations. Managers need to give juniors room to slow down and learn.

**Mentoring ([notthecode](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/), [Osmani](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your)):**
- Ask *how* in code review, not just *what*: the reasoning, which parts came from AI and were changed, and how they'd debug it in production.
- Review prompts occasionally, and pair on prompting.
- Add deliberate practice without AI from time to time.
- Seniors can show that it's normal to say "I don't trust this suggestion".
- Evaluate understanding, catching mistakes and learning from feedback, not only features shipped.
- Write a short team AI playbook.

**Organizational ([Dave Stevens](https://www.linkedin.com/pulse/rethinking-junior-developer-age-ai-dave-stevens-ncwuc/), Osmani):**
- Keep hiring juniors to maintain the senior pipeline.
- Clear codebases help both juniors and agents. Juniors force teams to write down tacit knowledge.
- Put senior time into mentoring and design.
- **Tutoring tools can be designed for learning.** Harvard's CS50 Duck uses instructions and filtering to avoid giving direct answers, plus usage limits. ([Harvard Gazette](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/)) That shows intent, not measured outcomes.

---

## 5. Counterpoints and nuance

- **"This isn't new."** Experienced devs made similar complaints about copy-pasting from Stack Overflow. A difference some note: LLM output is tailored to your code, so it can need less engagement than adapting a snippet. **[Opinion]**
- **The evidence is small and narrow.** Anthropic: n=52, with pattern subgroups of 2–7. METR: 16 devs, early-2025 tools, since updated. MIT: essays, n=18 for the switched session. Several sources are surveys, vendor reports or observational. **They answer different questions** and don't all point the same way. Copilot field experiments show output *gains*, for example.
- **Short-term vs. long-term.** A learner might understand less right away but put the saved effort into extra practice and end up ahead. Or they might build dependence. **No study here followed learners long enough to tell which.**
- **Reuse.** Not everyone agrees with "do it all by hand". The common ground is that reuse is fine and blind reuse is not.

---

## 6. One-page checklist (adapt as needed)

**Before prompting**
- [ ] Which mode am I in: learning, onboarding or delivery?
- [ ] Do I have a quick attempt or hypothesis (unless I'm blocked or it's urgent)?

**While prompting**
- [ ] Am I asking *why/how*, or only *give me*?
- [ ] Did I ask for explanations or alternatives along with the code?

**Before committing**
- [ ] I read every line and could debug it.
- [ ] Acceptance criteria and tests are based on the requirement, not only on the AI's code.
- [ ] Security-sensitive parts were checked against official docs *and* reviewed by someone competent.
- [ ] I simplified where I could.
- [ ] I noted what I rejected and why.

---

## Sources

**Your links**
- [Reddit r/ExperiencedDevs: "AI is a death trap for many junior devs"](https://www.reddit.com/r/ExperiencedDevs/comments/1po21hq/ai_is_a_death_trap_for_many_junior_devs_how_do_i/) (practitioner opinion)
- [daily.dev: How junior developers adapt to AI](https://daily.dev/blog/how-junior-developers-adapt-to-ai/) (opinion; its evaluator-ranking statistic is not used here because its source couldn't be traced)
- [Dave Stevens: Rethinking the junior developer in the age of AI (LinkedIn)](https://www.linkedin.com/pulse/rethinking-junior-developer-age-ai-dave-stevens-ncwuc/) (opinion)
- [Zachary Adam: AI-Assisted Development and the Reshaping of Junior Developer Roles (SSRN 6409098)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098) (**unverified**: abstract snippet only)
- [thoughtgears: How to use AI as a junior developer](https://thoughtgears.co.uk/blog/how-to-use-ai-as-a-junior-developer/) (opinion)

**Research and data**
- [Anthropic: How AI assistance impacts the formation of coding skills](https://www.anthropic.com/research/AI-assistance-coding-skills)
- [METR: Early-2025 AI and experienced OSS developer productivity (paper)](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study-paper.pdf) · [Feb 2026 update](https://metr.org/blog/2026-02-24-uplift-update/)
- [Cui et al.: Effects of Generative AI on High-Skilled Work (SSRN)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566)
- [Microsoft Research / CMU: GenAI and critical thinking (CHI 2025)](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/)
- [Kosmyna et al.: Your Brain on ChatGPT (arXiv)](https://arxiv.org/abs/2506.08872)
- [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/blog/genai-code-security-report/)
- [Stack Overflow 2025 Developer Survey: AI](https://survey.stackoverflow.co/2025/ai)
- [GitClear: AI Copilot Code Quality 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research)
- [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report)
- [Stanford Digital Economy Lab: Canaries in the Coal Mine (Nov 2025)](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf)
- [DX: AI cuts onboarding time in half](https://getdx.com/blog/ai-cuts-developer-onboarding-time-in-half/)
- [LACY: Simulating Expert Mentoring for Software Onboarding with Code Tours (FSE 2026)](https://arxiv.org/abs/2603.25391)
- [Towards Leveraging LLMs for Reducing Open Source Onboarding Information Overload (ASE 2024)](https://dl.acm.org/doi/abs/10.1145/3691620.3695286)

**Practitioner writing and docs**
- [Addy Osmani: The 70% problem](https://addyo.substack.com/p/the-70-problem-hard-truths-about) · [AI won't kill junior devs, but your hiring strategy might](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your)
- [notthecode: The Silent Silo](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/)
- [Harvard Gazette: Taming the Duck (CS50 AI tutor)](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/)
- [Claude Code output styles](https://docs.anthropic.com/en/docs/claude-code/output-styles) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices)

**Removed after the review:** the "evaluators rank catching AI mistakes highest (66% vs 28%)" statistic and the "38% say AI reduced mentoring" statistic, because neither could be traced to a primary source. Also removed: the "easy to replace" claim and the "onboarding benefit is largest" claim.
