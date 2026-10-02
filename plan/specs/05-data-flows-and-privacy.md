# Data flows and privacy

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 6. Review findings addressed here: P-03, P2-06, P2-07, P4-06, P6-01, P7-04, P8-01, P8-02, P9-03. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** data-flow and privacy acceptance must pass before the Phase 1 pilot build is distributed.

---

## Where data lives
**Private learner records never live in the repository worktree.**

| Location | Contents | Shareable? |
|---|---|---|
| `~/.jrdev/global/` | Topic identifiers from a **controlled vocabulary** (e.g. `react-hooks`, `sql-joins`), self-rated levels, review queue (topic IDs), assessment summaries with a **fixed schema** (see [assessment & grading spec](04-assessment-and-grading.md): `skill_id`, task and rubric versions, category, scorer, date, score band, evidence ref) | No |
| `~/.jrdev/projects/<project-id>/` | Session state, logs, debug notes, maps, exceptions, project-scoped goals and free-text mission | No |
| `~/.jrdev/assessments/` | Assessment attempts and the exposure log | No (scores are exported to a mentor only on explicit request) |
| `<repo>/.jrdev/config.json` | **Team defaults only**: default policy, allowlists, workflow settings. No learner data | Yes, may be committed intentionally |

- Before writing anything inside the repo (only `config.json`, and only on explicit request), jrdev checks the file's tracking and ignore status and tells the user. **It never untracks files automatically.**
- **If a repo already contains tracked `.jrdev/` logs** from older versions or manual copies, jrdev warns and explains the options (move them out, `git rm --cached`). The user decides.
- This avoids depending on `.gitignore` consent: routine `git add .` can't stage private records, because none are in the worktree.

**Project identity (P4-06):**
- **Git projects:**
  - `project-id` is a random UUID stored in the repository's **common git dir** (`$(git rev-parse --git-common-dir)/jrdev-id`). That location is untracked and never committed.
  - **Symlink aliases:** the path is resolved with realpath first, so aliases map to the same ID.
  - **Linked worktrees** share the common dir, so they're **one project**. That's declared.
  - **Separate clones** get different UUIDs, so they're **separate projects**. That's declared.
  - **A checkout replaced at the same path** has a new `.git`, so it gets a new ID. Old records show up as orphans in `jrdev projects` for the user to delete or relink.
- **Non-git directories:**
  - A mapping in `~/.jrdev/paths.json`, keyed by realpath.
  - This is weaker (a moved directory gets a new ID), and that's declared.
- **Monorepos:** one repository is one project. Sub-project scoping is a later option.
- **Multi-root sessions:**
  - jrdev injects private context **only for the session's starting project**.
  - `DirectoryAdded` runs **after** a directory is added and can't prevent it. jrdev then **doesn't inject** the added project's private context and recommends a fresh session for that project.
- **What we promise:** jrdev controls what **it stores and injects**. It can't remove information already present in a conversation. Changing directories doesn't "un-disclose" context.
- **Acceptance cases:**
  - symlink aliases
  - two clones
  - linked worktrees
  - a replaced checkout
  - `/add-dir` of another repo mid-session

  Each follows its declared identity and injection rule.

## What reaches the AI provider
- **Injected at session start:** `learning:on` topic identifiers, due-review topic IDs, and the current mode rules. **Nothing else by default.**
- **Free-text mission and goal descriptions are project-scoped by default.** Making one global is an explicit choice in setup, with a warning that global text is injected into *every* project.
- **Setup shows a preview** of the exact injected text, and `/jrdev:status --context` shows the live injected content at any time.
- **Controlled-vocabulary topic IDs** reduce, but can't eliminate, the risk of the user typing confidential details into free-text fields. The setup screen states plainly what jrdev controls and what remains the user's responsibility.
- **Assessments in Phase 1** aren't sent to any model (human scoring).

