# Good Practices for Using AI as a Junior Developer

*Research compiled 2026-10-01. It draws on the 5 links you supplied plus about 15 more sources: RCTs, surveys, labor data and practitioner writing.*

---

## TL;DR

1. **How you use AI matters more than whether you use it.** In Anthropic's RCT, juniors who handed coding to AI scored **below 40%** on a comprehension quiz. Juniors who used AI to *ask conceptual questions* or *asked for explanations with the code* scored **65% or higher**, about the same as people who coded by hand.
2. **Think first, then ask the AI.** The MIT EEG study and the daily.dev guidance agree: do the hard thinking yourself, then bring AI in for review, refinement and boilerplate.
3. **Debugging is the skill that suffers most, and the one that matters most.** The biggest gap in the Anthropic study was on debugging questions. Evaluators rank "catching and fixing AI mistakes" as the top skill they look for.
4. **The bar is "can you explain it?", not "does it run?"** The most common rule from experienced devs on Reddit: *if you can't explain the code, it doesn't pass review.*
5. **The job market raises the stakes.** Employment of 22–25-year-old developers is down about 20% from its late-2022 peak (Stanford), and entry-level postings are down about 28% (SSRN). Juniors who can *supervise* AI stand out. Juniors who only *produce output* with it are easy to replace.

---

## 1. What the evidence says

