# Phase 0 interview guide

*Part of the [jrdev.ai plan](../jrdev-ai-plan.md), Phase 0 (discovery). Draft 2026-10-02. Interviewer: Ivan (research owner). The Portuguese script is in the [appendix](#appendix-roteiro-em-português).*

## Purpose

**Questions to answer:**
1. **The Phase 0 gate:** is "the AI does it for me" (autopilot fixing, rubber-stamp approval, not learning) a top problem for our juniors?
2. **The early signals:** which of the [early-signal themes](../jrdev-ai-plan.md#early-signals-2026-10-02) recur beyond the first two people?
3. **Feature candidates:** which of [C1–C6](../specs/03-modes-and-workflows.md#feature-candidates-from-early-signals) and Typing mode address problems people actually describe, and which would they switch off?
4. **The name:** how does "jrdev" land?

**What we're not doing:** pitching jrdev or validating our own ideas. We learn from **specific past behavior**, not opinions about hypothetical features.

## Before the first interview (gate)

Interviews are a data-collection channel, so the [per-channel readiness checks](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03) must pass first. Also check:
- [ ] **Consent form** with separate scopes (P14-04):
  - (a) use for product research (required to take part)
  - (b) anonymized paraphrases in internal docs (required)
  - (c) paraphrases of what you said in the **public** plan on GitHub (optional; declining doesn't affect participation)
  - (d) verbatim quotes, public only together with (c) (optional)
  - (e) audio recording (optional)

  The form also states that **aggregate counts** (e.g. "6 of 10 juniors described this") are published in the public plan for everyone interviewed, without any individual detail.
- [ ] **Storage:** notes and any recordings live in **one private location** owned by Ivan, outside the repo. The retention and deletion steps are written down and tested with a fake participant.
- [ ] **Transcription:** if a transcription service is used, it's disclosed in the consent form along with its retention. Otherwise, notes only.
- [ ] **[Employer-pilot safeguards](../jrdev-ai-plan.md#employer-pilot-safeguards) apply to company juniors:**
  - participation is voluntary
  - managers aren't told who took part
  - nothing is used in performance reviews
- [ ] **Pseudonyms:** each participant gets a code (`J01`…, `M01`…). The name ↔ code list is kept separately.

## Participants and format

- **Who:** **10 juniors** (0–2 years of experience, backend, at the company; external juniors are fine if the company pool is small) and **5 mentors** (people who review juniors' code).
- **Format:** 30–40 minutes, remote, one interviewer. Recording only with consent (scope e).
- **Company code:** ask participants **not** to share company code on screen. Describe tasks in general terms.

## Interview principles

- **Ask about the last time,** not "usually" or "would you". "Tell me about the last time you…" gets real behavior.
- **Don't pitch, don't defend.** If they criticize an idea, ask "what makes you say that?" and write it down.
- **Stay neutral.** Avoid leading questions such as "Don't you think the AI makes you lazy?"
- **Leave silence** after an answer; people often add the most useful part.
- **Note specific instances separately** from general opinions. A specific instance is stronger evidence.
- **Concepts come last,** so they don't shape the earlier answers.

## Junior interview (≈ 35 min)

| # | Section | Time | Questions and probes |
|---|---|---|---|
| 1 | Intro and consent | 3 min | Purpose, voluntary, no performance use, what we record. Confirm the consent scopes. **Don't mention any theme or feature yet** |
| 2 | Background | 3 min | Role, time as a developer, main stack, which AI tools, how often |
| 3 | **Spontaneous difficulties, ranked** | 4 min | **Before any theme is named:** "What's hardest for you about developing with AI?" (let them answer freely) → "Which three matter most to you, in order?" Write the list down as given; it's the only spontaneous ranking |
| 4 | **Last task walkthrough** | 11 min | "Tell me about the last task you did with an AI tool, from start to finish." Probes:<br>• Who wrote the first version, you or the AI?<br>• **When something didn't work, what did you do next? And after that?** *(T1, autopilot loop)*<br>• **How did you decide whether to accept a change? What did you look at?** *(T2, approval)*<br>• Did you do anything else while the AI was working? *(T3, focus)*<br>• Could you explain what changed to a colleague right now? Which part would be hardest? *(T4, understanding)*<br>• How did you prepare the review or PR? *(T4, summary)*<br>• **For any instance above: "What did that cost you?"** (time, rework, a review comment, a bug later, not being able to explain it) **"How much does it matter to you?"** |
| 5 | Learning and debugging | 8 min | • "Think of something you learned in the last month. How did you learn it?" *(T5)*<br>• "Tell me about the last bug you fixed. What did you do first?" *(T6)*<br>• "When you're stuck, who or what do you go to first? When did you last ask a person?" *(T7, silent silo)* |
| 6 | Concept reactions | 4 min | Read the four one-line, neutral descriptions (below). For each: "When would this help you? When would it get in the way? Would you turn it off?" |
| 7 | Name | 1 min | "What does the name 'jrdev' suggest to you?" |
| 8 | Wrap-up | 1 min | "Anything we didn't ask that we should have?" "Who else should we talk to?" |

**Concept descriptions** (read them as written, in rotating order):
- **Typing:** "When you're learning something, the AI explains and gives hints, but you type the code yourself."
- **Check-ins (C2):** "After a set number of actions, the AI stops, summarizes what it did, and waits for you to reply."
- **Change summary (C6):** "Before a PR, a short plain-language summary of what changed, why, and what to review."
- **Loop nudge (C1):** "If you report several failures in a row, the AI asks what you think the cause is before fixing it."

## Mentor interview (≈ 30 min)

| # | Section | Questions |
|---|---|---|
| 1 | Intro and consent | As above |
| 2 | Recent review | "Tell me about the last PR from a junior that had AI-generated code. What did you notice? What did you ask them?" *(T2, T4)* |
| 3 | Patterns | "Have you seen a junior unable to explain their own change? What happened next?" "How do juniors ask you for help now compared with before AI tools?" *(T7)* |
| 4 | Their time | "How much of your review time goes on AI-generated code? What makes it slow?" |
| 5 | Concepts | The same four descriptions. Also: "Would you want to see anything about a junior's learning? What would feel like surveillance?" |
| 6 | Wrap-up | As above |

## Note template

One note per interview, in the private store. **Never** in the repo.

```
Code: J07 · Date · Role/experience · Stack · AI tools · Consent scopes: a b c d e
Spontaneous top 3 (section 3, before any theme was named): 1 … 2 … 3 …
Walkthrough (paraphrased): …
Specific instances by theme: T1 … T2 … T3 … (one line each, mark "instance" or "opinion"; add the stated cost and "matters: yes/no")
Concept reactions: Typing … C2 … C6 … C1 … (helps when / hurts when / would turn off?)
Name reaction: …
Quotes (only with scope d): …
```

## Codebook

**Themes:**

| Tag | Theme |
|---|---|
| T1 | Autopilot fix loop |
| T2 | Rubber-stamp approval |
| T3 | Losing focus, parallel runs |
| T4 | Not understanding changes or decisions |
| T5 | Feeling of not learning |
| T6 | Debugging approach |
| T7 | Help-seeking, the "silent silo" |
| T8 | Resources and performance |
| T9 | Satisfied delivery-mode use |

**Concepts:** Typing, C1, C2, C6, Name.

## Synthesis and decision rules (set before the first interview)

- **Count participants, not mentions.** A theme counts for a participant only if they described a **specific instance**.
- **Two levels of evidence (P13-04):**
  - **Pattern observed:** a specific instance of T1, T2 or T5.
  - **Important problem:** T1, T2 or T5 appears in the participant's **spontaneous top 3** (section 3), *or* they described a specific instance with a **concrete cost** and said it matters to them.

  Patterns without importance are reported, but they don't count towards the gate.
- **Phase 0 gate (juniors).** This is how the plan's "top problem" requirement is measured. Use shares of the juniors actually interviewed:

  | Result | Decision |
  |---|---|
  | **Important problem** for ≥ 50% of juniors, **and** T1/T2/T5 in the spontaneous top 3 for ≥ 30% | **Proceed** |
  | Important problem for < 30% | **Stop or pivot** |
  | Anything else | **Revise** (refocus the product on the themes that did recur) |

  **Minimum sample:** 6 juniors. Below that there's no gate decision: keep recruiting (external juniors are allowed). Report the actual n and both shares.
- **Dry run before the first interview:** classify two synthetic sets with these rules.
  - **Set A:** 5 of 10 juniors describe a prompted, low-cost instance, and none puts T1/T2/T5 in their top 3. It must **not** yield Proceed.
  - **Set B:** 6 of 10 describe instances with concrete costs, and 4 rank one spontaneously. It must yield Proceed.
- **Candidate eligibility (P13-05).** This is one input to the single [candidate promotion decision](../specs/03-modes-and-workflows.md#candidate-promotion-p13-05); it doesn't schedule anything by itself.
  - A **presented** concept (Typing, C1, C2, C6) is eligible if **≥ 3 juniors** describe the problem it targets as a specific instance, **and** fewer than half of the juniors it was presented to say they'd turn it off.
  - **Concepts not presented (C3, C4, C5) are unevaluated.** Report how many juniors described their target problem, but treat reactions as unknown, never as zero objections.
  - Mentor input can raise or lower a candidate's priority, but can't make one eligible on its own.
- **Name:** if **≥ 3** of the 15 participants find "jrdev" patronizing or off-putting, shortlist alternatives before buying the domain.
- **Output (P14-04):** two versions, both checked against **current** consent at the time of writing, so a withdrawal before synthesis changes them:
  - **Internal synthesis** (private store): paraphrases from everyone with scope (b).
  - **Public synthesis,** in the plan's early-signals section, replacing the n = 2 signal:
    - **aggregate counts** for everyone interviewed: theme counts at both evidence levels, gate shares, concept reaction tallies
    - **paraphrased themes or reactions** only from participants with scope (c), and only after the identifiability check: no distinctive workplace incident, team or project detail
    - **quotes** only with scopes (c) and (d)
  - No raw notes go into the repo.
- **Dry run for consent:** with synthetic notes where some participants have only scopes (a) and (b), the public synthesis contains their counts and nothing derived from their individual accounts.

---

## Appendix: Roteiro em português

**Abertura:** "Obrigado por participar. É voluntário, você pode parar a qualquer momento, e nada disso é usado em avaliação de desempenho, e seu gestor não fica sabendo quem participou. Posso tomar notas? Os números agregados (por exemplo, '6 de 10 juniores') vão para o plano público no GitHub, sem nada individual. Posso usar paráfrases do que você disser no plano público (opcional)? Posso citar suas palavras (opcional)? Posso gravar o áudio (opcional)?"

**Contexto**
- Qual seu papel e há quanto tempo você desenvolve?
- Qual sua stack principal? Quais ferramentas de IA você usa, e com que frequência?

**Dificuldades** (antes de mencionar qualquer tema)
- O que é mais difícil para você em desenvolver com IA? (deixar responder livremente)
- Quais são as três mais importantes para você, em ordem?

**Última tarefa com IA**
- Me conta a última tarefa que você fez com uma ferramenta de IA, do começo ao fim.
- Quem escreveu a primeira versão do código, você ou a IA?
- Quando algo não funcionou, o que você fez em seguida? E depois?
- Como você decidiu se aceitava uma alteração? O que você olhou?
- Você fez outra coisa enquanto a IA estava trabalhando?
- Você conseguiria explicar agora para um colega o que mudou? Qual parte seria mais difícil?
- Como você preparou a revisão ou o PR?
- Para cada situação acima: o que isso te custou? (tempo, retrabalho, comentário na revisão, bug depois, não conseguir explicar) Quanto isso importa para você?

**Aprendizado e debugging**
- Pensa em algo que você aprendeu no último mês. Como você aprendeu?
- Me conta o último bug que você corrigiu. O que você fez primeiro?
- Quando você trava, a quem ou a quê você recorre primeiro? Quando foi a última vez que perguntou a uma pessoa?

**Reações a conceitos** (ler exatamente assim, em ordem alternada)
- *Digitação:* "Quando você está aprendendo algo, a IA explica e dá dicas, mas é você quem digita o código."
- *Check-ins:* "Depois de algumas ações, a IA para, resume o que fez e espera sua resposta."
- *Resumo das alterações:* "Antes do PR, um resumo curto e em linguagem simples do que mudou, por quê, e o que revisar."
- *Empurrão no loop:* "Se você reportar várias falhas seguidas, a IA pergunta qual você acha que é a causa antes de corrigir."
- Para cada um: Quando isso te ajudaria? Quando atrapalharia? Você desligaria?

**Nome**
- O que o nome "jrdev" te sugere?

**Encerramento**
- Tem algo que não perguntamos e deveríamos ter perguntado?
- Quem mais você indicaria para conversarmos?

**Mentores (adaptar):**
- Me conta o último PR de um júnior com código gerado por IA. O que você notou? O que você perguntou?
- Você já viu um júnior não conseguir explicar a própria alteração? O que aconteceu depois?
- Como os juniores pedem ajuda hoje, comparado com antes das ferramentas de IA?
- Quanto do seu tempo de revisão vai em código gerado por IA? O que torna isso lento?
- Você gostaria de ver algo sobre o aprendizado de um júnior? O que pareceria vigilância?
