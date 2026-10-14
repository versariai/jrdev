# Related tools

*Notes on products that target the same problem as jrdev. Checked 2026-10-14 from public pages only; features are as each site describes them, not tested.*

## A desktop code explainer

**What it is:** a desktop app (Mac, Windows) that explains AI-generated code. Select code in Cursor, VS Code, JetBrains, a terminal, Claude Code or Codex, press a shortcut, and an explanation opens beside the editor. Free beta; the site lists a cap of 30 explanations a month on the free tier.

**Features:**
- Five explanation depths (New to Expert), "explain differently", and follow-up questions
- "I understand" and "Test me" buttons; a short quiz, and a cross-lesson quiz that revisits weak items first
- A saved library with "Understood" and "Review" states; saved explanations are **flagged as stale when the code changes**
- **A review of agent edits:** ranks the git diff an agent left, sensitive files first, then the largest edits, with a review checklist
- On-device secret scan of the selection before anything is sent to the cloud model
- Progress: lines understood, streaks, a heat map

**How it differs from jrdev:**
- Explanation after generation. jrdev's principle 1 has the learner do the thinking; the research it builds on suggests explanation alone may not build skill (Bastani 2025, see [ai-for-junior-devs.md](ai-for-junior-devs.md)).
- Its quiz result and "I understand" are presented as understanding. jrdev records quiz answers as coached practice only (principle 4).
- Streaks and "lines understood" are activity metrics, which jrdev doesn't use as goals.
- It never says "junior": it addresses anyone shipping AI code. Relevant to the open brand question in the plan.
- It's a separate paid app with its own model calls. jrdev runs inside the user's Claude Code, MIT-licensed, with no account or credits.
- Its resources page cites no studies.

**Ideas taken or worth considering** (none is scheduled; each goes through [candidate promotion](../plan/specs/03-modes-and-workflows.md#candidate-promotion-p13-05)):

| Idea | Status |
|---|---|
| Riskiest changes first in the change summary | Added to the [C6 skill](../experiments/c6-explain-changes/skills/jrdev-explain-changes/SKILL.md) (advisory) |
| Ask juniors whether they use an explainer tool | Added to the [interview guide](../plan/phase0/interview-guide.md), background section |
| Review items linked to code, retired or flagged when that code changes | Idea for the Phase 2 Learning item contract |
| `/jrdev:explain <file:lines>` with depth from the learner profile, ending in a question in learning mode | Idea; would be a new candidate |
| A hook that scans `Read` and `git diff` output for secret patterns before it reaches the provider | Idea; would be enforced, unlike C6's path filtering. Needs the hook performance budget check |
| An explain or change-summary pane | Question for the [mods spike](../plan/specs/02-commands-and-policy.md#mods-spike-phase-0) |
