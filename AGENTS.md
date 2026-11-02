# Agent Instructions

> **Note:** `CLAUDE.md` is a symlink to this file. Editing `AGENTS.md` updates both.

An optional, gitignored `AGENTS.local.md` at the repo root holds per-developer
overrides layered on top of this file. It is imported below and is simply absent
for anyone who has not created one.

@AGENTS.local.md

## Repository Overview

`jrdev` is the planning, research and site repo for jrdev.ai. It is public.

- `site/` — the jrdev.ai homepage and privacy page: static Astro (`astro` ^7), npm,
  deployed on Cloudflare Pages (root `site`, build `npm run build`, output `dist`).
  Site values live in `site/src/config.ts`; consent copy is versioned in
  `site/consent/vN.md` (see [`site/README.md`](site/README.md)).
- `plan/` — product and launch plans (`plan/phase0/`, `plan/c6-explain-changes/`).
- `research/` — research notes and adversarial reviews.
- `newsletter/`, `experiments/` — content drafts and experiments.

There is no CI config, test suite or linter in this repo; `npm run build` is the
only automated check.

## Hard Rules (Non-Negotiable)

These apply to **every** agent session. If a task appears to require breaking
one, stop and ask the human first.

### Never push to `main`

- **Do not `git push` to `main` under any circumstances**, including `--force`
  and pushes from a detached HEAD that resolves to `main`.
- **Do not create a git worktree whose upstream tracks `main`.** Base new
  worktrees on a feature branch, e.g.
  `git worktree add ../wt-foo -b feat/foo origin/main`.
- All changes land on `main` through a reviewed pull request.
- Force pushing a feature branch is fine; prefer `--force-with-lease`.

### Never leave a process you started running

Every process an agent starts is that agent's to stop. In this repo that means
chiefly the Astro dev/preview servers (`npm run dev` and `npm run preview` in
`site/`, default port `4321`) and any one-off `node` script or browser session.

- Plan the shutdown when you plan the startup; never start something you have no
  handle on.
- Prefer one-shot commands (`npm run build`) over daemons.
- Reuse what is already up: check `lsof -ti :4321` before starting a second copy.
- Before you report done, verify nothing you started is alive
  (`lsof -ti :4321`, `pgrep -fl astro`) and kill stragglers.
- Exception: if the user asks you to leave a server up, say what is running, on
  which port, and how to stop it.

## Code Style

- **No unnecessary comments.** Let clear names carry the meaning. Comment only to
  explain _why_ something non-obvious is done, in one terse line.
- **Never commit secrets** or real credentials. The values in `site/src/config.ts`
  are public by design; anything private stays out of the repo.
- **This repo is public.** Use role names rather than people's names, keep raw
  feedback and private correspondence out, and don't name competitors.
- **Check for similar code after adding a helper.** `rg` for existing code that
  does the same thing and reuse it rather than leaving duplicates.

## Testing Policy

- Verify site changes by running `npm run build` in `site/` and, for visible
  changes, checking the page in `npm run dev`. Stop the server afterwards.
- If tests are ever added, write them before the code they cover, prefer
  end-to-end checks of real behavior, and use unique fixtures per test.
  Do not write trivial tests (render-without-crash, constants, mocks returning
  what they were told to).

## Commit and PR Guidance

- Keep commits atomic, focused, and descriptive.
- Before pushing a `site/` change, run `npm run build` in `site/` and let it pass.
- Include lockfile changes (`site/package-lock.json`) with dependency changes.
- **Never credit an agent as a commit co-author.** No `Co-Authored-By:` trailer
  for Claude or any other AI tool, even when a harness default would add one.
- **Never attribute a PR to an agent.** No "Generated with …" lines, badges, or
  trailers in the title, body, or comments.
- **Never commit screenshots or PR media.** Save them outside the git tree and
  share the path so the human can attach them to the PR.
- Direct pushes to `main` are forbidden — see _Hard Rules_.

## AI Assistant Guidance

- **Change only what the task requires, and finish it.** Match neighboring files'
  patterns; leave unrelated cleanups for their own PR.
- Validate assumptions against this repo's own files (`site/package.json`,
  `site/README.md`, `plan/`) rather than other repositories' conventions.

### Subagent Model Selection

Right-size the model when spawning subagents instead of inheriting the parent's.

- **Haiku** — simple, mechanical, narrow tasks: file lookups, grep/find, running
  a command and reporting output, mechanical edits with clear instructions.
- **Sonnet** — moderate tasks needing synthesis but not deep judgment:
  multi-file edits with a settled design, bounded diff review, drafting a PR
  description.
- **Opus** — real thinking only: architecture decisions, non-trivial planning,
  debugging with an unclear cause, security review across trust boundaries.

Before every subagent call, ask "what's the smallest model that can do this?"
When in doubt between two tiers, pick the lower one.

## Preferred Commands

Run from `site/` (`npm install` first on a fresh checkout):

- `npm run dev` — dev server at http://localhost:4321 (stop it when done)
- `npm run build` — static build into `site/dist/`; the pre-push check
- `npm run preview` — serve the built output (stop it when done)
