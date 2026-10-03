# Plugin commands, state and edit policy

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 5.1–5.3, 5.6, 5.7 and part of 5.4. Review findings addressed here: P-04, P-05, P-10, P2-01, P2-03, P2-04, P3-01, P4-04, P5-02, P6-04, P13-06, P13-08, P14-03. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the Phase 0 prototype must pass the acceptance cases for the three settings, the command channel and the recovery paths. Data-compatibility tests gate the first public **update**.

---

## Three independent settings (P2-01)
**What you're working on and whether the agent may edit files are separate settings.** Changing one never silently changes the other.

| Setting | Values | What it controls |
|---|---|---|
| **Edit policy** | `open` · `type` | Whether the agent's file edits are **denied** (the only *enforced* setting) |
| **Workflow** | `none` · `debug` · `map` · `test` | Which guided workflow the agent follows (advisory) |
| **Learning** | `on` · `off` | Profile features: goal injection, recap, reviews (advisory) |

**Composition rules:**
- Workflows never change the edit policy. Debug + `type` means the learner inserts breakpoints themselves; Debug + `open` means the agent may insert them.
- `/jrdev:off` sets **all three** to off/none/open, and says explicitly: "Edit enforcement is now OFF."
- **Candidate C2 would add a fourth setting** (`checkin`). If it's adopted, every rule here that says "three" covers four, as listed in [C2 composition requirements](03-modes-and-workflows.md#c2-composition-requirements-p13-06) (P13-06).
- **Any** change to the edit policy is announced in the conversation and shown in the status line.
- The study treatment "Learning + Typing" = `learning:on` + `edit:type` with any workflow. Each session's configuration is logged, so the study can verify which treatment a participant actually received.

**Acceptance:**
- Start with `edit:type`, switch the workflow to `debug`, then turn learning on. The next Edit must still be denied, matching the status display.
- A study session's log shows `learning:on, edit:type`.

## Command contract and the trusted channel for state changes (P-10, P2-03)
Plugin skills are invoked as `/jrdev:<skill>`.

