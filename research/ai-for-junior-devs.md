# Good Practices for Using AI as a Junior Developer

*Compiled 2026-10-01 and revised the same day after five adversarial reviews ([round 1](adversarial-review.md), [round 2](adversarial-review-round-2.md), [round 3](adversarial-review-round-3.md), [round 4](adversarial-review-round-4.md), [round 5](adversarial-review-round-5.md)). It draws on the 5 links you supplied plus about 20 more sources: trials, surveys, observational and labor data, vendor reports and practitioner writing.*

**How to read this report.** Inline tags describe **study design** only:

| Design tag | Meaning |
|---|---|
| **[RCT]** | Randomized experiment. Supports causal claims about the *randomized* factor, for the population, task and time horizon studied. |
| **[RCT, post hoc]** | Observational analysis *inside* an RCT: subgroups the participants fell into, not ones they were randomized into. Shows associations only. |
| **[Controlled]** | Controlled comparison that isn't a full RCT, e.g. a small within-subject study. |
| **[Obs]** | Observational or correlational data. Shows associations, not causes. |
| **[Survey]** | Self-reported data. |
| **[Opinion]** | Practitioner advice. Plausible, but not tested. |
| **[Extrapolation]** | Evidence from another domain (e.g. essays, math, physics) applied to coding by this report. |

Who produced each study (e.g. a vendor with a commercial interest) and how far this report verified it are listed separately, in the evidence table's **Provenance / verification** column. The practices in sections 3 to 6 are **proposed habits** built on that evidence. They are not validated interventions.

---

## TL;DR

1. **AI access lowered immediate comprehension in one learning experiment.** In Anthropic's randomized study (52 mostly junior Python devs learning an unfamiliar library), the AI group averaged **50% vs. 67%** on a quiz taken right after the task. The study did not establish a completion-time benefit: the roughly 2-minute difference was not statistically significant, which is not the same as showing there was no speed effect. [RCT] Long-term effects were not measured.
2. **How people used AI was *associated* with different scores.** Participants who asked conceptual questions or asked for explanations scored higher on average than those who delegated. Those groups were small and self-selected, so this suggests habits to try. It does not prove a recipe. [RCT, post hoc]
3. **Tool design can change learning outcomes (outside coding).** In a high-school math field experiment, students who practised with unrestricted GPT-4 scored **17% lower (relative)** on an unaided exam than students who **never had AI access**. The exam came at the end of the same 90-minute session. A tutor version with learning safeguards largely mitigated that drop. [RCT, Extrapolation] In a crossover trial with 194 Harvard physics students, a custom, self-paced AI tutor used at home outperformed an in-class active-learning lesson on post-lesson tests. [RCT, Extrapolation] That trial evaluates a whole *package* (AI, self-pacing, location) and doesn't isolate the AI or measure delayed retention. Neither study tested coding agents or the skills in the companion report.
4. **Debugging showed the largest gap** in the Anthropic study. Practising independent debugging is a reasonable priority. [RCT]
5. **Use several complementary checks of your understanding:** explaining the code, predicting behavior, finding a bug planted on purpose, and later solving a related task without help. They measure different things; none is a complete test. [Opinion]
6. **The entry-level market has tightened.** Employment of US software developers aged 22–25 fell nearly 20% from late 2022 to September 2025 (payroll data). [Obs] That doesn't show which individual habits protect a career.
7. **Daily AI use was *associated* with faster onboarding.** In six enterprises, daily users reached their 10th PR in 49 days vs. 91 for non-users. [Obs] In a 5-learner industry study, expert-prepared AI-assisted code tours with podcasts scored 83% vs. 57% for learner-generated AI-only tours on a quiz. Experts rated the learners' explanations much closer (79% vs. 77%). [Controlled] See the [Onboarding mode playbook](#onboarding-mode-playbook).

---

## 1. What the evidence says

