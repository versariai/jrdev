---
name: jrdev-explain-changes
description: Explain the current changes in plain language before review or a PR. Covers staged, unstaged, untracked and branch-committed work, kept separate, plus the decisions made, rejected alternatives, trade-offs and what a reviewer should check. Use when the user asks what changed, wants a summary before committing or opening a PR, or says they don't understand a change.
argument-hint: "[base-ref] [--learn]"
---

<!-- jrdev-experiment-c6: installed by the jrdev C6 experiment. Safe to delete with this folder. -->

# Explain changes

Produce a short, plain-language explanation of **all** current changes so a developer can review them before committing or opening a PR. This skill is **read-only**: never edit, stage, commit or run tests unless the user asks separately.

**What reaches the model:** every diff or file you read becomes part of this conversation and is sent to the AI provider. So list paths first, settle the scope, and only then read content for paths that pass the exclusions in step 2.

Arguments: `$ARGUMENTS`
- An optional base ref (e.g. `main`, `origin/develop`, `HEAD~3`) overrides base detection.
- `--learn` adds one question at the end (see section 5).

## 1. Find the committed range

Run these read-only git commands yourself.

**Start with a health check:** `git rev-parse --git-dir`. If it fails, stop and show the error: either this isn't a git repository, or Git can't read its configuration.

**Failures (P17-02).** Only these commands may fail as a normal answer:
- **base and upstream probes** in this section (`rev-parse --abbrev-ref origin/HEAD`, `rev-parse --verify --quiet <ref>`, `rev-parse --verify --quiet @{u}`): a nonzero exit means "that ref doesn't exist". Move to the next fallback described below
- **the filter check** in step 2: follow its exit-code rules there

**Any other git command that fails** (`log`, `diff`, `diff-files`, `ls-files`, `merge-base`, `check-attr`, `rev-parse HEAD`) means the inspection failed: stop and show the error. Never treat empty output from a failed command as "no changes".

**No configured helpers (P15-01).** Some Git commands can run helper programs from the user's Git config, which would break "read-only". So:
- run **every** git command in this skill as `git -c core.fsmonitor=false …`
- add `--no-ext-diff --no-textconv` to **every** `git diff`, including `--stat` and `--name-status`
- don't run `git status`; the layer lists in step 2 replace it

**Paths are literal (P17-01).** Whenever a command takes file paths, pass an explicit list of allowed files, each as its own argument, and add Git's global `--literal-pathspecs` option. Otherwise Git reads names like `[a].py` or `*.py` as patterns that can match other files. Never use `:(exclude)` or other pathspec patterns; leave excluded files out of the list instead. If the list is empty, skip the command: with nothing after `--`, Git compares every file.

The commands below are written short; apply all three rules to each one. For example, the unstaged read in full is `git -c core.fsmonitor=false --literal-pathspecs diff --no-ext-diff --no-textconv -- <path1> <path2>`.