| Command | Effect | Invocable by |
|---|---|---|
| `/jrdev:setup` | First-run interview (with a preview of what will be injected; [data flows & privacy: what reaches the AI provider](05-data-flows-and-privacy.md#what-reaches-the-ai-provider)) | User |
| `/jrdev:type on\|off` | Sets the edit policy for this session | User only |
| `/jrdev:workflow <none\|debug\|map\|test>` | Sets the workflow | User only |
| `/jrdev:learn on\|off` | Turns profile features on or off | User only |
| `/jrdev:off` | Turns everything off for this session, with an explicit notice | User only |
| `/jrdev:task start <name>` / `/jrdev:task done` | Opens or closes the task that help stages and exceptions are bound to ([modes & workflows spec](03-modes-and-workflows.md)) | User only |
| `/jrdev:stuck` | Escalates help within the active task; the last step grants a one-call exception ([modes & workflows spec](03-modes-and-workflows.md)) | User only |
| `/jrdev:status [--context]` | Shows the three settings, enforced vs. advisory rules, pending items, and with `--context` the exact text injected into the session | User or model |
| `/jrdev:feedback` | Opens the feedback form | User or model |

**Trusted channel (design; the Phase 0 prototype must confirm it):**
- **State changes happen only in a `UserPromptExpansion` hook.** That hook fires when the *user types* a slash command. It receives `session_id`, `command_name` and `command_args`, and it can block the command or add context. It **doesn't fire when the model invokes a skill**, which goes through the `Skill` tool and `PreToolUse` instead.
- The hook handler validates the arguments and writes the session state atomically ([commands & policy: state layers and the edit decision table](#state-layers-and-the-edit-decision-table-p-05)). A state change is therefore a hook decision, not a shell command that permission rules could reject or that the model could imitate.
- **Model-invoked attempts are denied** by `PreToolUse` on the `Skill` tool for the user-only commands. `disable-model-invocation: true` is set as well, as a second layer.
- **Writes to protected files:**
  - **Paths are normalized** (realpath, case on macOS) before matching. Any model `Edit`/`Write`/`MultiEdit` targeting jrdev control files is denied.
  - The control files are session, config, and exception state.
  - **Control-file protection takes precedence over every allowlist.** jrdev scaffold files are written by hook code, never through model edit tools, so they need no allowlist entry.
- **Model `Bash` calls:** denied when they reference the jrdev control paths or the CLI's state-changing subcommands. **This is a seatbelt.** Indirect writes, such as through a script the model creates, aren't fully preventable. This limit is documented.
- **Recovery CLI:** `jrdev off` asks for confirmation typed at the terminal (`/dev/tty`). The agent's non-interactive Bash can't answer that prompt.
- **What we promise:** enforcement holds on the **tested tool paths** (the acceptance cases in this section and [commands & policy: state layers and the edit decision table](#state-layers-and-the-edit-decision-table-p-05)). We don't claim the model can never switch enforcement off.

**Acceptance:**
- A user-typed command changes state exactly once.
- Model attempts are denied and state stays unchanged: Skill tool calls, Edit/Write to control files, and Bash invoking the CLI.
- Test paths with spaces, symlinks or aliases to control files, and conflicts with the allowlist.
- If the hook fails, the command reports failure and state is unchanged.

## State layers and the edit decision table (P-05)
**Layers**, highest precedence first:
1. **Recovery override:** written by `jrdev off` or the kill switch. When present, lower layers aren't parsed at all.
2. **Session state:** private, outside the repo ([data flows & privacy: where data lives](05-data-flows-and-privacy.md#where-data-lives)), keyed by `session_id`.
3. **Project defaults:** `.jrdev/config.json` in the repo. This is shareable and may be committed. It contains **no** learner data.
4. **User defaults:** `~/.jrdev/config.json`.

Each file has a schema version, is written via a temporary file plus rename under a lock, and is validated on read.

**Edit decision** for `Edit|Write|MultiEdit|NotebookEdit`:

| # | Condition (evaluated in order) | Decision |
|---|---|---|
| 1 | Kill switch file `~/.jrdev/DISABLED` exists | Allow, with a "jrdev disabled" notice |
| 2 | Target is a jrdev control file | **Deny** (always) |
| 3 | A recovery override exists for this session | Allow |
| 4 | Session, project or user state can't be parsed | **Deny**, with recovery steps in the message |
| 5 | Effective edit policy ≠ `type` | Allow |
| 6 | Path is on the hard allowlist (lockfiles, generated dirs) | Allow |
| 7 | A valid one-call stuck exception covers this session + file ([modes & workflows spec](03-modes-and-workflows.md)) | Allow and consume the exception |
| 8 | Otherwise | **Deny**, with coaching instructions |
| — | Hook runtime is missing or crashes | Claude Code treats the failure as non-blocking, so the edit **is allowed**. This is a documented limit, and `/jrdev:status` reports the problem |

- **"Smart" topic-based triggering is advisory** until its classifier and failure behavior are specified and tested.

**Recovery (P2-04):**

| Situation | Steps | Scope |
|---|---|---|
| Typing is too strict right now | `/jrdev:type off` | This session |
| Project default forces `type` | `/jrdev:type off` writes a session override above the project default | This session; the project file is untouched |
| Malformed session, project or user file | `jrdev off --session <id>` (or `--latest`) writes a recovery override, so row 3 applies before row 4 | That session only. Other sessions' state and **pending records are preserved** |
| Runtime (Node/Python) missing | Edits already fall through as allowed (last row). `jrdev off` is a **POSIX `sh` script** with no runtime dependency | — |
| Everything is broken | `touch ~/.jrdev/DISABLED`. Each hook call is a new process, so running sessions see it immediately | **All sessions**, as documented |

`JRDEV_DISABLE=1` only affects sessions **started** with it; that's documented as a restart-based option. **Deleting directories is never presented as session-local recovery.**

**Leaving recovery (P5-02):**
- **The recovery override is tied to one `session_id`.** New sessions are never affected by it.
- **Re-enabling a session:** `/jrdev:type on` in that session **first re-validates every lower layer**.
  - If they all parse, it **removes that session's recovery override** and sets `type`.
  - If a layer is still malformed, it **refuses** and names the broken file. It never reports `type` as active while edits stay allowed.
- **The kill switch is global, so a session command never removes it.**
  - `/jrdev:type on` with `~/.jrdev/DISABLED` present answers: "Enforcement is disabled globally by the kill switch (created at T). Run `jrdev on --global` in a terminal to re-enable it for all sessions."
  - `jrdev on --global` asks for confirmation typed at the terminal (`/dev/tty`).
- **`/jrdev:status` shows the *effective* decision and why**, e.g. "edits allowed: recovery override (this session)", "edits allowed: global kill switch" or "edits denied: type (session)". It never shows just the requested setting.
- **Acceptance cases:**
  - recover, repair the config, then `/jrdev:type on` while a second session is active; the next Edit matches the status display
  - the same with the kill switch present: the refusal message appears, and `jrdev on --global` restores enforcement

**Acceptance:**
- recover from a project default of `type`
- recover from a malformed project config
- recover from a malformed user config
- recover with the runtime missing

In each case a second session stays active, its policy and pending records are unchanged, and the target session can work normally afterwards.

## Pending items, recaps and `Stop` (P-04)
- Human confirmations are **pending states**. The `Stop` hook adds one reminder when `stop_hook_active` is false and then yields.
- `Stop` doesn't run when the user interrupts, and Claude Code ends the turn after 8 consecutive continuations. Pending items are re-surfaced at the next `SessionStart`.
- Skipped items are never counted as passed.

## Data compatibility across versions (P3-01)
**Control state vs. learner records:**
- **Control state** (session, config, exceptions) carries a `policy_version` and a list of `required_capabilities` (P4-04). Fields are split into two groups:
  - **Informational** (in an `info` block): safe for older readers to ignore.
  - **Policy-affecting:** each one is tied to a named capability.
- **An older engine meeting an unknown required capability doesn't silently apply its older, more permissive rule.** It applies a **safe degraded policy**: edits in the affected scope are treated as `type`, and an explicit notice says an update is needed for full policy support.
  - **Recovery** (`/jrdev:type off`, `jrdev off`, kill switch) keeps working regardless of capability.
- **`/jrdev:status` shows** the engine version, its capabilities, and the policy version in effect. Concurrent versions report their own effective policy.
- **We don't claim every future policy is backward-compatible.**
- **Acceptance:** an old reader meeting a synthetic new restrictive field applies the degraded policy and reports it. Purely informational fields stay harmless.
- **Learner records** (profile, logs, assessments, review history) have explicit schema versions.

**Compatibility rules:**
- Each release **reads** its own schema and the previous one, and **writes only** its own.
- An older version that finds a newer learner-record schema switches to **read-only profile mode**. Enforcement keeps working from control state, and the user is told to update or restore.

**Migrations:**
- They run only through `jrdev migrate`, offered at session start and never applied silently.
- They take an automatic **backup snapshot** first (`~/.jrdev/backups/<timestamp>/`) and run under the global lock, so a second version can't write at the same time. An interrupted migration resumes or rolls back from the snapshot.
- **Evidence categories, scores and provenance are immutable.** A migration may restructure fields but can never upgrade `coached` to `assessed transfer`.

**Rollback:**
- Install the older SHA, then either keep using compatible data or restore the pre-migration snapshot with `jrdev restore <timestamp>`.
- **`jrdev reset` is never the documented fix.**

**Acceptance:**
- run A → B → A with populated goals, pending recaps, assessments and review history
- interrupt a migration
- run two versions concurrently

In every case, records are preserved, recovery works, and evidence categories are unchanged.

## Interface registry (P6-04)
**This table is the single source of truth** for every command and hook the plugin registers. The layout below is derived from it.

- **Clean-install check:** enumerates this registry and invokes every row active in the current phase.
- **Review rule:** spec reviews compare each consumer spec against it, so a missing entry fails review.
- **Minimum client versions:** recorded per row during the Phase 0 prototype ("TBD" until then).

### Commands
| Command | Kind | Implemented in | Consumers | Phase |
|---|---|---|---|---|
| `/jrdev:setup` | Skill (user) | `skills/setup` + `UserPromptExpansion` handler | specs 02, 05 | 1 |
| `/jrdev:type on\|off` | Skill (user only) | `skills/type` + expansion handler | specs 02, 03 | 0 (prototype), 1 |
| `/jrdev:workflow …` | Skill (user only) | `skills/workflow` + expansion handler | specs 02, 03 | 2 |
| `/jrdev:learn on\|off` | Skill (user only) | `skills/learn` + expansion handler | specs 02, 03 | 1 |
| `/jrdev:off` | Skill (user only) | `skills/off` + expansion handler | spec 02 | 0, 1 |
| `/jrdev:task start\|done` | Skill (user only) | `skills/task` + expansion handler | spec 03 (help stages, grants), spec 07 (oracle fixtures) | 0, 1 |
| `/jrdev:stuck` | Skill (user only) | `skills/stuck` + expansion handler | specs 03, 07 | 0, 1 |
| `/jrdev:status [--context]` | Skill (user or model) | `skills/status` | specs 02, 05 | 0, 1 |
| `/jrdev:feedback` | Skill (user or model) | `skills/feedback` | spec 01 | 1 |
| `/jrdev:debug`, `/jrdev:map`, `/jrdev:test` | Skills (workflow guides) | `skills/{debug,map,test}` | spec 03 | 2–3 |
| `/jrdev:explain-changes` (candidate C6) | Skill (user or model) | `skills/explain-changes` | spec 03 | candidate |
| `/jrdev:checkin off\|N` (candidate C2) | Skill (user only) + expansion handler | `skills/checkin` | specs 02, 03 | candidate |
| `jrdev off [--session\|--latest]`, `jrdev on --global` | CLI (POSIX sh, TTY-confirmed) | `bin/jrdev` | spec 02 | 0, 1 |
| `jrdev assess start\|submit [--manual]` | CLI | `bin/jrdev` | specs 04, 05 | 1 (manual), 3 (service) |
| `jrdev migrate`, `jrdev restore`, `jrdev reset`, `jrdev projects`, `jrdev verify-info` | CLI | `bin/jrdev` | specs 02, 05, 06 | 1+ |

### Hook events
| Event | Matcher | Handler responsibility | Consumers | Phase |
|---|---|---|---|---|
| `SessionStart` | — | Inject minimal context, re-inject task, stage and grant state, surface pending items, health check | specs 02, 03, 05 | 0, 1 |
| `UserPromptExpansion` | `jrdev:*` commands | **Trusted state changes**, grant creation | specs 02, 03 | 0, 1 |
| `UserPromptSubmit` | — | Mark stage-4 grants consumed on the next prompt | spec 03 | 0, 1 |
| `PreToolUse` | `Edit\|Write\|MultiEdit\|NotebookEdit` | Edit decision table, one-call exceptions | specs 02, 03 | 0, 1 |
| `PreToolUse` | `Bash` | Write-pattern seatbelt, control-path protection | spec 02 | 0, 1 |
| `PreToolUse` | `Skill` | Deny model invocation of user-only commands | spec 02 | 0, 1 |
| `Stop` | — | One reminder for pending items | specs 02, 03 | 1 |
| `PostModelSwitch` | — | Log model changes for configuration records | spec 07 | 1 (logging), 3 (study) |
| `DirectoryAdded` | — | Record the added root; **don't** inject its private context; recommend a fresh session | spec 05 | 1 |
| `UserPromptSubmit` (candidate C1) | — | Count failure reports with no hypothesis per task; add a loop-nudge context | spec 03 | candidate |
| `PreToolUse` (candidate C2) | `Edit\|Write\|MultiEdit\|NotebookEdit\|Bash` | Check-in denial after N tool calls with no learner message (row 4a of the edit table; its own step in the Bash order) | spec 03 | candidate |

## Hook performance budget
Signal (n = 1): learners can be **resource-constrained** (16 GB of RAM limiting parallel work). jrdev must not add noticeable load.
- **Latency:** each hook invocation takes **≤ 100 ms at p95**, and `SessionStart` ≤ 300 ms, on the reference machine.
- **Aggregate:** all enabled hooks for one tool call together take **≤ 150 ms at p95**.
- **Memory:** **≤ 50 MB** peak resident memory per invocation.
- **No background processes,** and **no network calls** from hooks.

**Benchmark contract (P13-08):**
- **Reference machine:** the oldest supported Node LTS on a 16 GB laptop. The exact model and OS version are recorded in the repo during the Phase 0 prototype, and re-recorded if they change.
- **What's timed:** the **whole invocation**, from process spawn to exit. That includes Node startup, module loading, state reads under the lock, and writing the JSON output. Handler logic alone isn't measured.
- **Samples:** for each handler, 20 cold runs (first run after a fresh install) and 200 warm runs. p95 is reported separately for each.
- **Memory:** peak resident memory of the hook process, measured with `/usr/bin/time` (`-l` on macOS, `-v` on Linux).
- **Contention fixture:** two sessions firing hooks at the same time against shared state, to include lock waits.
- **Inputs:** recorded hook inputs from the test suite, with every candidate hook that's enabled in the current phase turned on.
- **CI checks:**
  - **Absolute:** the budgets above, scaled by a CI-to-reference factor that's measured and recorded in the repo.
  - **Relative:** a regression of more than 20% against the last approved baseline fails the build, but only if it reproduces on a rerun. The baseline changes only through a reviewed commit.
- **Acceptance:** an artificial 150 ms startup delay, and an artificial 60 MB allocation, are both detected on the reference machine and in CI.

## Mods spike (Phase 0)
**Background.** On 2026-10-01 Anthropic launched **mods** for Claude Code 2.1.287 and later ([announcement](https://claude.com/blog/claude-code-mods), [getting-started guide](https://claude.dev/blog/getting-started-with-claude-code-mods/)). A mod is a TypeScript module inside a plugin:
- **How it runs:** it's loaded once per session and runs in its own sandbox, with no Node. It reaches files, processes and the UI only through Claude Code's `$` API.
- **What it hooks:** events such as `tool.call`, `prompt.submit`, `command.run`, `turn.start`/`turn.complete`, `session.start` and `ui.render`. A hook can watch an event, rewrite it, or answer it itself, for example by denying a tool call.
- **What else it can do:** keep session state, register slash commands and tools, draw panes, buttons and status lines, and redact tool output before the model reads it.
- **Limits:** the API "can change between releases". Mods run in load order, and each hook gets 10 seconds of its own time per event.

Mods could replace the per-event hook processes in this spec. They would remove the Node runtime dependency and most of the [performance budget](#hook-performance-budget), and give the candidates real UI. **This spec stays hooks-based until the spike below says otherwise.**

**Scope:** time-boxed to about **8 hours** of Phase 0 plugin time. Build the [edit decision table](#state-layers-and-the-edit-decision-table-p-05) and the command channel as a mod next to the hooks prototype, using `claude plugin validate` and `claude plugin test`, and answer these questions with tests:

| # | Question | Test | If it fails |
|---|---|---|---|
| M1 | **Trusted channel:** does `command.run` for a jrdev command fire **only** when the user types it? | Model attempts through the `Skill` tool, a model-callable tool and prompt text never change state | State changes stay in the `UserPromptExpansion` hook |
| M2 | **Failure behavior:** when the mod throws, hits the 10-second limit, or fails to load, is the tool call allowed or blocked? | A fixture mod for each case; observe the next `Edit` | The decision table's last row documents the observed behavior, as it does today for hooks |
| M3 | **Load order:** can a mod that loads earlier answer `tool.call` without calling `next`, so jrdev never sees it? What changes when `sec-default` loads first (Team and Enterprise plans, or managed settings)? | A fixture mod that answers `Edit` itself; the same suite with managed settings on | The bypass is a documented gap in "What we promise", with a `/jrdev:status` warning if it's detectable |
| M4 | **Recovery and state:** can control state stay in files under `~/.jrdev/`, so `jrdev off`, the kill switch and the [recovery cases](#state-layers-and-the-edit-decision-table-p-05) keep working without Claude Code? | Run every recovery acceptance case against the mod | Control state stays with the hooks version |
| M5 | **Version floor:** what happens on a client older than 2.1.287? Do the pilot's machines meet it? | Install on an older client; survey pilot machines | Hooks remain the default, and the mod becomes optional |
| M6 | **Release trust:** do `defaultEnabled: false`, SHA pinning and [verify-before-enable](06-repo-and-release-trust.md#release-trust-from-signing-to-verified-installation-p-09) work for a plugin that contains a mod? | The clean-install check on a mod build | No mod ships until they do |

The [benchmark contract](#hook-performance-budget) also runs on the mod, so the two versions can be compared.

**Decision rule** (recorded as decision 10 in the [overview](../jrdev-ai-plan.md#7-decisions) at the end of Phase 0):
- **Enforcement** (edit decision table, trusted channel, recovery) moves to a mod only if M1–M4 and M6 pass.
- **UI-only features** (the C2 check-in pane, C3's held edit, a C6 change view) can use mods if M2, M5 and M6 pass, even if enforcement stays in hooks.
- **A hybrid is allowed:** hooks enforce, and mods draw.
- **Redacting tool output** is a possible later upgrade to C6's content rules and [specs/05](05-data-flows-and-privacy.md#what-reaches-the-ai-provider). It's evaluated separately, and isn't claimed until tested.

## Technical layout
*Derived from the [interface registry](#interface-registry-p6-04). If the two disagree, the registry wins.*
```
plugins/jrdev/
  .claude-plugin/plugin.json
  hooks/hooks.json             # SessionStart, UserPromptSubmit, UserPromptExpansion, PreToolUse(Edit|Write|MultiEdit|NotebookEdit|Bash|Skill), Stop, PostModelSwitch, DirectoryAdded
  hooks/handlers/              # policy engine (edit decision table), state I/O with locking, exception reservations
  skills/{setup,type,workflow,learn,off,task,stuck,status,feedback,debug,map,test}/SKILL.md
  bin/jrdev                    # POSIX sh: off, on --global, status, assess, migrate, restore, reset, projects, verify-info (no runtime dependency for off/on)
  output-styles/jrdev-coach.md
  lib/{profile,policy,analysis}/
  vendor/                      # bundled deps, lockfile-pinned
  tests/                       # decision-table units, recorded hook inputs, concurrency tests, clean-install command tests
```