## Data-flow inventory
| Data | Stored | Sent to AI provider? | Human recipients | Retention | Can become public? |
|---|---|---|---|---|---|
| Topic IDs, levels, review queue | `~/.jrdev/global/` | Yes (topic IDs only) | None | Until `jrdev reset` | No |
| Global free-text goal (opt-in only) | `~/.jrdev/global/` | **Yes, in every project** (warned) | None | Until reset | No |
| Project goals, logs, debug notes, maps | `~/.jrdev/projects/<hash>/` | Only within that project's sessions, when used | None | Until `jrdev reset --project` | No |
| Assessment attempts | `~/.jrdev/assessments/` | **No** in Phase 1; Phase 3+ only to the forked scorer if the learner opts in | Human scorer (blind) and a mentor via explicit export | 24 months, or until the learner deletes them | No |
| Submitted assessment archives | Write-once (application-level), per-attempt keys, via a one-time token; executed only in the grading sandbox ([assessment & grading spec](04-assessment-and-grading.md)) | No | Human scorer | 24 months. **Deletion** goes through an authorized path that removes **all object versions** and checks at the storage-version level. Backups expire within 30 days (disclosed). The learner can keep the signed result **without** the archive, and it's then labelled "source deleted; graded bytes no longer retrievable" | No |
| Feedback consent records and deletion receipts | jrdev DB | No | jrdev team | As long as the item exists | No |
| Feedback instruction ledger (deletion markers, consent revisions; no content) | Separate store from the main DB | No | Ops owner | Until the oldest backup that could contain the item has expired, plus 30 days | No |
| Phase 1 planning scores (numeric rubric points, keyed by a random planning ID; pseudonymized, not anonymous) | Private analysis sheet | No | Research owner | Pilot duration plus 12 months; only with the planning consent scope (re-checked at every run). Withdrawal or deletion follows the ordered procedure in [specs/04](04-assessment-and-grading.md#phase-1-manual-route-p6-01). The planning figure frozen in the preregistration is the only retained aggregate | No |
| Study metadata collector logs (client version, model setting, timestamps; both study arms) | Study data store | No | Research owner (pseudonymous) | Study duration plus 12 months | Aggregates only |
| Assessment exposure ledger | Assessment service (authoritative); local append-only cache | No | jrdev team (pseudonymous) | Same as assessment data; deleted with it (prior exposure then becomes "unknown") | No |
| Pseudonym ↔ identity mapping | Separate restricted store | No | Study coordinator only | Study duration plus 12 months | No |
| **Phase 1 manual route:** exposure sheet | Private spreadsheet (coordinator) | No | Coordinator | Pilot duration plus 6 months; deleted on request (exposure then becomes "unknown") | No |
| **Phase 1 manual route:** drop-folder uploads | Upload-only, per-pseudonym file-request folder | No | Coordinator | Emptied as soon as transfer to the grading host is confirmed | No |
| **Phase 1 manual route:** custody log and pilot scores | Private sheet (coordinator, scorer) | No | Coordinator, scorer (pseudonymous) | Pilot duration plus 6 months; deleted on request | No |
| **Phase 1 manual route:** archives on the grading host | Grading host only (never on personal machines) | No | Scorer (pseudonymous) | Deleted after scoring plus the appeal window (30 days); deleted on request | No |
| **Phase 1 manual route:** coordinator messages (task codes, hashes, results) | Coordinator's chat or mail account | No | Coordinator | Deleted at pilot end; deleted on request | No |
| Feedback form | jrdev DB | Only if the submitter allows LLM processing | jrdev team | 24 months | Only as team-written paraphrased themes, or quotes with explicit consent |
| Survey and interviews | jrdev DB / notes | Same opt-in rule | jrdev team | 24 months | Aggregates only |
| Telemetry (opt-in, Phase 2+) | jrdev analytics | No | jrdev team | 12 months | Aggregates only |
| Conversation transcripts | Not collected in Phases 1–2 | n/a | n/a | n/a | n/a |

**Other rules:**
- Behavior review uses **transcripts generated by the team**.
- The hard-off setting disables telemetry and feedback prefill. It doesn't change injection, which [data flows & privacy: what reaches the AI provider](#what-reaches-the-ai-provider) covers.
- Private feedback can reach a public issue or newsletter only with a recorded consent flag.

**Acceptance:**
- A sensitive identifier placed in a **mission, a goal title, an assessment summary and a project log** doesn't reach a different project's context unless that field was explicitly made global.
- The context preview matches what was actually injected.
- Declining every repo write leaves the worktree untouched.
- With a pre-existing tracked `.jrdev/` log, jrdev warns and doesn't untrack it.
- A committed `config.json` works as a team default.