1. **Base.** Use the base ref from the arguments if one was given. Otherwise try `git rev-parse --abbrev-ref origin/HEAD`, and if that fails, the first of `origin/main`, `origin/master`, `main` or `master` that exists (`git rev-parse --verify --quiet <ref>`). If no base can be found, ask the user for one rather than guessing.
2. **Is the current branch the base?** Compare `git rev-parse --abbrev-ref HEAD` with the base name, after stripping a leading `origin/` (so `main` and `origin/main` both count as the base when you're on `main`). Then pick **exactly one** case and use its `RANGE` everywhere below:
   - **On the base branch, with an upstream** (`git rev-parse --verify --quiet @{u}` succeeds): `RANGE=@{u}..HEAD`, the commits not yet pushed.
   - **On the base branch, no upstream:** there's no committed range. Skip the committed steps, cover only the uncommitted work, and say so in "Scope covered".
   - **On a different branch:** `M=$(git merge-base HEAD <base>)`, then `RANGE=$M..HEAD`.

## 2. List paths, choose the scope, then read

**Keep four layers separate** all the way through: **committed** (the range), **staged** (what the next `git commit` would include), **unstaged** (only in the working files), and **untracked**. Never merge staged and unstaged into one diff: an unstaged edit can undo a staged one, and a combined diff would then show nothing even though the next commit changes the file.

1. **List paths and statistics only.** No content yet:
   - committed: `git log --oneline RANGE`, `git diff --name-only RANGE` and `git diff --stat RANGE`
   - staged: `git diff --cached --name-status` and `git diff --cached --stat`
   - unstaged paths: `git diff-files --name-only`. This doesn't run filters, but it can list files whose timestamp changed with no real edit; their diff will simply be empty
   - untracked: `git ls-files --others --exclude-standard`
2. **Unstaged statistics, without running filters (e.g. Git LFS).** Comparing working files runs any **clean filter** configured for them, even for statistics. So first decide which unstaged paths are **safe to compare**, then count lines for those only. Always do this, whether or not filters are configured:
   1. **Is any filter configured?** Run `git config --get-regexp '^filter\..*\.(clean|process)$'` and check its **exit code**:
      - **1** (no match): no filter is configured. Every unstaged path is safe.
      - **0** (match): run `git check-attr filter -- <unstaged paths>`. A path whose filter value is anything other than `unspecified` or `unset` is **not safe**. List it under "Not read: uses a configured filter (<name>)". Its staged and committed diffs are still fine, because those compare stored versions and don't run the filter.
      - **Any other code** (an error): you can't tell which paths are filtered. Treat **no** unstaged path as safe, say so, and ask the user before comparing any of them.
   2. **Count lines for the safe paths only:** `git diff --stat -- <safe unstaged paths>`, as a literal list. **If there are no safe paths, don't run this command:** with an empty list after `--`, it would compare every file, filtered ones included.
   3. **Unknown sizes count as large.** If some unstaged paths couldn't be counted (filtered, or the filter check failed), say their size is unknown, and treat that as "large" in substep 4.
   - If the user explicitly asks to include filtered files, run the normal command for them and say in "Scope covered" that their filter ran.
3. **Never read the content of these paths.** List them under "Not read" without printing anything from them:
   - environment and secret files: `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `id_ed25519*`, `.npmrc`, `.pypirc`, `.netrc`
   - any path containing `secret`, `credential`, `token` or `password` (case-insensitive)
   - binaries, lockfiles and generated files: list them and describe them from `--stat` only
4. **Choose the scope before any content read.** Using only the lists and statistics from substeps 1 and 2, ask the user which parts to cover, and **wait for the answer**, if either applies:
   - **large:** roughly 2,000 or more changed lines across all layers, or any layer whose size is unknown (substep 2)
   - **possibly sensitive:** configuration files that aren't excluded by name but may hold credentials, such as `settings.*`, `config.*`, `*.yaml`, `*.yml`, `*.toml`, `*.ini`, `*.conf`, `*.properties`, or anything under a `config/` directory

   Paths the user declines are added to the exclusions. Otherwise, continue with everything not excluded.
5. **Read each layer separately,** each with its own literal list of allowed files: the layer's paths from substep 1, minus the "never read" paths from substep 3 and any the user declined. Skip a layer whose list is empty:
   - committed: `git diff RANGE -- <allowed committed paths>`
   - staged: `git diff --cached -- <allowed staged paths>`
   - unstaged: `git diff -- <allowed unstaged paths>`, starting from the safe paths in substep 2
6. **Untracked files aren't read automatically.** Read one only if it's clearly source code or docs that belong to this change, and it's within the chosen scope. If in doubt, list it and ask.
7. **Limits:** these rules are instructions you follow, not a technical privacy boundary.
   - Path filtering isn't secret redaction: a secret in an ordinary source file will still be read. If you notice one, don't repeat it; tell the user to remove it.
   - Disabling fsmonitor, external diff drivers, text conversion and clean filters covers the Git helpers known to run here. It doesn't sandbox Git.
   - Without text conversion, some formats (e.g. documents or notebooks with a converter) show as raw or binary diffs. Describe those from statistics.

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
| File | Layer (committed / staged / unstaged / untracked) | What changed | Why (recorded / inferred) |

### What the next commit would include
- The staged changes only. If unstaged changes alter or undo them, say so plainly, file by file.

### Decisions and alternatives
- Decision → alternatives considered → why this one (recorded / inferred)

### Trade-offs and risks
- What could break, what got more complex, what was deliberately left out

### What a reviewer should check
- 3–6 concrete things to look at (specific functions, edge cases, data changes)

### How to test it
- Steps or commands to verify the behavior

### Scope covered
- Base: <ref> · Range: <RANGE or "none (no upstream)"> · Commits: <n> · Staged files: <n> · Unstaged files: <n> · Untracked files: <n>
- Not read: <excluded secret-like, binary, generated or lock files, paths the user declined, and filtered files in the unstaged layer; paths only>
```

**Be honest about gaps:**
- **Too big to explain in full:** say which parts you summarized only at a high level.
- **Unknown changes:** if some changes weren't made in this conversation, such as edits by another person or another session, say so and label their reasons as inferred.

## 5. With `--learn`

End with **one** question that checks understanding of the most important change, for example "Which input would now take the new branch in `parse_order`?" or "Why does this need a transaction?".
- **Don't answer it.** Wait for the user's reply.
- **Then give feedback on the reply:** say what was right and what was missing.
