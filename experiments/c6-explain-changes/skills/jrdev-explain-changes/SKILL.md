---
name: jrdev-explain-changes
description: Explain the current changes in plain language before review or a PR. Covers uncommitted, staged, untracked and branch-committed work, plus the decisions made, rejected alternatives, trade-offs and what a reviewer should check. Use when the user asks what changed, wants a summary before committing or opening a PR, or says they don't understand a change.
argument-hint: "[base-ref] [--learn]"
---

<!-- jrdev-experiment-c6: installed by the jrdev C6 experiment. Safe to delete with this folder. -->

# Explain changes

Produce a short, plain-language explanation of **all** current changes so a developer can review them before committing or opening a PR. This skill is **read-only**: never edit, stage, commit or run tests unless the user asks separately.

**What reaches the model:** every diff or file you read becomes part of this conversation and is sent to the AI provider. So list paths first, and read content only for paths that pass the exclusions in step 2.

Arguments: `$ARGUMENTS`
- An optional base ref (e.g. `main`, `origin/develop`, `HEAD~3`) overrides base detection.
- `--learn` adds one question at the end (see section 5).

## 1. Find the committed range

Run these read-only git commands yourself. Stop and say so if this isn't a git repository.

1. **Base.** Use the base ref from the arguments if one was given. Otherwise try `git rev-parse --abbrev-ref origin/HEAD`, and if that fails, the first of `origin/main`, `origin/master`, `main` or `master` that exists (`git rev-parse --verify --quiet <ref>`). If no base can be found, ask the user for one rather than guessing.
2. **Is the current branch the base?** Compare `git rev-parse --abbrev-ref HEAD` with the base name, after stripping a leading `origin/` (so `main` and `origin/main` both count as the base when you're on `main`). Then pick **exactly one** case and use its `RANGE` everywhere below:
   - **On the base branch, with an upstream** (`git rev-parse --verify --quiet @{u}` succeeds): `RANGE=@{u}..HEAD`, the commits not yet pushed.
   - **On the base branch, no upstream:** there's no committed range. Skip the committed steps, cover only the uncommitted work, and say so in "Scope covered".
   - **On a different branch:** `M=$(git merge-base HEAD <base>)`, then `RANGE=$M..HEAD`.

## 2. List paths, then decide what to read

1. **List before reading:**
   - committed: `git log --oneline RANGE` and `git diff --stat RANGE`
   - uncommitted (staged and unstaged together): `git diff --stat HEAD`
   - untracked: `git ls-files --others --exclude-standard`
2. **Never read the content of these paths.** List them under "Not read" without printing anything from them:
   - environment and secret files: `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `id_ed25519*`, `.npmrc`, `.pypirc`, `.netrc`
   - any path containing `secret`, `credential`, `token` or `password` (case-insensitive)
   - binaries, lockfiles and generated files: list them and describe them from `--stat` only
3. **Read the diffs** with those paths excluded, e.g. `git diff RANGE -- . ':(exclude).env*' ':(exclude)*.pem'` (add an exclude for each matching path), and the same for `git diff HEAD`.
4. **Untracked files aren't read automatically.** Read one only if it's clearly source code or docs that belong to this change. If in doubt, list it and ask.
5. **Large or sensitive changes:** if the diff is very large (roughly 2,000 changed lines or more) or touches configuration that may hold credentials, show the path list first and ask which parts to cover.
6. **Limits:** path filtering isn't secret redaction. A secret in an ordinary source file will still be read. If you notice one, don't repeat it; tell the user to remove it.

## 3. Find the reasons

- **Recorded:** a reason that appears in **this conversation**, such as a decision you made or the user asked for, an error you fixed, or an option you rejected.
- **Inferred:** a reason you're only guessing from the code. **Label it "inferred"** and don't present it as fact.
- **Never invent a rationale.** If you can't tell why something was done, say "reason unknown, worth asking".

## 4. Write the explanation

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
- Base: <ref> · Range: <RANGE or "none (no upstream)"> · Commits: <n> · Uncommitted files: <n> · Untracked files: <n>
- Not read: <excluded secret-like, binary, generated or lock files, paths only>
```

**Be honest about gaps:**
- **Too big to explain in full:** say which parts you summarized only at a high level.
- **Unknown changes:** if some changes weren't made in this conversation, such as edits by another person or another session, say so and label their reasons as inferred.

## 5. With `--learn`

End with **one** question that checks understanding of the most important change, for example "Which input would now take the new branch in `parse_order`?" or "Why does this need a transaction?".
- **Don't answer it.** Wait for the user's reply.
- **Then give feedback on the reply:** say what was right and what was missing.
