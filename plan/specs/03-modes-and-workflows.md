# Modes and workflows: Typing, Learning, Debug, Map, Test

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 5.4. Review findings addressed here: P2-05, P3-02, P3-03, P3-05, P4-03, P4-05, P5-03; Map/Test: P-01, P-08. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the help-stage and task acceptance cases are part of the Phase 0 prototype. The review-item contract gates Phase 2 progress features. The pre-commit integration gates its distribution.

---


## Typing (edit policy `type`, Phase 1)
- Edits are denied per [commands & policy: state layers and the edit decision table](02-commands-and-policy.md#state-layers-and-the-edit-decision-table-p-05), and the deny reason tells the agent to coach. Bash write patterns are caught as a seatbelt.
- **`/jrdev:stuck` escalates:** hints, then pseudocode, then a partial snippet, then a worked solution plus a **one-call exception** (P2-05):
  - **Unit:** one *approved* `PreToolUse` decision for an `Edit`/`Write`/`MultiEdit` whose targets are all within the named file(s). A `MultiEdit` touching any other file is denied.
  - **Consumption:** the exception is checked and consumed in one critical section under an exclusive lock, and bound to that call's `tool_use_id`. Two simultaneous calls can't both use it.
  - **If the approved call later fails** (execution error, or another hook or permission rule denies it), the exception is **still spent**. That's documented, and the learner runs `/jrdev:stuck` again. We deliberately avoid reconciling through post-tool events, so there's no state that could grant a second use or leak an exception.
  - **Expiry:** 15 minutes of wall-clock time, checked when the decision is made, so process death or interruption can't leave an exception open indefinitely.
  - **Log:** `granted` → `used (tool_use_id)` or `expired`.
- **Every escalation step is logged as coached practice** ([assessment & grading spec](04-assessment-and-grading.md)).
- **Help stage is recorded by hooks, not by the model (P3-02):**
  - The session state holds `help_stage` ∈ {0 none, 1 hint, 2 pseudocode, 3 partial snippet, 4 worked solution}.
  - It advances **one step per user-typed `/jrdev:stuck`**, through the `UserPromptExpansion` channel.
  - **Task-scoped (P4-03):**
    - **Explicit task boundaries:** help stages and stuck exceptions are keyed by a `task_id`, created with a user-typed `/jrdev:task start <short name>` and closed with `/jrdev:task done`. Starting a new task closes the previous one.
    - **No active task:** `/jrdev:stuck` asks the learner to name the task first.
    - **Stage-4 authorization is single-use (P5-03).** Its lifecycle is defined by observable events:
      - **Created** by the user-typed `/jrdev:stuck` that reaches stage 4, through `UserPromptExpansion`. It's bound to `(session_id, task_id, grant_id)`.
      - **Covers exactly the turn started by that `/stuck` prompt.** The **next user prompt of any kind** (`UserPromptSubmit` or `UserPromptExpansion`) marks it `consumed`. That holds whatever happened in the turn: full delivery, partial delivery, interrupt or API error.
      - **Retry:** if the turn failed, the learner types `/jrdev:stuck` again. Stage 4 is terminal, so this creates a **new** grant, logged as a separate worked-solution grant.
      - **Resume:** at `SessionStart` with source `resume`, any grant whose turn has ended is marked consumed, and the authoritative state ("no active grant") is re-injected. Replayed conversation text saying stage 4 is allowed is overridden by that injection.
      - **Fork:** state is keyed by `session_id`, so **a forked session gets no grant**. `SessionStart` injects "no active grant", and a fresh `/stuck` is required.
      - **Compaction:** the state lives outside the conversation and is re-injected.
      - **Status and other commands** can't create or revive a grant. Only `/jrdev:stuck` can.
      - **Event ordering** (e.g. whether `UserPromptExpansion` and `UserPromptSubmit` both fire for one slash command) is **to be verified in the Phase 0 prototype**.
      - **This bookkeeping controls authorization state, not what the model says.** Whether a response stays within its stage remains **advisory**, and is checked by behavior review ([evaluation & studies: behavior-review oracle](07-evaluation-and-studies.md#behavior-review-oracle-p3-02)).
    - **Stages 1–3** persist until the task closes.
    - **Model task-change detection is advisory only.** The coach is instructed to ask "is this the same task?" when a request looks different, but only the user-typed command changes `task_id`.
    - **Resume and compaction:** the active `task_id` and stage live in hook-managed state, and `SessionStart` re-injects them after resume or compaction.
    - **Acceptance cases:**
      - reach stage 4 on task A, then run `/jrdev:task start B` (same file, then another file); B starts at stage 0. **Without the explicit command**, a question about B is only *advisorily* detected, and the promise is limited to the coach asking "same task?"
      - grant lifecycle: interrupt during partial and complete delivery, resume, compact, fork, and retry after an API error; no two solution turns ever share one grant
      - resume preserves the binding
      - compaction preserves the binding
  - Each stage's allowance is written in the coach instructions and used as the review oracle ([evaluation & studies: behavior-review oracle](07-evaluation-and-studies.md#behavior-review-oracle-p3-02)).

## Learning (`learning:on`; Phase 1 minimal, Phase 2 full)
- **Phase 1:** setup, session recap (pending until the learner confirms it), and injection of **topic identifiers only** ([data flows & privacy: what reaches the AI provider](05-data-flows-and-privacy.md#what-reaches-the-ai-provider)).
- **Phase 2:** FSRS reviews computed by a library in code, plus a local progress report.
- **Review-item contract (P3-03), defined before Phase 2:**
  - **Item fields:** `{item_id, objective_id, format (recall | explain | predict | debug-snippet), prompt_variants[], answer_criteria, version}`.
  - **One schedule per item.** Variants of an item must share its objective and format.
  - **Changes:** changing an item's objective or format creates a **new item with a new schedule**. Wording fixes bump `version` and keep the schedule.
  - **Ratings:**
    - The learner rates their own recall after seeing the answer criteria, as in classic spaced repetition.
    - jrdev **caps** the rating by assistance used: any hint caps it at `Hard`, and a shown answer forces `Again`.
    - The model can't set ratings.
  - **Storage:** every review stores the FSRS review log (rating, timestamps, assistance), so schedules can be replayed reproducibly.
  - **Scope:** reviews are **recall practice**. They never create "assessed transfer" evidence or raise demonstrated levels.

## Debug workflow (Phase 2)
- **Advisory steps:** symptom, reproduce, the learner's hypothesis, breakpoint (`# jrdev-bp` marker), predict-then-inspect, fix, regression test, remove breakpoints.
- **Who places breakpoints:** the learner if `edit:type`, otherwise the agent.
- **Enforcement:** the optional git pre-commit hook blocks commits that contain markers. The `Stop` hook gives one reminder.
- **Pre-commit details (P3-05):**
  - **Scope (P4-05):** the hook blocks **every active jrdev breakpoint in the files the commit touches**, not just newly added ones. Files the commit doesn't touch are out of scope, as declared.
  - **What counts as an active breakpoint** is decided from staged content alone, without local registration:
    - a line with a recognized breakpoint statement (`breakpoint()`, `pdb.set_trace()`, `debugger;`, `binding.irb`, `runtime.Breakpoint()`, …) **and** a `jrdev-bp:<id>` tag on the same line
    - marker text without a breakpoint statement (docs, test fixtures) **warns** only
  - **Optional `--strict` setting:** also blocks breakpoint statements that carry no jrdev tag.
  - **What it scans:** the **full staged blobs** (`git show :<path>`) of every path in `git diff --cached --name-only -z -M --diff-filter=ACMR`, including pure renames. Never the working tree.
    - **Verified locally:** a pure `git mv` produces zero added diff lines while the staged blob still contains the marker. That's why blobs are scanned, not diffs.
  - **Acceptance cases:**
    - a pure rename
    - an unchanged marker inside a modified file
    - a fresh clone with no session history
    - deleted local registration
    - a docs example
  - **Installation never overwrites existing hooks:**
    - It detects `core.hooksPath`, the `pre-commit` framework, husky and lefthook, and provides a config snippet for the one in use.
    - With a plain `.git/hooks/pre-commit` already present, it chains into it only with consent, after backing it up.
    - Uninstalling restores the backup.
  - **Documented limit:** `--no-verify` bypasses the hook.
  - **Acceptance cases:**
    - a staged marker with a clean working file
    - a clean index with an unstaged marker
    - renames
    - paths with spaces
    - an existing hook
    - a custom `core.hooksPath`
    - uninstall
- **Interactivity:** v1 is learner-driven in their own terminal. v2 would be a DAP-based MCP server (not verified yet).

## Map workflow (Phase 2–3) and Test workflow (Phase 3)
Unchanged from the previous revision:
- **Map:** static analysis with source-labelled edges and explicit "unresolved" markers. It's described as static structure, not runtime flow.
- **Test:**
  - disposable local fixtures only
  - pre-flight checks are **warnings**, and the residual risk is stated
  - checklist items are `done` / `skipped` / `n/a`, never auto-passed
  - coverage is shown as "executed lines/branches in this run"; ordered flow only comes from request-scoped tracing

Their acceptance checks also carry over:
- a fixture that looks local/dev but reaches production-like endpoints or inherits credentials triggers warnings, and undetected paths are documented
- same coverage with a different order can't be told apart without tracing
- dynamic dispatch shows as unresolved edges

Pending items, recaps and the `Stop` hook are covered in the [commands & policy spec](02-commands-and-policy.md#pending-items-recaps-and-stop-p-04).
