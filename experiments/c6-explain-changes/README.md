# Experiment C6: readable change summary

*An informal experiment, started 2026-10-02. Tests [candidate C6](../../plan/specs/03-modes-and-workflows.md#feature-candidates-from-early-signals) from the plan's [early signals](../../plan/jrdev-ai-plan.md#early-signals-2026-10-02): both early respondents (2 of 2) found it hard to see **what the AI changed and why** before a review or PR.*

**Status:** advisory only. Nothing here blocks or enforces anything; it's instructions for Claude Code. This is **not** pilot data. It's a quick check with the two developers who gave the early feedback and agreed to take part.

## What's included

| File | What it does |
|---|---|
| [`skills/explain-changes/SKILL.md`](skills/explain-changes/SKILL.md) | A `/explain-changes` skill: a plain-language summary of committed, uncommitted and untracked changes, with decisions, trade-offs, what to review, and an optional learning question |
| [`CLAUDE-snippet.md`](CLAUDE-snippet.md) | A short CLAUDE.md section asking Claude to note non-obvious decisions as it works, and to summarize before suggesting a commit or PR |

You can use either part on its own. Trying both is the most informative.

## Install (about 2 minutes)

1. **Skill:** copy the skill folder into your personal skills directory:
   ```bash
   mkdir -p ~/.claude/skills && cp -r experiments/c6-explain-changes/skills/explain-changes ~/.claude/skills/
   ```
   Start a new Claude Code session. It's invoked as `/explain-changes`. In the future jrdev plugin it will be `/jrdev:explain-changes`.
2. **Snippet** (optional): paste the section from `CLAUDE-snippet.md` into your project's `CLAUDE.md`, or into `~/.claude/CLAUDE.md`.

Read both files before installing. They're short, and they contain no scripts.

**To uninstall:** `rm -rf ~/.claude/skills/explain-changes` and remove the snippet.

## How to use it

- **Before a commit or PR:** `/explain-changes`. The base branch is detected automatically.
- **Against a specific base:** `/explain-changes origin/develop`.
- **To check your own understanding:** `/explain-changes --learn`. It ends with one question for you to answer.

The skill is read-only: it runs `git log` / `git diff` / `git ls-files` and reads files. Claude Code may ask permission for those git commands, depending on your settings.

## Duration and feedback

- **Duration:** about **1–2 weeks** of normal work.
- **Feedback:** send it to Ivan by message. **Don't paste code, diffs or company details.** Short answers are fine.

### Perguntas de feedback (PT-BR)
1. Quantas vezes você usou `/explain-changes` (ou o resumo do snippet) antes de um commit/PR?
2. O resumo te ajudou a revisar antes do PR? (1 = nada, 5 = muito) Por quê?
3. Você entendeu alguma alteração que antes teria aprovado sem entender? Dê um exemplo genérico, sem código.
4. Algum motivo marcado como "recorded" estava errado, ou algum "inferred" era inventado?
5. O que faltou ou sobrou no resumo? (tamanho, formato, linguagem)
6. Se usou `--learn`: a pergunta foi útil ou atrapalhou?
7. Continuaria usando? O que mudaria?

## How results are used
- **In the plan:** answers are paraphrased into the plan's early-signals section, as n = 2 informal feedback. No raw text goes into the repo.
- **Deciding on C6:** if it helps, C6 moves into the Phase 1 Starter pack. If not, it's revised or dropped.
- **Measures:** the descriptive measures for C6 are in [specs/07](../../plan/specs/07-evaluation-and-studies.md#measures).
