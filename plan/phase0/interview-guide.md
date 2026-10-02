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
- [ ] **Consent form** with separate scopes: (a) use for product research, (b) anonymized paraphrases in internal docs, (c) verbatim quotes (optional), (d) audio recording (optional).
- [ ] **Storage:** notes and any recordings live in **one private location** owned by Ivan, outside the repo. The retention and deletion steps are written down and tested with a fake participant.
- [ ] **Transcription:** if a transcription service is used, it's disclosed in the consent form along with its retention. Otherwise, notes only.
- [ ] **[Employer-pilot safeguards](../jrdev-ai-plan.md#employer-pilot-safeguards) apply to company juniors:**
  - participation is voluntary
  - managers aren't told who took part
  - nothing is used in performance reviews
- [ ] **Pseudonyms:** each participant gets a code (`J01`…, `M01`…). The name ↔ code list is kept separately.

## Participants and format

- **Who:** **10 juniors** (0–2 years of experience, backend, at the company; external juniors are fine if the company pool is small) and **5 mentors** (people who review juniors' code).
- **Format:** 30–40 minutes, remote, one interviewer. Recording only with consent (scope d).
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
| 1 | Intro and consent | 3 min | Purpose, voluntary, no performance use, what we record. Confirm the consent scopes |
| 2 | Background | 3 min | Role, time as a developer, main stack, which AI tools, how often |
| 3 | **Last task walkthrough** | 12 min | "Tell me about the last task you did with an AI tool, from start to finish." Probes:<br>• Who wrote the first version, you or the AI?<br>• **When something didn't work, what did you do next? And after that?** *(T1, autopilot loop)*<br>• **How did you decide whether to accept a change? What did you look at?** *(T2, approval)*<br>• Did you do anything else while the AI was working? *(T3, focus)*<br>• Could you explain what changed to a colleague right now? Which part would be hardest? *(T4, understanding)*<br>• How did you prepare the review or PR? *(T4, summary)* |
| 4 | Learning and debugging | 8 min | • "Think of something you learned in the last month. How did you learn it?" *(T5)*<br>• "Tell me about the last bug you fixed. What did you do first?" *(T6)*<br>• "When you're stuck, who or what do you go to first? When did you last ask a person?" *(T7, silent silo)* |
| 5 | **Difficulties, unprompted then ranked** | 3 min | "What's hardest for you about developing with AI?" (let them answer freely) → "Of those, which matters most?" |
| 6 | Concept reactions | 5 min | Read 3–4 one-line, neutral descriptions (below). For each: "When would this help you? When would it get in the way? Would you turn it off?" |
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
Code: J07 · Date · Role/experience · Stack · AI tools · Consent scopes: a b c d
Walkthrough (paraphrased): …
Specific instances by theme: T1 … T2 … T3 … (one line each, mark "instance" or "opinion")
Unprompted difficulties, in their order: 1 … 2 … 3 …
Concept reactions: Typing … C2 … C6 … C1 … (helps when / hurts when / would turn off?)
Name reaction: …
Quotes (only with scope c): …
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
- **Phase 0 gate (juniors):** count the juniors who described a specific instance of **T1, T2 or T5**, *or* put one of them in their unprompted top 3:

  | Juniors (out of 10) | Decision |
  |---|---|
  | ≥ 5 | **Proceed** |
  | 3–4 | **Revise** (refocus the product on the themes that did recur) |
  | ≤ 2 | **Stop or pivot** |

  Scale proportionally if fewer than 10 are interviewed, and report the actual n.
- **Candidate priority:** a candidate (C1–C6, Typing) moves into the Phase 1 scope if **≥ 3 juniors** describe the problem it targets **and** fewer than half say they'd turn it off. Mentor input can raise or lower a candidate's priority, but can't add one on its own.
- **Name:** if **≥ 3** of the 15 participants find "jrdev" patronizing or off-putting, shortlist alternatives before buying the domain.
- **Output:** a paraphrased synthesis (theme counts, unprompted top difficulties, concept reactions) goes into the plan's early-signals section, replacing the n = 2 signal. No raw notes go into the repo.

---

## Appendix: Roteiro em português

**Abertura:** "Obrigado por participar. É voluntário, você pode parar a qualquer momento, e nada disso é usado em avaliação de desempenho, e seu gestor não fica sabendo quem participou. Posso tomar notas? Posso gravar o áudio (opcional)?"

**Contexto**
- Qual seu papel e há quanto tempo você desenvolve?
- Qual sua stack principal? Quais ferramentas de IA você usa, e com que frequência?

**Última tarefa com IA**
- Me conta a última tarefa que você fez com uma ferramenta de IA, do começo ao fim.
- Quem escreveu a primeira versão do código, você ou a IA?
- Quando algo não funcionou, o que você fez em seguida? E depois?
- Como você decidiu se aceitava uma alteração? O que você olhou?
- Você fez outra coisa enquanto a IA estava trabalhando?
- Você conseguiria explicar agora para um colega o que mudou? Qual parte seria mais difícil?
- Como você preparou a revisão ou o PR?

**Aprendizado e debugging**
- Pensa em algo que você aprendeu no último mês. Como você aprendeu?
- Me conta o último bug que você corrigiu. O que você fez primeiro?
- Quando você trava, a quem ou a quê você recorre primeiro? Quando foi a última vez que perguntou a uma pessoa?

**Dificuldades**
- O que é mais difícil para você em desenvolver com IA? (deixar responder livremente)
- Dessas, qual é a mais importante?

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
