# Website, newsletter and feedback

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 3. Review findings addressed here: P-03, P3-06, P3-07, P6-03, P8-01, P9-01, P10-01. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates (P6-03):** each collection channel must pass its consent and deletion acceptance **before its first real participant or subscriber**. That **includes Phase 0 discovery**. Team-generated fixtures can be used while a channel is unfinished. See [Per-channel readiness](#per-channel-readiness-p6-03).

---

## Pages
| Page | Purpose | Phase |
|---|---|---|
| **Home** | Value proposition; the three modes of using AI (learning / onboarding / delivery); subscribe and install CTAs | 0 |
| **Newsletter** | Double opt-in signup, archive | 0 |
| **Feedback** | See [website & feedback: feedback system](#feedback-system) | 0 |
| **Privacy & data flows** | The data-flow inventory ([data flows & privacy spec](05-data-flows-and-privacy.md)) in plain language | 0 |
| **Guides** | The research reports rewritten for juniors, keeping the evidence tags | 1 |
| **Skills catalog** | For each pack or mode: what it does; **what is enforced and what is advisory**; what it writes to disk; **what it sends to the model**; supported tool and versions; pinned SHA; known limitations | 1 |
| **Install & verify** | Per-tool install steps, plus how to verify the installed version ([repo & release trust: release trust: from signing to verified installation](06-repo-and-release-trust.md#release-trust-from-signing-to-verified-installation-p-09)) | 1 |
| **Changelog** | Release notes plus "you said → we did" items | 1 |
| **For mentors/teams** | Rollout guidance, the behavior-review practice | 2+ |

## Newsletter
- **Cadence:** every 2 weeks. Fixed sections:
  - one practice
  - one tool or mode tip
  - one "stuck of the week" (a paraphrased theme, never a quoted submission without consent)
  - a changelog
- **Segmentation:** by the learning tracks chosen at signup.
- **Every issue ends with a one-question pulse survey.**

## Feedback system
**Inputs:**
1. **Onboarding survey**, about 3 minutes:
   - role and experience
   - tools
   - stack
   - how they use AI
   - top 3 difficulties
   - learning goals
2. **"What are you stuck on?"**, a free-text box that can be anonymous.
3. **Per-skill feedback** from the catalog page, and `/jrdev:feedback` in the tool, which opens a prefilled form with **no code or transcripts attached**.
4. **Interviews** with consenting volunteers.
5. **GitHub issues and Discussions** (public; [repo & release trust: gitHub as a feedback channel](06-repo-and-release-trust.md#github-as-a-feedback-channel)).

**Processing loop (monthly):**
```
collect → tag → cluster themes → prioritize (frequency × severity × feasibility) → backlog → ship → announce → re-survey
```
**Rules** (P-03):
- **LLM-assisted clustering** of feedback is disclosed on the form. Submitters can opt out of LLM processing, and opted-out items are clustered by hand. Clusters get a human review before they drive priorities.
- **Public themes board:** only paraphrased themes written by the team. **Private feedback is never linked to, or quoted in, public issues or newsletters** unless the submitter explicitly agrees.
- The form warns submitters not to paste proprietary code or secrets. The team reviews for sensitive content before anything is reused.

**Consent and deletion lifecycle (P3-06):**
- **Consent record:** each submission stores a versioned consent record with separate scopes: `llm_processing`, `quote_publication`, `contact`.
- **Checked at the moment of action:** every external processing job and every publication step reads the *current* consent record when it runs, not the consent copied into its queued payload. Consent withdrawn between queueing and running means the job is skipped.
- **Deletion receipt:** anonymous submitters get an opaque **deletion receipt** (a random token shown once at submission). Without a receipt or an account, we can't promise to locate an anonymous item, and the form says so.
- **What deletion removes:**
  - the primary row
  - queued jobs
  - cluster memberships
  - derived summaries that reference the item
  - interview notes linked by item ID

  Backups age out within 30 days, and that is stated.
- **Honest limits:**
  - material already sent to an LLM processor is subject to that processor's retention terms, which are disclosed
  - newsletters already sent can't be recalled
- **Publication:**
  - **Distinctive incidents** (identifiable workplace, person or event) are **never published, even paraphrased**, without `quote_publication` consent.
  - Generic themes may be paraphrased after a reviewer checks them against an identifiability checklist.

**Restoring from backup can't undo deletions or withdrawals (P8-01):**
- **Instruction ledger:**
  - Deletions and consent changes are also written to a small, **append-only instruction ledger**, kept in a **separate store**, separately backed up, from the main database.
  - **Entry fields:** item ID or deletion-receipt hash, action, consent scope and revision, and timestamp.
  - **No feedback content** is ever stored in it.
- **Write protocol: the ledger is authoritative and written first (P9-01):**
  1. **Each request gets an ID:** a deletion or consent change gets an `op_id` (UUID) and a per-item revision number.
  2. **The ledger commit comes first:**
     - **Entries are committed durably** to the ledger store, a transaction committed with replication or `fsync`, **before** the user sees success.
     - **On failure, nothing is acknowledged:** if the ledger write fails, the request returns an error and the user retries.
  3. **The database copy is applied after:**
     - The instruction is then applied to the primary database **idempotently**.
     - The database records each applied `op_id`, so a retry with the same `op_id` is a no-op.
  4. **Consent checks read the ledger:** processing and publication jobs read the **latest ledger revision** for the item at run time, and the database copy is only a cache. **An unapplied withdrawal in the ledger still blocks the job.**
- **Segments, checkpoints and completeness (P9-01, P10-01):**
  - **Sealed segments:**
    - The ledger is a series of **sealed segments**, for example one per month.
    - Within the open segment, entries carry a gapless sequence number and a **hash chain**: each entry includes the previous entry's hash.
  - **Checkpoints:** sealing a segment writes an **authenticated checkpoint**, signed with the ops key. It records:
    - the segment's sequence range and last hash
    - the previous checkpoint's hash, so **checkpoints form their own chain**, which is never pruned and stays tiny
    - a **state snapshot**: for every item still inside its retention, its **latest consent revision** and **deletion status**. Only IDs, scopes, revisions and flags are stored, never content.
  - **The authoritative current state** is the latest checkpoint's snapshot **plus** the entries after it. Jobs read consent from that state, so **pruning old entries never loses an item's current consent**: it's carried forward in every checkpoint.
  - **Mirrored heads:** the mirrored head records `(checkpoint ID, sequence, hash)` in the two independent places (the primary database's "last applied" marker, and the append-only ops log in a third location).
  - **What a verifier expects:**
    1. The checkpoint chain verifies from genesis.
    2. Entries after the latest checkpoint are gapless and chain from that checkpoint's last hash.
    3. The head is at or beyond every mirrored head.

    **A sequence range covered by a valid checkpoint is legitimately absent. Any gap not covered by a checkpoint is corruption** and blocks processing.
  - **Ledger storage:** durable, replicated storage with point-in-time recovery. A restored ledger goes through the same verification.
- **Restore runbook:** after any database restore, run these steps in order:
  1. **Quarantine:** pause all processing workers, publication jobs and theme recounts.
  2. **Verify, then apply the current state:** verify the ledger (above), then apply the **authoritative current state** (latest checkpoint snapshot plus later entries) to the restored database, idempotently by `op_id`. This works for backups on **either side** of any checkpoint, because it applies the current state rather than replaying pruned history.
  3. **Resume** only once replay completes and is logged.
- **If the ledger is unavailable or incomplete,** external processing and publication stay **blocked** until consent is re-established. Old consent in a restored snapshot is never treated as fresh authorization.
- **When an instruction can disappear (P10-01):**
  - **Superseded entries:** an individual ledger entry disappears only when its **whole sealed segment** is pruned. A segment may be pruned only after a later checkpoint covers it, at which point every entry in it is captured in that checkpoint's snapshot or has expired.
  - **An item's current state** (latest consent revision, deletion flag) leaves the checkpoint snapshot only at the **first checkpoint after** both of these hold:
    - **(a)** for a **deleted** item: the oldest backup that could still contain the item has expired, plus 30 days. For an **active** item: its 24-month feedback retention has ended and the item itself has been deleted.
    - **(b)** no restorable backup newer than that point still references it.

    Until then, a deletion stays effective for every restorable backup.
  - **Content** is never in the ledger, so pruning only shortens the window in which minimal markers exist. Access is limited to the ops owner.
- **Maintenance is crash-safe.** Each step is idempotent and runs in this order:
  1. Write the new checkpoint durably.
  2. Update both mirrored heads to reference it.
  3. Prune the segments it covers.

  **If interrupted:** before step 2, the previous checkpoint stays authoritative. Before or during step 3, extra covered segments simply remain, which the verifier accepts. **No step can leave an uncovered gap.**
- **Provider-held data** (newsletter provider, transcription service) follows each provider's documented backup and restore behavior. It's listed per channel, and no guarantee is claimed beyond what the provider documents.
- **Acceptance, before the affected channel opens:**
  - Back up three synthetic items, delete one, withdraw LLM and publication consent from another, then restore the old snapshot.
  - The deleted item stays absent from records, counts and jobs. The withdrawn item never reaches a processor or publication step. The third resumes normally.
  - Simulate a lost ledger: processing stays blocked.
  - **Pruning, before pruning is enabled (P10-01):**
    - Interleave instructions for two synthetic items, expire only one item's retention, and checkpoint and prune.
    - Verification still succeeds, the expired markers are gone, and the other item's current consent is unchanged.
    - Restore backups taken **before and after** the checkpoint.
    - Interrupt maintenance at each of its three steps.
    - Legitimate pruning passes verification, while removing an **uncovered, unexpired** entry fails it.
  - **Crash injection (P9-01):** interrupt before and after the ledger commit, and before the success response. Retry the same withdrawal or deletion, then restore an earlier database.
    - Every **acknowledged** instruction stays effective.
    - Retries are idempotent.
    - A truncated ledger (head behind a mirrored head) blocks processing.

**Counting and triage (P3-07):**
- **The themes board reports three separate numbers per theme:**
  - submissions
  - distinct submitters, best-effort (accounts or hashed emails when given; anonymous items counted as "unverified")
  - **corroborated observations** (seen in the pilot, interviews or behavior review)
- **Duplicates:**
  - near-duplicates and cross-channel copies are merged by a human reviewer
  - the anonymous form has proportionate abuse controls (a honeypot field and rate limits)
  - perfect anonymous deduplication is not claimed
- **Severity is assessed separately and isn't diluted by low frequency.** Any item touching privacy, safety, data loss or a blocked learner gets an explicit decision even if it appears once.
- **Separate audiences:** feedback from the **pilot audience** is kept apart from public-channel feedback when judging demand.
- **Transparency:** the board shows the evidence basis and uncertainty, never an unsupported "affected users" count.

## Per-channel readiness (P6-03)
A channel opens to real people only after its checks pass on **team-generated fixtures**:

| Channel | Opens in | Consent and deletion checks before opening |
|---|---|---|
| Newsletter (chosen provider) | Phase 0 | Double opt-in works. Unsubscribe and **subscriber deletion** in the provider actually remove the record; the provider's backup retention is documented. Pulse-survey answers can be deleted |
| Website feedback form | Phase 0 | Consent record per scope. **Restore reconciliation test** passes (P8-01). **Withdrawal between queueing and job execution** skips the job. A deletion receipt removes the primary row, queued jobs, cluster memberships and derived summaries |
| Onboarding survey | Phase 0 | The same consent and deletion checks as the feedback form |
| Interviews (recordings and notes) | Phase 0 | Consent form with scopes. Defined storage for recordings and notes, and who holds them. Deletion removes the recording, notes, derived theme summaries that cite the participant, and the scheduling record. Transcription services are disclosed, along with their retention |
| GitHub issues and Discussions | Phase 0–1 | Templates warn against pasting proprietary code. The triage process never copies private feedback into public threads |
| In-tool `/jrdev:feedback` | Phase 1 | Opens the same form; no code or transcripts attached |
| Pilot assessments (manual route) | Phase 1 | Covered by the [assessment spec's trace test](04-assessment-and-grading.md#phase-1-manual-route-p6-01) |

For each channel, **document which copies can't disappear immediately** (provider backups, already-sent emails), with their expiry.

## Suggested stack (lightweight, swappable)
- **Site:** Astro or Next.js with MDX content, hosted on Vercel or Cloudflare Pages.
- **Newsletter:** Buttondown, Resend or ConvertKit. The subscriber list must be exportable.
- **Feedback:** a native form posting to a small API with Postgres (e.g. Supabase). Set a retention policy ([data flows & privacy spec](05-data-flows-and-privacy.md)).
- **Analytics:** privacy-friendly (Plausible or cookieless PostHog).
- **Before committing:** check that `jrdev.ai` is available and what it costs, and run a trademark search on "jrdev".