| Finding | Source | What it means for juniors |
|---|---|---|
| AI group scored **50% vs 67%** on a comprehension quiz (17 pts lower, about two letter grades). The speed gain was about 2 min and **not statistically significant**. Largest gap was in **debugging**. | [Anthropic RCT, 52 mostly junior devs learning Trio](https://www.anthropic.com/research/AI-assistance-coding-skills) | Using AI while learning something new costs you understanding and barely saves time. |
| Low scorers (<40%) used **delegation**, **progressive reliance** or **iterative AI debugging**. High scorers (≥65%) used **conceptual inquiry**, **generate-then-comprehend** or **code + explanation**. | same | This is the most actionable result: it is a list of usage patterns to copy and to avoid. |
| Experienced devs were **19% slower** with AI but *believed* they were **20% faster**. | [METR RCT, 2025](https://x.com/METR_Evals/status/1943360399220388093) · [summary](https://letsdatascience.com/blog/developers-thought-ai-made-them-faster-the-data-said-otherwise) | Feeling productive is not the same as being productive. Measure your work, and don't trust the feeling. |
| Copilot gave **+26% completed tasks**, with the **largest gains for less experienced devs**. | [Cui et al., field experiments at Microsoft/Accenture](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) | AI does speed juniors up on *output*. The risk is that output grows while skill does not. |
| The more people **trusted the AI**, the **less critical thinking** they did. **Self-confidence** went with *more* critical thinking. | [Microsoft Research + CMU, CHI 2025, n=319](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/) | Over-trust is the failure mode. Building your own skill keeps your judgment switched on. |
| Writing **unaided first, then with AI** led to better recall and brain engagement. Starting **with AI** left people underperforming when they later worked alone ("cognitive debt"). | [MIT Media Lab "Your Brain on ChatGPT"](https://www.leximancer.com/blog/e34e38yedoeltictpfkf2lms2jfpon) (essay writing, n=54) | Order matters: struggle first, assist second. |
| **45%** of AI-generated code samples had security flaws. XSS defenses failed **86%** of the time. Larger, newer models were not more secure. | [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/blog/genai-code-security-report/) | Never trust AI code that touches security, auth or user input without checking it. |
| **66%** of devs say AI answers are "almost right, but not quite". **45%** lose significant time debugging AI code. Trust in AI accuracy fell to **29%**. | [Stack Overflow Dev Survey 2025](https://stackoverflow.co/internal/resources/2025-stack-overflow-developer-survey-for-leaders/ai-adoption/) | The last 30% of the work is where the real effort and the learning are. |
| Copy-pasted and duplicated code **overtook refactored code** for the first time. Code revised within 2 weeks of being written rose from 3.1% to 5.7%. | [GitClear 2025, 211M lines](https://www.gitclear.com/ai_assistant_code_quality_2025_research) | AI tends to add code rather than improve it. Juniors need to learn to refactor deliberately. |
| AI is an **amplifier**: it increases throughput and also **delivery instability**. | [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report) | Good habits get stronger with AI, and so do bad ones. |
| Employment of devs aged 22–25 is down about **20%** from its late-2022 peak. Older age groups are flat. | [Stanford "Canaries in the Coal Mine"](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf) | Competition at entry level is fierce. Being able to show judgment is what makes you stand out. |
| Entry-level SWE postings are down **28%** since 2022 while their requirements got more complex. It names a "vibe coding" paradox: the tools that make it easy to produce code also erode deliberate practice. | [Adam, SSRN 6409098 (Mar 2026)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098) | Your link. SSRN blocked automated access, so this summary comes from the abstract only. |

---

## 2. The core model: "Supervise, don't delegate"

Every source arrives at the same framing, worded differently:

- *"A fast, occasionally careless colleague, not an oracle."* ([thoughtgears](https://thoughtgears.co.uk/blog/how-to-use-ai-as-a-junior-developer/))
- *"A very eager junior developer on your team"* that needs supervision. ([Addy Osmani](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your))
- *"Juniors who can guide AI and check its work will move faster than juniors who just lean on it."* ([daily.dev](https://daily.dev/blog/how-junior-developers-adapt-to-ai/))
- **Knowledge paradox:** seniors use AI to *speed up what they already know*, while juniors use it to *learn what to do*. That is the riskier use. ([Osmani, "The 70% problem"](https://addyo.substack.com/p/the-70-problem-hard-truths-about))
- *"The engineer's worth isn't delivering… it's being responsible for the code."* (Reddit r/ExperiencedDevs)

---

## 3. Practices for junior developers

### A. Learning mode vs. delivery mode
Decide which mode you're in **before** you open the AI.

| | **Learning mode** (new concept, library or codebase) | **Delivery mode** (things you already understand) |
|---|---|---|
| Goal | Understanding | Throughput |
| AI role | Tutor: conceptual questions, explanations, Socratic hints | Pair programmer: boilerplate, scaffolding, refactors |
| Rule | Write the code yourself. Ask *why*, not *give me*. | Read every line of the diff. Test it. Be able to explain it. |
| Tools | Claude **Learning** output style (`/config → Output style → Learning` in Claude Code, which leaves `TODO(human)` parts for you to write), ChatGPT **Study Mode** | Normal agent/editor modes |

### B. Usage patterns to copy (from the Anthropic RCT)
- ✅ **Conceptual inquiry:** ask only conceptual questions, then write the code yourself. This was the highest-scoring pattern *and* the fastest of the high scorers.
- ✅ **Code + explanation:** "Generate this **and explain each decision**."
- ✅ **Generate then comprehend:** if AI writes it, ask follow-ups until you could rewrite it from scratch.
- ❌ **Delegation:** "just write it."
- ❌ **Progressive reliance:** starting with questions, then handing over everything.
- ❌ **Iterative AI debugging:** pasting errors back in a loop without forming your own hypothesis.

### C. Concrete habits
1. **Attempt first.** Sketch the solution or write a first draft before you prompt. Then use AI to review it. (daily.dev, MIT)
2. **The 20–30 minute rule.** Debug by yourself for 20–30 minutes before asking AI. When you do ask, bring your hypothesis: *"I think X is happening because Y. What am I missing?"* (daily.dev)
3. **Read the diff, every line.** Never accept code you couldn't explain in code review. (thoughtgears, Reddit)
4. **Test everything AI writes.** Write the tests yourself, or at least read the AI's tests critically. Tests are how you prove you understand the code. (thoughtgears)
5. **No AI shortcuts on security.** For auth, crypto, input handling and permissions, check the output against official docs and OWASP. (daily.dev, Veracode)
6. **Do fundamentals by hand.** Data structures, recursion, parsers, SQL and HTTP: practice without AI. Some seniors go further: *"type it out, don't even copy/paste until you're fluent."* (daily.dev, Reddit)
7. **Check against primary sources.** Confirm API usage in the official docs. "Almost right" answers are the most common failure. (Stack Overflow survey)
8. **Explain it out loud** (Feynman technique) to a teammate, a rubber duck or a PR description before you ship. (daily.dev, Dave Stevens)
9. **Refactor on purpose.** AI adds code. Your job includes removing duplication and simplifying. (GitClear)
10. **Keep a decision log.** Record what AI suggested, what you **rejected and why**, and the trade-offs. This also makes strong portfolio and interview material. (daily.dev)
11. **Keep talking to humans.** Don't let AI's privacy turn you into a "silent silo". Asking seniors builds context the AI doesn't have. ([notthecode](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/))
12. **Master one stack.** One AI editor, one terminal agent and one inline assistant. *"Depth in one beats shallow use of many."* (thoughtgears)
13. **Measure, don't guess.** METR shows people overestimate AI speedups by about 40 points. Time some tasks with and without AI.

### D. Skills to invest in (the parts AI doesn't do for you)
- **Debugging and reading code**, especially unfamiliar code and novel failures
- **Code review**: judging code, whether AI or a human wrote it
- **Fundamentals**: memory, databases, networks, concurrency
- **System design**: architecture, abstractions, failure modes (Stevens: the shift from *tactical* to *strategic* programming)
- **Communication**: specs, PR descriptions, explaining trade-offs (this is also what makes prompts good)

### E. A 90-day plan (adapted from thoughtgears)
- **Month 1:** Learn one AI editor in depth and verify every suggestion. Do one fundamentals exercise a week with no AI.
- **Month 2:** Ship a project end to end. Document your AI workflow and decision log.
- **Month 3:** Focus on judgment: review others' PRs, write ADRs, debug real issues with AI as your *second* step.

---

## 4. Practices for teams, mentors and managers

**From the Reddit thread (experienced devs):**
- *"If they can't explain the code, the code shall not pass the review."*
- Set a clear bar: low-quality AI code doesn't get merged to main, and hold to it.
- Pair program occasionally, include juniors in design meetings, and review code together.
- Give honest feedback on the code regardless of how it was produced.
- Recognize that AI raises **velocity expectations**. Management has to give juniors room to *"take a step back to leap forward."*

**Mentoring ([notthecode "Silent Silo"](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/), [Osmani](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your)):**
- **Reframe code review** from *what* to *how*: "Walk me through your reasoning", "Which parts came from AI and what did you change?", "If this fails in prod, how would you debug it?"
- **Review prompts too.** The input reveals the thinking behind the code.
- **Pair on prompting** to show how seniors critique, reject and refactor AI output.
- **Add deliberate friction**: occasional exercises without AI.
- **Model uncertainty**: seniors saying "I don't trust this suggestion" makes it normal to question AI.
- **Change what you evaluate**: from "features shipped" to "can explain, catches AI mistakes, learns from feedback".
- Write a team **AI playbook** (where AI is fine, where it's restricted, and the review expectations).

**Organizational ([Dave Stevens](https://www.linkedin.com/pulse/rethinking-junior-developer-age-ai-dave-stevens-ncwuc/), Osmani, DORA):**
- **Keep hiring juniors.** Cutting them hollows out the future senior pipeline.
- **Codebases that are easy for juniors to understand are also easy for AI agents**: clear boundaries, conventions and docs. Juniors force teams to write down tacit knowledge.
- Juniors' "why" questions get more valuable as prototyping gets cheap.
- Put senior time into mentoring and design rather than only individual coding.
- **Learning-oriented tool design works.** Harvard's CS50 Duck is told not to give answers, its output is filtered for compliance, and a "hearts" limit discourages over-asking. ([Harvard Gazette](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/))

---

## 5. Counterpoints and nuance

- **"This isn't new."** Many experienced devs note the same complaints about copy-pasting from Stack Overflow. The difference: LLM output is *tailored* to slot straight into your code, so you engage with it even less than with an SO snippet you had to adapt. (Reddit)
- **Studies are small or narrow.** Anthropic: n=52, and some pattern subgroups have only 2–7 people. METR: 16 senior devs on large OSS repos, with early-2025 tools. MIT: essays, not code. The direction is consistent across studies, but treat exact numbers as indicative.
- **AI really does help juniors produce output** (Copilot field experiments). The concern is long-term skill formation, not short-term productivity.
- **Not everyone agrees with "do it all by hand."** Some seniors argue the real skill is solving high-level problems and reusing existing solutions. The consensus line is: **reuse is fine, blind reuse is not.**

---

## 6. One-page checklist

**Before prompting**
- [ ] Am I in learning mode or delivery mode?
- [ ] Have I tried it or sketched it myself first?
- [ ] Do I have a hypothesis (when debugging)?

**While prompting**
- [ ] Am I asking *why/how*, or only *give me*?
- [ ] Did I ask for explanations or alternatives along with the code?

**Before committing**
- [ ] I read every line and could explain it in review.
- [ ] Tests exist and I understand what they check.
- [ ] Security-sensitive parts were checked against official docs.
- [ ] I removed duplication and simplified where I could.
- [ ] I noted what I rejected and why.

---

## Sources

**Your links**
- [Reddit r/ExperiencedDevs: "AI is a death trap for many junior devs"](https://www.reddit.com/r/ExperiencedDevs/comments/1po21hq/ai_is_a_death_trap_for_many_junior_devs_how_do_i/)
- [daily.dev: How junior developers adapt to AI](https://daily.dev/blog/how-junior-developers-adapt-to-ai/)
- [Dave Stevens: Rethinking the junior developer in the age of AI (LinkedIn)](https://www.linkedin.com/pulse/rethinking-junior-developer-age-ai-dave-stevens-ncwuc/)
- [Zachary Adam: AI-Assisted Development and the Reshaping of Junior Developer Roles (SSRN 6409098)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6409098). Only the abstract was available.
- [thoughtgears: How to use AI as a junior developer](https://thoughtgears.co.uk/blog/how-to-use-ai-as-a-junior-developer/)

**Additional research**
- [Anthropic: How AI assistance impacts the formation of coding skills](https://www.anthropic.com/research/AI-assistance-coding-skills)
- [METR: Early-2025 AI and experienced OSS developer productivity](https://x.com/METR_Evals/status/1943360399220388093)
- [Cui et al.: Effects of Generative AI on High-Skilled Work (SSRN)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566)
- [Microsoft Research / CMU: GenAI and critical thinking (CHI 2025)](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/)
- [MIT Media Lab: Your Brain on ChatGPT (summary)](https://www.leximancer.com/blog/e34e38yedoeltictpfkf2lms2jfpon)
- [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/blog/genai-code-security-report/)
- [Stack Overflow 2025 Developer Survey: AI](https://stackoverflow.co/internal/resources/2025-stack-overflow-developer-survey-for-leaders/ai-adoption/)
- [GitClear: AI Copilot Code Quality 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research)
- [DORA 2025: State of AI-assisted Software Development](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report)
- [Stanford Digital Economy Lab: Canaries in the Coal Mine](https://digitaleconomy.stanford.edu/app/uploads/2025/11/CanariesintheCoalMine_Nov25.pdf)
- [Addy Osmani: The 70% problem](https://addyo.substack.com/p/the-70-problem-hard-truths-about)
- [Addy Osmani: AI won't kill junior devs, but your hiring strategy might](https://addyo.substack.com/p/ai-wont-kill-junior-devs-but-your)
- [notthecode: The Silent Silo, mentoring junior developers in the age of AI](https://notthecode.com/silent-silo-mentoring-junior-developers-ai/)
- [Harvard Gazette: Taming the Duck (CS50 AI tutor)](https://news.harvard.edu/gazette/story/2026/09/taming-the-duck-for-starters/)
- [Claude Code output styles (Learning mode)](https://docs.anthropic.com/en/docs/claude-code/output-styles)