| Finding | Design | Population · outcome · time horizon | Provenance / verification | Reasonable takeaway |
|---|---|---|---|---|
| AI group scored **50% vs. 67%** on a quiz. The time difference (about 2 min) was **not significant**. Largest gap on **debugging** questions. | RCT (AI access randomized) | 52 mostly junior Python users new to the Trio library · comprehension quiz · **immediately after** a short task | [Anthropic](https://www.anthropic.com/research/AI-assistance-coding-skills) (AI vendor). Publisher summary read. | AI access lowered average immediate quiz performance in this task. The study did **not establish** a completion-time benefit, but a nonsignificant result doesn't show the speed effect is zero. Lasting effects unknown. |
| Delegation-style use averaged **<40%**. Conceptual inquiry and code + explanation averaged **≥65%**. | **RCT, post hoc** (usage patterns *not* randomized) | Same; subgroups of n=2–7 | Same. Anthropic says this analysis does not establish causality. | Habits worth trying, not a proven recipe. |
| Compared with a **textbook-only control**, unrestricted GPT-4 ("GPT Base") raised assisted-practice scores by **48%** and a safeguarded "GPT Tutor" by **127%** (both relative). On the **unaided exam**, GPT Base students scored **17% lower (relative) than the control**. The tutor arm **largely mitigated** this but showed no positive exam effect. | RCT (field) | Nearly 1,000 students in grades 9–11 in Turkey, four 90-min sessions, Fall 2023 · assisted practice, then a closed-book exam **in the same session** on problems similar to the practice ones | [Bastani et al., PNAS 2025](https://doi.org/10.1073/pnas.2422633122) ([abstract](https://pubmed.ncbi.nlm.nih.gov/40560616/)). Academic. Full text read via PMC. | **[Extrapolation]** Tool design matters, and so does measuring *unaided* performance separately from assisted performance. This is same-session transfer, not durable learning. |
| Students scored significantly higher on post-tests, with less time on task, after a lesson from a **custom, self-paced AI tutor used at home** than after an **in-class active-learning lesson** on the same material. | RCT (crossover) | **194** Harvard undergraduate physics students · **two lessons in consecutive weeks**, each student getting both conditions · post-test **right after each lesson**, plus engagement self-reports | [Kestin et al., Sci Rep 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/). Academic. Full text read via PMC. | **[Extrapolation]** Supports this *instructional package* in this setting. It doesn't separate the AI from self-pacing or location, doesn't measure delayed retention, and doesn't test junior programmers or any skill package. |
| Experienced devs had **19% longer completion times** with AI, while *believing* they were about 20% faster. | RCT | 16 experienced devs on their own large OSS repos · task time · early-2025 tools | [METR paper](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study-paper.pdf) (nonprofit). The [Feb 2026 update](https://metr.org/blog/2026-02-24-uplift-update/) says later tools likely speed devs up more, but its estimates are unreliable because of selection effects. | Perceived speedups can differ greatly from measured ones. |
| The authors' pooled instrumental-variable estimate: Copilot adoption raised weekly completed tasks by **26.08% (SE 10.3 percentage points)**. Less experienced devs adopted more and gained more. | Field RCTs, IV (LATE) analysis | 4,867 devs at Microsoft, Accenture and an anonymous company · weekly completed tasks (output) · 2–8 months · **2022–2023 Copilot autocomplete** | [Cui, Demirer et al. (author-hosted)](https://www.mertdemirer.com/Papers/Demirer_AI_productivity.pdf) · [SSRN](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566). Co-authors include Microsoft researchers. Paper read. | A local average treatment effect: the effect for developers whose adoption was **induced by random assignment**, under the paper's identifying assumptions. That is neither the effect of merely offering access nor necessarily the effect for every adopter. It concerns autocomplete-era tools, not today's agents, and it measures output, not learning. |
| Higher confidence in AI was **associated** with less *self-reported* critical thinking. Higher self-confidence was associated with more. | Survey | 319 knowledge workers (not only developers) · self-report | [Microsoft Research + CMU, CHI 2025](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/). Summary read. | Watch for over-trust. |
| Participants who wrote **unaided first, then with an LLM** showed better recall and broader brain activity than those who switched the other way. | Experiment (EEG) | **Essay writing.** 54 participants in sessions 1–3, **18** in the switched session 4 · EEG and recall | [Kosmyna et al. (arXiv)](https://arxiv.org/abs/2506.08872). Preprint. Abstract read. | **[Extrapolation]** "Attempt first" is a reasonable habit to try, not proven for coding. The math-tutor study above suggests well-designed *assistance* can also work. |
| **45%** of AI-generated samples failed security tests. XSS defenses failed in **86%** of relevant samples. | Benchmark | About 80 tasks across 100+ models · security test pass/fail | [Veracode 2025](https://www.veracode.com/blog/genai-code-security-report/) (security vendor). Summary read. | Security-sensitive AI code needs competent review. These are not population-wide failure rates. |
| **66%** report AI solutions that are "almost right, but not quite". **45%** report that debugging AI code takes more time. About **33%** trust AI accuracy (3.1% highly + 29.6% somewhat), about **46%** distrust it. | Survey | Among respondents to each question: **33,244** for the trust item and **31,476** for the frustrations item (about 49K took the survey overall) · self-report | [Stack Overflow 2025](https://survey.stackoverflow.co/2025/ai). Original survey read. | Expect near-misses. These figures are reported frustration, not measured time lost. |
| Copy-pasted and duplicated lines **overtook "moved" lines** in 2024. Code revised within 2 weeks rose from 3.1% to 5.7%. | Obs | 211M changed lines, 2020–2024 · proxy metrics | [GitClear 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research) (dev-analytics vendor). Summary read. | Refactoring on purpose is sound advice regardless of the cause. The data doesn't show AI caused the trend. |
| AI adoption is **associated** with higher throughput and with more delivery instability. | Survey/Obs | Organizations worldwide | [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report) (Google). Summary read. | Team practices matter as much as the tool. |
| Employment of devs aged 22–25 is down **nearly 20%** from its late-2022 peak to **September 2025**. Older age groups are flat or growing. | Obs | US payroll records (ADP) · employment counts | [Stanford "Canaries in the Coal Mine" (Nov 2025)](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf). Academic. Paper text checked. | The entry-level market is tighter. Says nothing about which individual habits help. |
| Entry-level SWE postings are down **28%** since 2022, according to the author. | Paper (methods not reviewed) | n/a | [Adam, SSRN 6409098](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098). Title, author and the claim appear in the indexed abstract. The analysis itself is **not verified**. | Treat as the author's claim, not as established. |

**Note on selection.** Most sources here study either dependence or productivity. The two education trials above are the main outcome evidence about structured AI *tutoring*, and both come from outside software engineering. They evaluate whole tutoring packages, not individual techniques such as Socratic questioning. In programming education, CS50 has published *behavioral* evaluations of its tutor (section 4), but not causal learning outcomes. A systematic search might change the balance.

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
**[RCT, post hoc]** These come from a post hoc analysis of the Anthropic study: participants fell into these patterns themselves (n=2–7 per group). Treat them as **habits to try**, not proven techniques. Note that the actor here is the *human*. In "conceptual inquiry", the learner asks the questions and writes the code. That is not the same as a tutor that questions the learner, although both aim to keep the learner engaged.

| Pattern | Group size | Average quiz score |
|---|---|---|
| ➕ **Conceptual inquiry:** ask only conceptual questions, then write the code yourself | n=7 | ≥65% average; also the fastest of the high-scoring groups |
| ➕ **Code + explanation:** ask for code *and* the reasoning behind each decision | n=3 | ≥65% average |
| ➕ **Generate then comprehend:** if AI writes it, ask follow-up questions until you understand it | n=2 | ≥65% average |
| ➖ **Delegation:** "just write it" | n=4 | <40% average |
| ➖ **Progressive reliance:** starts with questions, then hands everything over | n=4 | <40% average |
| ➖ **Iterative AI debugging:** pastes errors back in a loop without forming a hypothesis | n=4 | <40% average |

### C. Habits worth trying
1. **Attempt first, when it's practical.** A short sketch or hypothesis before prompting. **[Opinion; Extrapolation from MIT]** This is not always the better order. In the math study, a tutor *designed with learning safeguards* helped during practice without the drop on the same-session unaided exam. What seems to matter is that the assistance keeps you doing the thinking, not that you always go first. If you're blocked on setup, tooling or an urgent incident, ask right away.
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
13. **Measure, don't guess, and measure fairly.** METR found a large gap between perceived and measured speed. **[RCT, narrow population]** For a personal comparison:
    - Don't repeat the same task, because the second run benefits from the first.
    - Use *comparable* tasks, alternate which ones get AI, and note difficulty and how familiar you already were.
    - Count end-to-end effort: prompting, verifying, review comments received, rework and later fixes, not just typing time.
    - Track *aided delivery*, *immediate understanding* and *later unaided performance* separately.
    - Treat a handful of tasks as a personal observation, not a result.

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
- **[Obs] Daily AI use was associated with a shorter time to 10th PR.** Across six enterprises: 49 days for daily users vs. 91 for non-users. ([DX](https://getdx.com/blog/ai-cuts-developer-onboarding-time-in-half/); DX sells developer-productivity analytics.)
  - Observational, with no published sample sizes.
  - Other factors could explain it: prior experience, task assignment, PR size, review culture, and AI enthusiasts possibly being faster learners anyway.
  - New hires are not all juniors.
  - Reaching a 10th PR shows *delivery*, not *understanding of the system*.
- **[Controlled] Expert involvement may help; small study.** LACY was run at Beko on a 30K+ line legacy finance system. **Five learners** (average 4.6 years of programming experience) each used two kinds of tour:
  - expert-prepared, AI-assisted code tours **with podcasts**: 83% on a 10-question quiz, about 25 minutes
  - tours they generated themselves with AI and no expert input: 57%, about 35 minutes

  Order and features were counterbalanced, and there was no traditional-documentation baseline. Expert ratings of learners' explanations were much closer (79% vs. 76.8%). The two conditions differ in several ways (expert input, structure, podcasts), so this **doesn't isolate** the effect of an expert reviewing a generated guide. It is *consistent with* expert-guided onboarding being valuable. ([LACY methods and results](https://arxiv.org/html/2603.25391v1))
- **[Opinion; vendor documentation] Anthropic's guidance** suggests asking Claude Code "the questions you'd ask a senior engineer" when you join a new codebase. ([Claude Code best practices](https://code.claude.com/docs/en/best-practices))
- **[Research paper; design not reviewed here]** LLMs may help reduce information overload in onboarding docs. ([ASE 2024](https://dl.acm.org/doi/abs/10.1145/3691620.3695286))
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
- **Have a maintainer review AI-generated onboarding guides or code tours** before newcomers rely on them, as far as time allows. **[Opinion; consistent with LACY]** How much it helps and what it costs will vary by team.
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
- **Learning-oriented tutors need ongoing behavioral checks.** Harvard's CS50 Duck uses instructions and filtering to avoid giving direct answers, plus usage limits. ([Harvard Gazette](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/)) Its team has published measurements of how the tutor actually behaves ([Liu et al., SIGCSE 2025](https://cs.harvard.edu/malan/publications/fp0627-liu.pdf)):
  - **Despite instructions not to give solutions**, about **22% of 10 million responses** and **48% of 1.3 million conversations** contained a Markdown code block. That rate rose unexpectedly after switching from GPT-4 to GPT-4o. A code block is an *indicator*, not proof of a full homework answer.
  - In a comparison of the original prompt against a more question-led one (29 teaching fellows, 24 of whom finished all 50 queries; 1,309 judgments), preferences **split** and varied by task type. More experienced fellows leaned towards the question-led version.
  - These are measures of tutor behavior and reviewer preference, **not** of student retention.

  **Practical step:** if you deploy a tutoring skill or prompt for juniors, regularly review a sample of real conversations for answer leakage, useful hints, and appropriate escalation when someone is stuck. Repeat the check after any model or prompt change. Decide in advance which code examples are acceptable, so not every code block counts as a failure.

---

## 5. Counterpoints and nuance

- **"This isn't new."** Experienced devs made similar complaints about copy-pasting from Stack Overflow. A difference some note: LLM output is tailored to your code, so it can need less engagement than adapting a snippet. **[Opinion]**
- **The evidence is small and narrow.** Anthropic: n=52, with pattern subgroups of 2–7. METR: 16 devs, early-2025 tools, since updated. MIT: essays, n=18 for the switched session. Several sources are surveys, vendor reports or observational. **They answer different questions** and don't all point the same way. Copilot field experiments show output *gains*, for example. Education trials in math and physics show that structured AI tutoring *packages* can support learning in those settings, on short horizons. They don't show which component is responsible.
- **Short-term vs. long-term.** A learner might understand less right away but put the saved effort into extra practice and end up ahead. Or they might build dependence. **No coding study here followed learners long enough to tell which.** The math study separated assisted practice from an unaided exam, but the exam came **at the end of the same session**. It shows short-term transfer, not durable retention. It found harm from unrestricted access, largely mitigated by a safeguarded tutor.
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
- [ ] If an AI review tool checked my change, I confirmed it covered my *uncommitted* edits. Some review skills only diff committed changes.

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
- [Cui, Demirer, Jaffe, Musolff, Peng, Salz: The Effects of Generative AI on High-Skilled Work (author-hosted PDF)](https://www.mertdemirer.com/Papers/Demirer_AI_productivity.pdf) · [SSRN](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566)
- [Bastani et al.: Generative AI without guardrails can harm learning (PNAS 2025)](https://doi.org/10.1073/pnas.2422633122)
- [Kestin et al.: AI tutoring outperforms in-class active learning (Sci Rep 2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/)
- [Microsoft Research / CMU: GenAI and critical thinking (CHI 2025)](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/)
- [Kosmyna et al.: Your Brain on ChatGPT (arXiv)](https://arxiv.org/abs/2506.08872)
- [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/blog/genai-code-security-report/)
- [Stack Overflow 2025 Developer Survey: AI](https://survey.stackoverflow.co/2025/ai)
- [GitClear: AI Copilot Code Quality 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research)
- [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report)
- [Stanford Digital Economy Lab: Canaries in the Coal Mine (Nov 2025)](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf)
- [DX: AI cuts onboarding time in half](https://getdx.com/blog/ai-cuts-developer-onboarding-time-in-half/)
- [LACY: Simulating Expert Mentoring for Software Onboarding with Code Tours (FSE 2026)](https://arxiv.org/html/2603.25391v1)
- [Towards Leveraging LLMs for Reducing Open Source Onboarding Information Overload (ASE 2024)](https://dl.acm.org/doi/abs/10.1145/3691620.3695286)

**Practitioner writing and docs**
- [Addy Osmani: The 70% problem](https://addyo.substack.com/p/the-70-problem-hard-truths-about) · [AI won't kill junior devs, but your hiring strategy might](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your)
- [notthecode: The Silent Silo](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/)
- [Harvard Gazette: Taming the Duck (CS50 AI tutor)](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/) · [Liu et al.: Improving AI in CS50 (SIGCSE 2025)](https://cs.harvard.edu/malan/publications/fp0627-liu.pdf)
- [Claude Code output styles](https://docs.anthropic.com/en/docs/claude-code/output-styles) · [Claude Code best practices](https://code.claude.com/docs/en/best-practices)

**Corrected after review round 5:** a checklist item now covers the scope of AI review tools. In the skills report, the `code-review` diff scope, the `tdd` loop and the `teach` workspace are described from source.

**Corrected after review round 4:** the physics trial's sample, crossover design and post-lesson timing are stated, and it is framed as evidence for a package. CS50's published behavioral evaluations are added, along with a conversation-review step.

**Corrected after review round 3:** the Anthropic time result is no longer read as showing "little benefit". The math study's comparison group, relative units and same-session exam timing are stated. The Copilot estimate is described as a LATE for 2022–2023 autocomplete with its SE in percentage points. Stack Overflow figures now use per-question response counts.

**Corrected after review round 2:** the Stanford endpoint is now September 2025. LACY now includes its 5-learner sample and the differences between its conditions. The Copilot estimate is now cited from the paper as the effect of usage for adopters, with its SE. Two non-coding tutoring RCTs were added. Evidence tags now describe design only.

**Removed after review round 1:** the "evaluators rank catching AI mistakes highest (66% vs 28%)" statistic and the "38% say AI reduced mentoring" statistic, because neither could be traced to a primary source. Also removed: the "easy to replace" claim and the "onboarding benefit is largest" claim.
