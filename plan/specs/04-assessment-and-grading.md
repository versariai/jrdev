# Evidence, assessment and grading

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 5.5. Review findings addressed here: P-02, P2-02, P3-04, P3-09, P4-02, P4-07, P5-04, P5-05, P5-06, P6-01, P6-06, P7-01, P7-02, P7-03, P7-04, P9-02, P9-03. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the grading sandbox fixtures must pass before any pilot submission is executed, on **both** routes. The Phase 1 manual route must pass its trace test before the pilot opens. The narrow-claims and eligibility rules gate Phase 2 progress features.

**Two routes (decision 11, resolved 2026-10-01):**
- **Phase 1 uses the [manual route](#phase-1-manual-route-p6-01)**, sized for 8–12 learners and 1–2 part-time people.
- **The service route** (everything from "Service route" onward) is built **before the Phase 3 efficacy study**. The service-only guarantees are explicitly **deferred**, not approximated: one-time upload tokens, write-once storage, signed finalization and the server-side exposure ledger.

---

## Evidence and the assessment lifecycle (P-02, P2-02)

| Category | Source | Raises |
|---|---|---|
| **Coached practice** | Any work in a tutored session, including Typing exercises, hints, escalations and visible solutions | "Practised" |
| **Self-reported independent** | Learner statement | Nothing; shown separately and labelled as self-report |
| **Pilot-scored** | A Phase 1 manual-route assessment, scored blind but **not signed** | Nothing in the product; shown as "Pilot assessment (manually scored)". Used for the pilot's feasibility measures |
| **Assessed transfer** | Only a **finalized, signed** service-route assessment that is **eligible as independent evidence** ([rules](#result-fields-and-eligibility-p6-06)) | "Demonstrated" |

## Phase 1 manual route (P6-01)
**What it is:** the same learner experience (`jrdev assess start` / `submit`, a minimal payload, blind scoring in the sandbox), but with **people and a few simple tools** instead of the assessment service.

**Roles:**
- **Coordinator:** holds the pseudonym ↔ identity mapping, assigns tasks, and moves archives to the grading host.
- **Scorer:** sees only pseudonyms and archives. The coordinator and scorer are **different people**. If the team has only one person, blinding is **not possible**, and the pilot report says so.

**Lifecycle:**
1. **Assignment and exposure:**
   - The coordinator assigns tasks from the vetted bank using an **exposure sheet** (a private spreadsheet keyed by pseudonym: task, version, date served).
   - A task already in a learner's row is never served as fresh.
   - The sheet is the authoritative exposure record for Phase 1. Local `jrdev restore` can't touch it.
2. **Starting:** `jrdev assess start --manual <task-code>` creates the scratch directory, using the task code the coordinator sent. Public tests and the task text ship in the task package. Hidden tests and reference solutions stay on the grading host.
3. **Submitting:**
   - `jrdev assess submit --manual` builds the archive with the **same minimal-payload rules** as the service route: manifest-only files, metadata stripped, a secret-pattern warning and a confirmed file list.
   - It prints the archive's SHA-256.
   - It writes the "I used no AI or other help" answer into `meta.json`, which contains only the pseudonym and task code.
4. **Transfer, never by email:**
   - The learner uploads the archive to a **per-pseudonym, upload-only file-request link**, such as a cloud-drive file request, where uploaders can't see other files.
   - The learner also sends the hash shown in step 3 through the same form, if it supports a text field, or otherwise in a separate message to the coordinator.
5. **Custody:**
   - The coordinator moves each archive from the drop folder to the grading host and records it in a **custody log**: pseudonym, task code, SHA-256 on receipt, SHA-256 on the grading host, and timestamps.
   - **A hash mismatch stops the attempt.**
   - **Nothing is downloaded to personal machines.** The drop folder is emptied once transfer is confirmed.
   - **Attempts and revisions (P7-01):**
     - **Attempt ID:** each assignment gets `M-<pseudonym>-<task-code>-<n>`. Each accepted submission is a **revision**, starting at 1.
     - **Accepted:** the first **complete** upload whose hash matches the hash the learner reported becomes the accepted submission for that revision.
     - **Interrupted or mismatched transfer:** not accepted. The learner re-uploads under the same revision.
     - **Identical re-upload** (same hash): logged as a duplicate and otherwise ignored.
     - **Changed re-upload after acceptance:** rejected, unless the coordinator **and** the scorer jointly **reopen** the attempt. They log the reason, and a new revision is created. The earlier revision and its score are kept.
     - **Appeals and rescoring:** these add a new score row linked to the same revision, with the rubric version and grader image. **The original score is never overwritten.**
     - **Canonical result:** the latest score of the latest accepted revision. Every earlier row stays in the custody log.
     - **Attempt-level counts:** each **attempt ID counts once** in *operational* counts, however many revisions or rescorings it has. **The pilot gate doesn't use attempt counts.** It uses a **learner-level** measure defined in the [pilot decision table](07-evaluation-and-studies.md#studies-p-07) (P9-02).
     - **Acceptance:** an identical retry, a changed second upload, an interrupted transfer and an appeal. Each ends with one canonical result per attempt, earlier provenance retained, and a single feasibility count.
6. **Scoring:**
   - The scorer runs the hidden tests in the **same grading sandbox** (see [Grading environment](#grading-environment-both-routes-p3-09)) and applies the rubric, blind to identity.
   - The result goes into the custody log as `pilot-scored`, together with the rubric version and grader image. It also records **numeric rubric points** (see planning data below).
   - **Grading environment:** the [task environment contract](#task-environment-contract-both-routes-p5-06-p7-02) applies, including `infra_error` (P7-02).
7. **What the learner sees:** the coordinator returns pass/fail counts and the rubric band. The result shows in the learner's progress view as "Pilot assessment (manually scored)", with **no "verified" badge**.

**Deletion:**
- **Ordered procedure for deletion or planning-consent withdrawal (P9-03).** The coordinator and the research owner follow the same steps:
  1. **Resolve first:** the coordinator looks up **all** of the learner's attempt IDs and planning IDs in the mapping **before anything is deleted**.
  2. **Planning data:** the research owner deletes those planning rows and any exports containing them, and marks affected unpublished derived inputs **invalid**. For a withdrawal of the planning scope only, this step is the whole procedure. The learner's assessment data otherwise stays.
  3. **Assessment data** (full deletion requests only): the coordinator removes archives from the drop folder and grading host, plus the learner's rows in the exposure sheet and custody log.
  4. **Mapping last:** the coordinator deletes the mapping entry **last**, so no derived row is left unlocatable.
- **Backups:** retention for the drive and host backups is documented (target: 30 days).
- **What happens to exposure:** deleted exposure becomes "unknown" if the learner rejoins.

**What a manual result can and can't support:**
- **Can support:** pilot feasibility measures (assessments taken, completion) and the scorer's qualitative notes.
- **Can also support exploratory planning data (P7-04),** but only with that consent scope. This is the input for the [Phase 3 sample-size procedure](07-evaluation-and-studies.md#studies-p-07):
  - **What's recorded:** numeric rubric points (per criterion and total, 0–100), task version and rubric version, with the attempt's assistance observations.
  - **Keyed by a planning ID:** rows use a random **planning ID**, not the attempt ID or pseudonym. Only the coordinator's identity store maps planning IDs to attempt IDs. The sheet is therefore **pseudonymized, not anonymous**, and it stays deletable.
  - **Consent:** a separate scope, "use my pilot scores, de-identified, for study planning".
  - **Owner:** the **research owner** owns the analysis sheet. The coordinator owns the consent register and the mapping.
  - **Export:** rows are copied from the custody log **only** for attempts whose learner holds the planning scope **at export time**, checked against the consent register.
  - **Consent is re-checked at every planning run:** before computing anything, each run drops rows whose planning scope is no longer active.
  - **Permitted use:** estimating score spread for Phase 3 sizing only. Never product evidence, never published per person. Retained for pilot duration plus 12 months.
  - **Withdrawal and deletion (P9-03):** see the ordered procedure below. **Derived inputs** (e.g. a computed SD) that used withdrawn rows are marked **invalid** and recomputed at the next run.
  - **The only irreversible aggregate** is the planning figure **once frozen in the Phase 3 preregistration**. It can't be recomputed after that. The consent text says so, and rows withdrawn afterwards are still deleted.
- **Can't support:**
  - it never becomes "assessed transfer" evidence
  - it raises no demonstrated level
  - it's never presented as signed or verified

**Acceptance (trace test before the pilot opens):** run one synthetic attempt end to end: assignment, start, submit, upload, custody, sandboxed scoring, the result shown to the learner, its count in the pilot denominator, and deletion.
- **Copies:** every identity-bearing copy has a named owner and retention rule. These are the mapping, the drop-folder upload, the coordinator's messages and the grading-host files.
- **Mismatches:** a hash mismatch stops the attempt.
- **Blinding:** the scorer never sees identity.
- **Planning copy (P9-03):** the trace test also exports the synthetic score to the analysis sheet with planning consent. It then withdraws the planning scope (once **before** export and once **after**) and runs a deletion. Withdrawn and deleted rows stop contributing to planning runs, affected derived inputs are invalidated, a consenting control row stays usable, and the mapping is deleted last.

## Result fields and eligibility (P6-06)
Every scored attempt, on either route, stores the score plus these fields. **Observations are append-only**; eligibility is **derived** from them and recomputed whenever a new observation arrives (P7-03):

| Field | Values | Meaning |
|---|---|---|
| `finalization` | `pilot-scored` · `finalized-signed` | How authoritative the score is (route-dependent) |
| `self_report` | `no-help` · `help` · `missing` | The learner's answer at submission |
| `tool_events` | List of detected events (possibly empty) | Hook-observed tool calls touching the attempt (definition below) |
| `monitoring_coverage` | `full` · `partial` · `unavailable` | Whether jrdev hooks were running for the whole attempt window |
| `eligible_independent` | `true` / `false`, with reasons | **Derived**, never written directly |

**What counts as a tool flag:** a jrdev hook recorded a tool call whose `cwd` or target path was **inside the attempt's scratch directory** during the attempt window. That says AI tooling touched the attempt; it **doesn't prove** the help was material.

**Eligibility rule:**
- **Eligible:** `eligible_independent = true` only if **all** of these hold:
  - `finalization = finalized-signed`
  - `self_report = no-help`
  - `tool_events` is empty
  - freshness is **known**
- **Disqualifiers are sticky:** `self_report = help`, a missing self-report, or **any** tool event makes the attempt ineligible. This holds **regardless of the order observations arrive in**. A later loss of monitoring can't remove an earlier tool event or turn a reported "help" into "no help".
- **Disclosed only:** `monitoring_coverage` of `partial` or `unavailable` (jrdev not installed, or hooks not running for part of the window). The claim shows a "help monitoring incomplete" note.
- **Scores are kept** for every attempt, with all observations and provenance.
- **Not eligible yet:** manual-route results (`pilot-scored`).

**Study results:**
- Study analysis follows its own protocol ([evaluation & studies](07-evaluation-and-studies.md)): **all** attempted scores are used, whatever these fields say.
- Study results are **not** automatically imported as product achievements. Importing one requires the learner's consent, and the attempt must pass this eligibility rule.

**Acceptance:**
- assisted, tool-flagged, clean and monitoring-unavailable attempts all keep their scores and provenance
- only eligible attempts raise independent evidence
- the study's all-attempt analysis is unaffected
- order cases (P7-03): reported help plus unavailable monitoring; a tool event followed by a hook failure; no-help plus partial monitoring. Each yields the same eligibility whatever the processing order

## Service route (built before the Phase 3 efficacy study)

**Assessments run outside the AI session**, so the tutor can't leak hints and no model grades the work:
1. **Start:** `jrdev assess start <topic>` in a plain terminal.
   - The **assessment service** picks a task from a vetted bank that this learner **hasn't been exposed to before**, according to an exposure ledger (see below).
   - Creates a scratch directory **outside** any repo, records the start time and the allowed resources (e.g. official docs only), and prints the task.
   - **Hidden tests and reference answers aren't on the learner's machine.**
2. **Work:** the learner solves it without an AI assistant.
   - **Self-reported condition:** a submit-time checkbox "I used no AI or other help". **Ticking "I used help" makes the attempt coached.**
   - **Tool check (product use only, not in studies):** if a jrdev-hooked Claude Code session records tool use inside the scratch directory during the window, the attempt gets a **validity flag**. The **score itself is kept**, and the flag is a separate field.
3. **Submit:** `jrdev assess submit` packages the submission and runs the public tests.
   - **Minimal payload (P5-05):**
     - Each task version declares a **submission manifest**: the exact paths or globs graders need.
     - Everything else is **excluded by default**: dotfiles, `.env*`, `.git/`, editor and history files, logs, notebook outputs, build artifacts.
     - **Before upload**, the learner sees the exact file list and sizes and confirms them. Adding anything outside the manifest requires a deliberate choice, with a warning.
     - **Metadata is stripped:** archives are built with normalized owner, permissions and timestamps, and no VCS metadata.
     - **Secret-pattern scanning** warns about likely secrets. It's **not presented as guaranteed redaction**.
     - **Identity stays out of the payload:** the upload token maps to the pseudonymous attempt server-side, so graders receive no account, email or git-author data.
     - **Acceptance:** scratch seeded with a valid solution, a private `.env`, identifying metadata, generated logs and a required fixture. Only manifest files are sent, the fixture still works, and graders see no identity data.
4. **Score:** a human scorer applies the rubric and the hidden tests, **blind** to the learner's identity and to whether they're in a study arm.
5. **Finalize:** the scorer marks the attempt finalized through the assessment service. Only then does an "assessed transfer" record exist.

**Submission identity and finalization authority (P4-02):**
- **One-time upload token:**
  - Each attempt has an `attempt_id` and a `submission_revision`.
  - The service issues a **one-time upload token** bound to `(attempt_id, revision)`. It's **not** a reusable pre-signed URL.
- **Write-once storage:**
  - The service computes the **SHA-256 of the received archive** and stores it in **write-once storage** (no overwrite).
  - **Storage model (P5-04):**
    - **Immutability is enforced by the application**, through no-overwrite object keys and versioning: `attempts/<attempt_id>/<revision>/<digest>`.
    - **No compliance-mode retention lock is used**, so prompt deletion stays possible.
    - **No sharing across attempts:** objects are **not deduplicated** across attempts or learners, so no reference counting is needed and one learner's deletion never affects another's evidence.
  - Re-uploading the same digest is a no-op.
  - A **different** digest after acceptance is rejected, unless a scorer reopens the attempt. That creates revision n+1 and is logged.
- **The grading record binds:**
  - the submission digest and storage version
  - task and rubric versions
  - the grader image digest
  - the hidden-test version
  - the result
- **Grading:**
  - Runs are idempotent per (digest, grader image, test version).
  - The [task environment contract](#task-environment-contract-both-routes-p5-06-p7-02) applies.
  - A database constraint allows **one finalization per revision**, so repeated callbacks can't produce conflicting results.
- **Signed results:**
  - Only an authenticated scorer can finalize.
  - The service **signs** the finalized result, and the local client shows an achievement as *verified* only after checking that signature.
  - Locally edited or unsigned records are displayed as **unverified**.
- **Blinding:** scorers see pseudonymous attempt IDs. The identity mapping is stored separately, with restricted access.
- **Acceptance cases:**
  - reusing an upload token
  - changing files after submission
  - a double upload
  - importing a fabricated "finalized" record
  - duplicate grading callbacks

  None may silently change certified evidence, and every achievement resolves to the exact graded bytes.

**Exposure ledger (P4-07):**
- **Where the record lives:** the **authoritative exposure record is server-side** (the service serves the tasks), keyed by pseudonymous learner ID.
- **Local backup and restore don't touch it:** the local cache `~/.jrdev/exposure.log` is append-only, and `jrdev restore` and migration snapshots **never rewrite it**.
- **Missing history means "unknown":** if exposure history is unavailable (offline, deleted, or a new device without login), the attempt's freshness is marked **unknown**. Such attempts can't count as protocol-controlled transfer evidence.
- **Retention and deletion:** the ledger follows the assessment data's retention. If a learner deletes their data, their ledger goes too. If they rejoin, prior exposure is **unknown** rather than assumed absent. This is disclosed.
- **Practice on repeated tasks** is allowed, and is labelled "previously exposed".

**Interruption and abandonment:**
- An attempt stays `in progress` for up to 24 hours, then becomes `abandoned`, which is never counted.
- A restart always uses a **new** task, and the exposure log is updated.
- **Delay doesn't establish novelty.** Novelty comes from the exposure log.

## Grading environment, both routes (P3-09)
**Required before any pilot submission is executed**, on the manual route as well as the service route:
- **Where it runs:** a dedicated grading host (a VM or CI runner) with **no credentials or personal files**, separate from scorer workstations. Scorers view results; they don't run submissions on their own machines.
- **One disposable container per submission:**
  - no network (`--network none`)
  - non-root user and a read-only base image
  - a size-limited writable scratch area (tmpfs)
  - CPU, memory and process-count limits, and a **wall-clock timeout**
  - destroyed after each run, so nothing carries over to the next submission
- **Intake:** archives are size-checked and safely extracted **inside** the container. Absolute paths, `..`, symlinks and device files are rejected.
- **Test bank protection:**
  - Hidden tests are mounted read-only for the run. **Reference solutions are never mounted.**
  - The learner receives only pass/fail counts and the names of public tests. Full output stays with the scorer and is reviewed before anything is shared.
  - Any task whose hidden tests could have leaked through returned output is marked **exposed** and retired from fresh-task selection.
- **Transfer:** on the service route, `jrdev assess submit` uploads over HTTPS with a one-time token to write-once storage (see above). On the manual route it goes through the upload-only drop folder and the custody log ([manual route](#phase-1-manual-route-p6-01)). The storage is listed in the data-flow inventory ([data flows & privacy: data-flow inventory](05-data-flows-and-privacy.md#data-flow-inventory)).
- **SHA pinning of the plugin says nothing about participant code.** This boundary is what protects the scorer.
- **Acceptance fixtures**, which must pass before the pilot:
  - external network access
  - reading a seeded host secret
  - writing outside scratch
  - archive path traversal
  - an infinite loop
  - memory exhaustion

  Each must be contained or terminated, leave the next run clean, and leak no reference answer through logs or diagnostics.

## Task environment contract, both routes (P5-06, P7-02)
- **Pinned before assignment:** each task version pins **one environment image** before the task is assigned, on **either** route. The image covers the runtime and dependency versions, preinstalled resources, the test command, resource limits, and seeded or deterministic inputs.
- **One image everywhere:**
  - The same image is used for local public tests (`jrdev assess start` pulls it), the hosted study workspace (Phase 3) and the hidden-test grader.
  - The task package states the **supported public-test conditions**: run the public tests in the provided image. Results from a learner's own runtime are informational only.
  - Intentional differences are listed in the task text.
- **Infrastructure failures are separate from learner failures:**
  - An image mismatch, a missing dependency, a provisioning error or a grader crash gets the status `infra_error`. A **valid but incorrect** solution is scored normally.
  - `infra_error` is **never** recorded as a learning result.
  - Retries re-run the **same accepted submission bytes**, so no new task exposure is needed.
  - On the manual route, an `infra_error` attempt counts **once** in feasibility numbers, under its own "infrastructure failure" tally, never as a scored or failed assessment.
- **Acceptance:** a fixture exercising each declared feature and dependency passes in the public-test and hidden-test environments (and in the hosted workspace, in Phase 3). A deliberately missing dependency or wrong image produces `infra_error`, not a low score, and the retry gets no duplicate feasibility credit.

## What a passed assessment claims (P3-04)
- **Narrow skill IDs:**
  - Assessments map to **narrow skill IDs** (e.g. `python.exceptions.handling`), declared per task.
  - A task can raise only the skill IDs it declares, and only through an explicit, documented mapping. **Umbrella labels like "backend" or "debugging" are never raised by a single task.**
- **Global summary record:**
  - **Fields:** `{skill_id, task_id, task_version, rubric_version, category, scorer, scored_at, score_band, evidence_ref}`.
  - **Rubric changes** create a new `rubric_version`. Historical results keep their original version and are displayed with it.
- **The progress view shows concrete achievements** ("Passed task T v2 under conditions C on date D, scored by a human") with recency.
- **General levels are deferred** until aggregation and recency rules are defined and reviewed (not before Phase 3).

**Later automated scoring (Phase 3+):**
- It runs in a skill with `context: fork`, a fresh subagent that **doesn't see the conversation history**. The subagent gets only the rubric and the submission, with the submission marked as untrusted data.
- Automated grades can raise "demonstrated" only after **agreement with human scores is validated** on a held-out set. Disagreements go to a human.

**Reports** separate protocol-controlled conditions (fresh task, hidden tests, rubric, blind scoring) from self-reported ones (no outside help).

**Acceptance:** none of the following can produce a qualifying record:
- a worked solution sitting in the chat
- unsolicited hints (no AI is in the loop in Phase 1)
- resuming after an interruption
- an answer that contains grading instructions (human scorer in Phase 1; untrusted-data handling and validation later)
