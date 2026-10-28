# Adversarial review — homepage round 3

Reviewed 2026-10-28. Scope: the current uncommitted [homepage plan](homepage.md), remediation of [round 2](homepage-review-round-2.md), and corresponding [spec 01](../specs/01-website-and-feedback.md), checked against the [overview](../jrdev-ai-plan.md), [privacy spec](../specs/05-data-flows-and-privacy.md) and [practices research](../../research/ai-for-junior-devs.md). Current primary provider documentation was checked on the web. This reviews a plan, not a deployed site or configured account.

**Result: two medium findings, no high or low findings.**

## Round 2 disposition

| Previous finding | Disposition in the current working files |
|---|---|
| **H2-01 — analytics** | **Partially addressed.** Stack now specifies `transformRequest`, the notice acknowledges transient IP/User-Agent processing and a daily hash, and the checklist includes campaign and referrer fixtures. The new “only” claim still understates processing; see H3-01. The [documented transformation](https://plausible.io/docs/stop-tracking-utm-tags) changes the event URL; it does not by itself establish referrer sanitization. The explicit outgoing-referrer check must still pass. |
| **H2-02 — consent bundle** | **Addressed at design level.** [Form fields](homepage.md#form-fields) defines the bundle as the signup promise/form, cadence, track purpose, privacy notice and confirmation email. Every bundle change bumps `consent-vN`; `consent/vN.md` preserves it and the deployed commit. Two fixtures across a copy change must resolve to separate bundles. Provider-held evidence remains to be verified before launch. |
| **H2-03 — pulse readiness** | **Addressed in both documents.** The [readiness row](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03) now requires answer deletion before the first pulse response and records deferred collection in templates. Homepage constraints and checklist prohibit Phase 0 pulses. Subscriber checks remain mandatory. |
| **H2-04 — checklist heading** | **Addressed.** `## Launch checklist` is standalone and the [anchor](homepage.md#launch-checklist) resolves. |

The earlier round-1 dispositions were also rechecked: hard-erasure evidence, exports, transfer/DPA checks, purpose/version records, research limits, metrics and a recurring capacity budget remain present. H1-01's retention caveat has not been fully carried into the notice contract; H3-02 below makes that remaining issue explicit. No private early-signal content has been added to the public cards. No new database or pulse-answer store is required by these fixes.

## H3-01 — Medium: the revised analytics disclosure still incorrectly limits IP/User-Agent use to hashing

**Evidence:** [Privacy page](homepage.md#privacy-page-phase-0-version): “Plausible (EU-hosted) receives each page view with its URL minus query string, the referrer and device type. It uses the IP address and User-Agent only to compute a visitor hash that changes daily, and doesn't store either; retention follows its data policy, linked.” The current [Plausible data policy](https://plausible.io/data-policy), sections “First things first” and “How we count unique visitors”, additionally describes deriving browser/OS information from User-Agent and approximate location from IP, retaining event/session records and grouping events with session identifiers. The homepage checklist says “The privacy text matches what's sent”.

**Failure case:** the implementer uses the standard script plus the specified query stripping and publishes this notice. The processor also derives and retains city/region/country and browser/OS information, while the project says the inputs are used only for a visitor hash and omits session-level records. The outgoing-query fixture passes without detecting these server-side derivations. A link to a broader provider policy does not make the project's narrower assertion accurate.

**Required change:** remove “only” and describe the actual derived categories, daily visitor/session identifiers and event/session storage, alongside aggregate reporting and the absence of stored raw IP/User-Agent. Set or verify the selected account's retention and state its period or applicable criterion; the linked data-policy page does not specify an event/session retention duration. Keep the simple script configuration and EU-hosted service.

**Acceptance:** compare the final notice against the provider's current policy and selected account configuration, including server-derived fields and retention. Query/referrer and storage fixtures still pass. The notice no longer suggests hashing is the sole use of request metadata or that only aggregate records are held.

## H3-02 — Medium: unconfirmed-signup retention still stops at active-list removal

**Evidence:** under [Privacy page → How long](homepage.md#privacy-page-phase-0-version): “unconfirmed sign-ups: removed by Buttondown's cleanup after 30 days (cleanup is turned on)”. Only the separate **unsubscribed** bullet discloses suppression retention. The [launch checklist](homepage.md#launch-checklist) requires that cleanup “removes the fixture after 30 days”; written confirmation of remaining data is explicitly attached to the separate **Erasure** fixture. It acknowledges that “cleanup soft-deletes by default” without requiring the notice to state retention of soft-deleted unconfirmed records. [Buttondown's cleanup documentation](https://docs.buttondown.com/subscriber-cleanup) distinguishes the 30-day unactivated cleanup from hard deletion: cleanup normally preserves soft-deleted subscriber records; support can enable hard deletion. Round 2's H1-01 disposition explicitly required the final notice to cover retained soft-deleted records as well as active-list removal.

**Failure case:** default cleanup removes the pending fixture from the active list at day 30, the explicit erasure fixture separately succeeds through support, and all specified checks are signed off. Unconfirmed records can nevertheless remain soft-deleted in the live provider store beyond day 30. The “How long” list gives no retention period or criterion for that copy; the backup bullet does not cover a live soft-deleted record. The problem concerns notice accuracy, not a claim that every suppression record must be erased automatically.

**Required change:** obtain provider evidence for what unconfirmed cleanup retains and its retention period/criterion, and distinguish active-list removal from retained soft-deleted data in the notice and expiry acceptance. Alternatively, verify and document account-level hard cleanup, including its effect on suppression and re-import safeguards. Keep requested erasure and backup-expiry checks separate.

**Acceptance:** after the unconfirmed fixture expires, evidence identifies any retained address/metadata, its purpose and retention or confirms hard deletion. The final notice covers that state explicitly, and list/export absence alone cannot pass the expiry-retention check. P6-03's independent requested-erasure gate remains intact.

## Coverage and limits

Read all requested context files and both earlier homepage reviews. Homepage relative Markdown file targets and heading fragments were checked against the working files; none was unresolved. The [coding-study source](https://www.anthropic.com/research/AI-assistance-coding-skills) and [math paper](https://doi.org/10.1073/pnas.2422633122) support the cards' population, association and same-session limitations. The copy does not claim demonstrated learning outcomes for this project. P13-01's publication prohibition remains explicit.

[Buttondown's form documentation](https://docs.buttondown.com/building-your-subscriber-base) supports plain POST forms and hidden tags, including possible hosted validation/CAPTCHA journeys; [double opt-in documentation](https://docs.buttondown.com/double-opt-in) supports confirmation before delivery. Its [subscriber schema](https://docs.buttondown.com/api-subscribers-introduction) exposes source, tags and lifecycle transitions. The [DPA](https://marketing.buttondown.com/legal/data-processing-agreement) supports the stated processor/subprocessor and SCC transfer arrangements; account applicability, confirmation evidence, hard deletion and backup retention still require the specified launch evidence. A hidden honeypot is not assumed effective merely because it appears in HTML. No unsupported absence of provider abuse controls is asserted.

[Astro components](https://docs.astro.build/en/basics/astro-components/) support the zero-client-JavaScript default; [Cloudflare Pages Git integration](https://developers.cloudflare.com/pages/configuration/git-integration/) supports deployments on push. The explicitly added analytics script is compatible with that framework-default claim. The unselected analytics alternative is not a launch configuration. No vendor switch is needed to resolve the findings.

Keyboard/screen-reader, JS-off, opt-in, provider-hosted error/CAPTCHA and abuse behavior remain implementation checks, not observed passes. No concrete accessibility defect or commercial-email violation can be established from this design alone; sender requirements depend on actual purpose and applicable jurisdictions. The roughly two-hour weekly allocation has an explicit build budget, recurring issue/admin allowance and cadence fallback; its feasibility remains an estimate to review after launch. No account settings, deployed interface or retention clock was exercised. Only this review file was created; no source plan was edited and no commit was made.

## Reviewed snapshot

SHA-256 identifies the current uncommitted remediation sources.

| Source | SHA-256 |
|---|---|
| `plan/phase0/homepage.md` | `a3adb700a2dff8d139b416325cc785d00abc7b87d9dc9657b993bec9aa39625a` |
| `plan/specs/01-website-and-feedback.md` | `9d2f8bc1bcd1a0a3aa9196213949acd58e9e914812fbc5d3c2e641b9674c9b59` |
