---
name: explain-changes
description: Explain the current changes in plain language before review or a PR. Covers uncommitted, staged, untracked and branch-committed work, plus the decisions made, rejected alternatives, trade-offs and what a reviewer should check. Use when the user asks what changed, wants a summary before committing or opening a PR, or says they don't understand a change.
argument-hint: "[base-ref] [--learn]"
---

# Explain changes

Produce a short, plain-language explanation of **all** current changes so a developer can review them before committing or opening a PR. This skill is **read-only**: never edit, stage, commit or run tests unless the user asks separately.

Arguments: `$ARGUMENTS`
- An optional base ref (e.g. `main`, `origin/develop`, `HEAD~3`) overrides base detection.
- `--learn` adds one question at the end (see step 5).

## 1. Find what to cover

Run these read-only git commands yourself. Stop and say so if this isn't a git repository.

1. **Base.** Use the base ref from the arguments if one was given. Otherwise try `git rev-parse --abbrev-ref origin/HEAD`, and if that fails, the first of `origin/main`, `origin/master`, `main` or `master` that exists (`git rev-parse --verify --quiet <ref>`).
   - If the current branch **is** the base, the committed scope is the commits ahead of its upstream (`git log --oneline @{u}..HEAD`). If there's no upstream, there are no committed changes; cover only the uncommitted work and say so.
   - If no base can be found, ask the user for one rather than guessing.
2. **Committed on this branch:** `M=$(git merge-base HEAD <base>)`, then `git log --oneline $M..HEAD` and `git diff $M HEAD`.
3. **Uncommitted** (staged and unstaged together): `git diff HEAD`.
4. **Untracked files:** `git ls-files --others --exclude-standard`, then read the relevant ones.
5. **Skip, but list:** binary files, lockfiles and generated files. Read only the parts needed to explain them.

## 2. Find the reasons

- **Recorded:** a reason that appears in **this conversation**, such as a decision you made or the user asked for, an error you fixed, or an option you rejected.
- **Inferred:** a reason you're only guessing from the code. **Label it "inferred"** and don't present it as fact.
- **Never invent a rationale.** If you can't tell why something was done, say "reason unknown, worth asking".

## 3. Write the explanation

Use this structure. Keep it under about 400 words unless the change is large. Avoid jargon, and explain any term a junior developer might not know.

```
### Summary
2–3 sentences: what this change does and why, in plain language.

### What changed
| File | What changed | Why (recorded / inferred) |

### Decisions and alternatives
- Decision → alternatives considered → why this one (recorded / inferred)

### Trade-offs and risks
- What could break, what got more complex, what was deliberately left out

### What a reviewer should check
- 3–6 concrete things to look at (specific functions, edge cases, data changes)

### How to test it
- Steps or commands to verify the behavior

### Scope covered
- Base: <ref> · Commits: <n> · Uncommitted files: <n> · Untracked files: <n>
- Skipped: <binary/generated/lock files, if any>
```

## 4. Be honest about gaps

- **Too big to explain in full:** if the diff is too large, say which parts you summarized only at a high level.
- **Unknown changes:** if some changes weren't made in this conversation, such as edits by another person or another session, say so and label their reasons as inferred.

## 5. With `--learn`

End with **one** question that checks understanding of the most important change, for example "Which input would now take the new branch in `parse_order`?" or "Why does this need a transaction?".
- **Don't answer it.** Wait for the user's reply.
- **Then give feedback on the reply:** say what was right and what was missing.
