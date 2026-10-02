# Experiment C6: readable change summary

*An informal experiment, drafted 2026-10-02. Tests [candidate C6](../../plan/specs/03-modes-and-workflows.md#feature-candidates-from-early-signals) from the plan's [early signals](../../plan/jrdev-ai-plan.md#early-signals-2026-10-02): both early respondents (2 of 2) found it hard to see **what the AI changed and why** before a review or PR.*

**Status:** advisory only. Nothing here blocks or enforces anything; it's instructions for Claude Code. This is **not** pilot data.

**Participant rollout is on hold until the [readiness checklist](#readiness-before-inviting-anyone-p13-01) passes.** Until then, only team-only artifact testing is allowed: Ivan runs the skill on his own or synthetic repositories. If the experiment was already shared with anyone, pause feedback collection until the checklist passes; they can keep or uninstall the skill.

## What's included

| File | What it does |
|---|---|
| [`skills/jrdev-explain-changes/SKILL.md`](skills/jrdev-explain-changes/SKILL.md) | A `/jrdev-explain-changes` skill: a plain-language summary of committed, staged, unstaged and untracked changes, with decisions, trade-offs, what to review, and an optional learning question |
| [`CLAUDE-snippet.md`](CLAUDE-snippet.md) | A short, marked CLAUDE.md section asking Claude to note non-obvious decisions as it works, and to summarize before suggesting a commit or PR |

You can use either part on its own. Trying both is the most informative.

## Readiness before inviting anyone (P13-01)

Calling this experiment informal doesn't make it exempt: it's a real collection channel, registered in [per-channel readiness](../../plan/specs/01-website-and-feedback.md#per-channel-readiness-p6-03) and the [data-flow inventory](../../plan/specs/05-data-flows-and-privacy.md#data-flow-inventory). Before inviting or enrolling anyone:

- [ ] **Employer safeguards:** the [employer-pilot safeguards](../../plan/jrdev-ai-plan.md#employer-pilot-safeguards) are confirmed privately in writing, including no performance use. Participation is voluntary, and whoever invites people is outside their reporting line.
- [ ] **AI policy:** the company's AI policy allows running Claude Code on the work repositories where the skill will be used. If it doesn't, use only personal or open-source repositories.
- [ ] **Scoped consent,** recorded per person: (a) using their feedback for product research, (b) paraphrased themes in the public plan, (c) verbatim quotes (optional, off by default).
- [ ] **Feedback storage:** one private location owned by Ivan, outside the repo. Ivan is the only recipient. The chat or mail message is copied there and then deleted from the messaging app.
- [ ] **Retention:** raw feedback is kept until the C6 promotion decision plus 6 months, then deleted.
- [ ] **Withdrawal and deletion:** on request, delete the message, the stored copy, and any derived note that identifies the person. Paraphrases already published stay only if they can't identify anyone.
- [ ] **Dry run:** a synthetic feedback message is taken through storage, a paraphrased output, withdrawal and deletion, and nothing is left behind.

**Early feedback already collected (2026-10-02):** its raw text stays private and isn't processed further or published until both respondents confirm scope (b), and nothing that would identify a specific workplace incident is published. The disposition is recorded privately, without identities.

## What reaches the AI provider

The skill is read-only on disk, but **every diff and file it reads is sent to your AI provider** as part of the conversation. To limit that:
- it first lists only file names and change statistics, and never reads `.env`-style files, keys or paths that look like credentials (see the skill's step 2)
- on very large changes, or configuration files that might hold credentials, it asks which parts to cover **before reading any content**, and leaves out what you decline
- it doesn't read untracked files automatically

**Limits:** these are instructions Claude follows, not a guaranteed technical boundary. That includes switching off configured Git helpers (see "How to use it"). Path filtering isn't secret redaction: a credential inside an ordinary source file would still be read. Use the skill only where your company's AI policy allows it.

## Install (about 2 minutes)

Read both files before installing. They're short, and they contain no scripts.

1. **Skill.** This copies it only if nothing with the same name exists:
   ```bash
   dest="$HOME/.claude/skills/jrdev-explain-changes"
   if [ -e "$dest" ]; then
     echo "Stop: $dest already exists. Move it aside first."
   else
     mkdir -p "$HOME/.claude/skills" &&
     cp -R experiments/c6-explain-changes/skills/jrdev-explain-changes "$dest"
   fi
   ```
   Start a new Claude Code session. It's invoked as `/jrdev-explain-changes`. In the future jrdev plugin it will be `/jrdev:explain-changes`.
2. **Snippet** (optional). Back up the file first, then paste the marked block from `CLAUDE-snippet.md`, markers included:
   ```bash
   [ -f CLAUDE.md ] && cp CLAUDE.md CLAUDE.md.pre-jrdev-c6
   ```
   Do the same for `~/.claude/CLAUDE.md` if you paste it there.

## Uninstall

1. **Skill.** This removes the folder only if it's the experiment's copy:
   ```bash
   dest="$HOME/.claude/skills/jrdev-explain-changes"
   if grep -q 'jrdev-experiment-c6' "$dest/SKILL.md" 2>/dev/null; then
     rm -rf "$dest"
   else
     echo "Not the experiment's copy; left untouched."
   fi
   ```
2. **Snippet.** This removes only the text between the markers. Then compare with your backup:
   ```bash
   perl -0pi -e 's/<!-- BEGIN jrdev-experiment-c6 -->.*?<!-- END jrdev-experiment-c6 -->\n?//s' CLAUDE.md
   diff CLAUDE.md.pre-jrdev-c6 CLAUDE.md && rm CLAUDE.md.pre-jrdev-c6
   ```
   If `diff` shows differences, you edited the file in the meantime. Check them by hand before deleting the backup.

## How to use it

- **Before a commit or PR:** `/jrdev-explain-changes`. The base branch is detected automatically.
- **Against a specific base:** `/jrdev-explain-changes origin/develop`.
- **To check your own understanding:** `/jrdev-explain-changes --learn`. It ends with one question for you to answer.

It runs only `git log`, `git diff`, `git diff-files`, `git rev-parse`, `git merge-base`, `git ls-files`, `git config --get-regexp` and `git check-attr`, and reads files. Staged and unstaged changes are reported separately, so you can see exactly what the next commit would include.

**Configured Git helpers are switched off (P15-01).** Ordinary `git diff` can run helper programs from your Git config: fsmonitor, external diff drivers, text-conversion filters, and clean filters such as Git LFS. The skill disables the first three on every command. It skips the unstaged comparison for files with a clean filter and lists them as not read, unless you ask to include them. This covers the helpers known to run during these commands; it doesn't sandbox Git. Without text conversion, some formats show as raw or binary diffs. Claude Code may ask permission for those git commands, depending on your settings.

## Duration and feedback

- **Duration:** about **1–2 weeks** of normal work, once the readiness checklist has passed.
- **Feedback:** send it to Ivan by message. **Don't paste code, diffs or company details.** Short answers are fine.

### Perguntas de feedback (PT-BR)
1. Quantas vezes você usou `/jrdev-explain-changes` (ou o resumo do snippet) antes de um commit/PR?
2. O resumo te ajudou a revisar antes do PR? (1 = nada, 5 = muito) Por quê?
3. Você entendeu alguma alteração que antes teria aprovado sem entender? Dê um exemplo genérico, sem código.
4. Algum motivo marcado como "recorded" estava errado, ou algum "inferred" era inventado?
5. O que faltou ou sobrou no resumo? (tamanho, formato, linguagem)
6. Se usou `--learn`: a pergunta foi útil ou atrapalhou?
7. Continuaria usando? O que mudaria?

## How results are used
- **In the plan:** answers are paraphrased into the plan's early-signals section, as n = 2 informal feedback, only within each person's consent scope. No raw text goes into the repo.
- **Deciding on C6:** results are **formative input** to the single [candidate promotion decision](../../plan/specs/03-modes-and-workflows.md#candidate-promotion-p13-05). Two favorable users don't authorize shipping C6; it goes through the same interview evidence, design review and capacity check as every candidate.
- **Measures:** the descriptive measures for C6 are in [specs/07](../../plan/specs/07-evaluation-and-studies.md#measures).
