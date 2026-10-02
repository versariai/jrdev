# Modes and workflows: Typing, Learning, Debug, Map, Test

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 5.4. Review findings addressed here: P2-05, P3-02, P3-03, P3-05, P4-03, P4-05, P5-03, P6-05, P13-05, P13-06, P14-03, P15-01; Map/Test: P-01, P-08. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

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
    - **After a stage-4 grant is consumed (P6-05):** the task stays at stage 4, but with **no valid grant** the coach may only explain the solution already shown and give stage-3-level help. Another complete solution needs a new `/jrdev:stuck`. The behavior-review oracle judges responses by **stage plus grant validity**, per [evaluation & studies](07-evaluation-and-studies.md#behavior-review-oracle-p3-02).
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

## Feature candidates from early signals
These come from the [early signals](../jrdev-ai-plan.md#early-signals-2026-10-02) (n = 2). They're **candidates**, not commitments: each needs Phase 0 interview support and a design review before it's scheduled. Each one is labelled enforced or advisory.

| ID | Candidate | Problem it targets | Mechanism | Label | Earliest phase |
|---|---|---|---|---|---|
| **C1** | **Loop nudge** | The autopilot fix loop | A `UserPromptSubmit` hook counts consecutive prompts **within a task** that report failure (pasted errors, "doesn't work") with no stated hypothesis. After N (default 2), it adds context telling the coach to ask "what do you think is causing it?" and to wait for an answer before proposing a fix | **Advisory.** The hook can't force the model; behavior review checks compliance | 1 |
| **C2** | **Check-ins** | Losing focus and passive approval | A fourth, independent setting `checkin: off \| N`. A `PreToolUse` hook **denies** further `Edit`/`Write`/`MultiEdit`/`NotebookEdit`/`Bash` calls after N tool calls with no learner message, and tells the agent to summarize progress and wait. The next learner prompt resets the counter | **Enforced** for those tool paths; other tools are a documented gap | 1 |
| **C3** | **Predict before approve** | Rubber-stamp review in normal work (where the AI is allowed to edit) | Before applying a non-trivial change, the coach states what it will change and asks one prediction question (e.g. "which tests should now fail?"). It applies the change after the learner answers. It could later become a third edit policy, `review`, if the pilot shows value | **Advisory** at first | 2 |
| **C4** | **Frontend comprehension view** | Not understanding UI changes | An extension of Map: component tree, props/state flow, and before/after screenshots of the affected UI | Advisory | 3+ (the pilot juniors are backend) |
| **C5** | **No parallel agents while learning** | Focus, and machine limits | With `learning:on`, the coach is told not to start background agents. Documented guidance discourages parallel runs for learners | Advisory | 1 |
| **C6** | **Readable change summary**, `/jrdev:explain-changes` | Opaque decisions, hard pre-PR review, not understanding changes (**2 of 2**) | A plain-language summary of the **branch's commits plus staged, unstaged and untracked changes, kept separate** (avoiding the committed-only gap from [skills report R5-01](../../research/agent-skills-for-junior-devs.md)). It reads Git with configured helpers switched off (fsmonitor, external diff, text conversion; clean-filtered files skipped in the unstaged layer; P15-01). It covers what changed, the **decisions taken and alternatives rejected**, the trade-offs, and what a reviewer should check. In learning mode it ends with one "why" or "predict" question. Also shipped as a **CLAUDE.md snippet** in the Starter pack asking for a decision log and a pre-PR summary | Advisory (a user-invoked skill) | 1 |

### Candidate promotion (P13-05)
**One decision moves a candidate into a phase.** The interview eligibility rule, the C6 experiment and design reviews are **inputs** to it; none of them schedules a candidate on its own.

| Step | Rule |
|---|---|
| 1. **Eligibility** | From the [interview rule](../phase0/interview-guide.md#synthesis-and-decision-rules-set-before-the-first-interview). Concepts that weren't presented (C3, C4, C5) stay **unevaluated** until they're tested, in later interviews or the pilot. The C6 experiment is **formative only**: it can shape C6's design, but it doesn't make C6 eligible |
| 2. **Earliest phase** | A hard floor. An eligible candidate is never scheduled before its "Earliest phase" column (C3 not before Phase 2, C4 not before Phase 3) |
| 3. **Design acceptance** | Before scheduling: add it to the [interface registry](02-commands-and-policy.md#interface-registry-p6-04) with its phase; add its rows to the [data-flow inventory](05-data-flows-and-privacy.md#data-flow-inventory); for C1 and C3, extend the behavior-review oracle with compliance scenarios; for C2, meet the [composition requirements](#c2-composition-requirements-p13-06) below. A plan review round checks it |
| 4. **Capacity** | Phase 1 adds **at most two** candidates to its core scope (Typing, `/jrdev:stuck`, minimal Learning, manual assessments), given 10 hours a week. If more qualify, prefer the one with the most "important problem" juniors, then the lowest build cost |

- **Decision owner:** Ivan, recorded in the plan's [decisions table](../jrdev-ai-plan.md#7-decisions) with the evidence for each step.
- **Acceptance:** with a synthetic result where **every** candidate qualifies, the procedure still yields a documented selection of at most two that respects every earliest phase. Two favorable C6 experiment users alone produce no scheduling change.

### C2 composition requirements (P13-06)
C2 is the only candidate that **enforces** something, so before it's adopted:
- **A fourth setting everywhere.** The [three settings](02-commands-and-policy.md#three-independent-settings-p2-01) contract becomes four:
  - `/jrdev:off` also sets `checkin: off`
  - `/jrdev:status` shows the effective check-in state and the current counter
  - the Phase 0-style acceptance cases are repeated with four settings
- **Configuration:** set only through the trusted `UserPromptExpansion` channel (`/jrdev:checkin off|N`). The default is `off`. It can be set per session, or as a user default; a project default can't force it on.
- **Counter:** stored in hook-managed session state and updated under the same lock as exceptions. It resets on the next user prompt (`UserPromptSubmit` or `UserPromptExpansion`). Concurrent tool calls increment it atomically, so it's never double-counted.
- **One tool set (P14-03):** `Edit`, `Write`, `MultiEdit`, `NotebookEdit` and `Bash`. The candidate table, the [interface registry](02-commands-and-policy.md#interface-registry-p6-04) matcher and the tests all use this set. Other tools (reads, MCP tools, subagents) are a documented gap.
- **File edits:** a new row **4a** in the [edit decision table](02-commands-and-policy.md#state-layers-and-the-edit-decision-table-p-05), after rows 1–4 (kill switch, control files, recovery override, parse failure) and **before every ordinary Allow** (row 5, open policy; row 6, allowlist; row 7, exception). Check-ins are independent of the edit policy, so they also pause open-policy and allowlisted edits.
  - Row 4a: `checkin` is N, and the counter has reached N → **Deny** with the check-in message.
  - **The full decision is computed before any exception is consumed,** so a check-in denial never spends a stuck exception.
- **Bash:** Bash isn't in the edit table. Its own order is: kill switch → control-path and CLI protection (always deny) → recovery override → **check-in** → the Typing write-pattern seatbelt.
- **Recovery:** check-ins never block `/jrdev:off`, `/jrdev:type off`, `jrdev off` or the kill switch. Those are user-typed commands or terminal CLI calls, not model tool calls.
- **No continuation loop:** repeated denials within one turn return the same message, and they don't trigger `Stop` reminders.
- **Acceptance:** at the threshold, each of these follows the declared combined decision:
  - an open-policy `Edit`, an allowlisted `Write`, a `MultiEdit`, a `NotebookEdit` and a `Bash` call: all denied by check-in
  - a Typing-denied edit: denied, with the check-in message taking precedence
  - an exception-covered edit: denied, and the exception stays unspent
  - a control-file edit: denied by row 2 as before
  - with a recovery override or the kill switch: check-ins don't apply

  Also:
  - Reach the threshold, run `/jrdev:off`, and a new burst of tool calls is unrestricted by C2.
  - `/jrdev:status` matches the effective state.
  - Parallel tool calls at the threshold produce exactly one transition to "denied", with no continuation loop.

