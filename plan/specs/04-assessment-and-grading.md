# Evidence, assessment and grading

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 5.5. Review findings addressed here: P-02, P2-02, P3-04, P3-09, P4-02, P4-07, P5-04, P5-05, P5-06. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the grading sandbox fixtures must pass before any pilot submission is executed. The narrow-claims rules gate Phase 2 progress features.

---

## Evidence and the assessment lifecycle (P-02, P2-02)

| Category | Source | Raises |
|---|---|---|
| **Coached practice** | Any work in a tutored session, including Typing exercises, hints, escalations and visible solutions | "Practised" |
| **Self-reported independent** | Learner statement | Nothing; shown separately and labelled as self-report |
| **Assessed transfer** | Only a **finalized** assessment (below) | "Demonstrated" |

**Phase 1: assessments run outside the AI session**, so the tutor can't leak hints and no model grades the work:
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
  - **Task environment contract (P5-06):**
    - Each task version pins **one environment image**: the runtime and dependency versions, preinstalled resources, the test command, resource limits, and seeded or deterministic inputs.
    - **One image everywhere:** the same image is used for local public tests (`jrdev assess start` pulls it), the hosted study workspace and the hidden-test grader. Intentional differences are listed in the task text.
    - **Infrastructure failures are separate from learner failures.** An image mismatch, a missing dependency, a provisioning error or a grader crash gets the status `infra_error`. It's **never finalized as a learning result**. Retries re-run the **same submission revision**, so no new task exposure is needed.
    - **Acceptance:** a fixture exercising each declared feature and dependency passes in all three environments. A deliberately mismatched image produces `infra_error`, not a low score.
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

**Grading environment (P3-09), required before any pilot submission is executed:**
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
- **Transfer:** `jrdev assess submit` uploads the archive over HTTPS with a one-time token to write-once storage (see above). The storage is listed in the data-flow inventory ([data flows & privacy: data-flow inventory](05-data-flows-and-privacy.md#data-flow-inventory)).
- **SHA pinning of the plugin says nothing about participant code.** This boundary is what protects the scorer.
- **Acceptance fixtures**, which must pass before the pilot:
  - external network access
  - reading a seeded host secret
  - writing outside scratch
  - archive path traversal
  - an infinite loop
  - memory exhaustion

  Each must be contained or terminated, leave the next run clean, and leak no reference answer through logs or diagnostics.

**What a passed assessment claims (P3-04):**
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
