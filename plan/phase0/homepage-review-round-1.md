# Adversarial review — homepage round 1

Reviewed 2026-10-28. Scope: the current uncommitted [Phase 0 homepage plan](homepage.md), checked against the [overview](../jrdev-ai-plan.md), [website spec](../specs/01-website-and-feedback.md), [privacy spec](../specs/05-data-flows-and-privacy.md) and [practices research](../../research/ai-for-junior-devs.md). Tool and legal guidance linked below was checked on the web on this date. This reviews a design, not an implemented site.

**Result: six medium findings, two low findings, no high findings.**

## Previous round disposition

There is no previous homepage review. Round 17 supplies the report format only; its experiment findings are outside this scope.

## H1-01 — Medium: export absence does not establish subscriber erasure

**Evidence:** [Launch checklist](homepage.md#launch-checklist): “Deletion through Buttondown removes the record, and export no longer shows it”. [Privacy page](homepage.md#privacy-page-phase-0-version): “Retention: until you unsubscribe or ask for deletion. Unconfirmed sign-ups are purged after 30 days.” Buttondown documents that cleanup must be enabled, unactivated records are cleaned after 30 days, unsubscribed records after seven days, and cleanup uses soft deletion by default; hard deletion requires contacting support ([subscriber cleanup](https://docs.buttondown.com/subscriber-cleanup)).

**Failure case:** the fixture disappears from the active list/export but remains soft-deleted at the provider. Alternatively, cleanup is never enabled and unconfirmed records remain past the promised deadline. The checklist can pass without establishing the erasure required by P6-03.

**Required change:** distinguish unsubscribe, suppression, soft deletion and erasure; verify the exact API/support erasure route and enable/test cleanup. State actual active-record, suppression and backup retention, including any exceptions, rather than promising immediate removal at unsubscribe. Keep exports used during administration in the deletion procedure. Do not infer hard deletion from export absence.

**Acceptance:** a fixture follows unsubscribe, unconfirmed expiry and explicit erasure separately; provider documentation or written confirmation establishes what remains and for how long. The privacy wording matches that evidence, and the readiness table records the actual erasure route before real subscribers.

## H1-02 — Medium: the privacy inventory omits provider-collected personal data and transfer details

**Evidence:** [Privacy page](homepage.md#privacy-page-phase-0-version) lists “email, chosen tracks, confirmation date” and “Who sees it: Ivan only”, with “Buttondown (US). Link their privacy policy and DPA.” Buttondown also lists subscriber IP addresses and referrer metadata ([provider disclosure](https://buttondown.com/legal/gdpr-eu-compliance)); its [DPA](https://marketing.buttondown.com/legal/data-processing-agreement), sections 5 and 8, covers subprocessors and US transfers with SCC provisions. [EU guidance](https://europa.eu/youreurope/business/governance-and-sustainability/digital-and-data-compliance/data-protection-gdpr/index_en.htm) requires clear information on the controller, purposes, legal basis, recipients, rights and international-transfer safeguards where applicable.

**Failure case:** a reader reasonably believes only Ivan receives a small email record, although the direct signup sends their IP/referrer to a US processor and its subprocessors. A link to the processor’s notice does not explain the project’s own controller obligations or rights route.

**Required change:** disclose actual signup data, provider/subprocessor access, purposes and lawful basis; identify the controller and contact, applicable rights and complaint route, and the relevant transfer safeguard and how to obtain it. Verify the DPA applies to the selected account and assess applicable transfer obligations. Include hosting/analytics processing where relevant. This does not require a new database or a large legal page.

**Acceptance:** compare a fixture request and provider record with the notice; every retained category and recipient is covered. Document the applicable DPA/transfer arrangement before collecting real addresses; do not imply newsletter consent itself authorizes every international transfer.

## H1-03 — Medium: confirmation date alone cannot demonstrate what was consented to

**Evidence:** [Privacy page](homepage.md#privacy-page-phase-0-version) stores a “confirmation date”; [Form fields](homepage.md#form-fields) specifies only email, tracks and honeypot. No notice/copy version or signup provenance is specified. [ICO consent guidance](https://cy.ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/lawful-basis/a-guide-to-lawful-basis/consent/) calls for evidence of who consented, when, how and what they were told.

**Failure case:** the project renames or changes cadence/purpose after launch. An old confirmation timestamp proves address control but cannot establish the newsletter promise or privacy notice shown to that subscriber.

**Required change:** define a minimal provider-held consent record linking signup/confirmation to the versioned form promise and privacy notice, with source and withdrawal status. Verify supported embed metadata or another proportionate method; keep historical notice versions free of subscriber data. State the purpose of optional tracks. Do not add a mandatory checkbox merely to duplicate the Subscribe action.

**Acceptance:** for a synthetic subscriber, retrieve evidence of the exact notice and newsletter purpose accepted, signup method, confirmation and later withdrawal. Repeat across a notice-version change without retaining private addresses in the public repo.

## H1-04 — Medium: disabling persistence is not the specified cookieless analytics configuration

**Evidence:** [Stack](homepage.md#stack) equates “PostHog with persistence off” with “No cookie banner”; [Privacy page](homepage.md#privacy-page-phase-0-version) promises “aggregate, no personal identifiers”; the checklist checks only cookies. Current [PostHog documentation](https://posthog.com/docs/privacy/data-collection#cookieless-tracking) specifies project enablement and `cookieless_mode: "always"`, and treats data capture, identification and storage as separate controls. It warns that identification can negate cookieless privacy protections.

**Failure case:** the implementation disables persistence but retains default event capture, identifiers or sensitive URL properties. The no-cookie check passes while the actual data exceeds the aggregate-only notice.

**Required change:** pick the provider before launch and specify its documented privacy configuration. For PostHog, explicitly address cookieless mode, person profiles, IP handling, autocapture/replay and event/URL allowlisting; otherwise use the simpler aggregate alternative. Review network payloads and storage, not just cookies. State any applicable consent basis based on the chosen configuration rather than assuming no cookies resolves all obligations.

**Acceptance:** a fixture signup, error and confirmation journey sends no email, track values, sensitive query parameters or unintended identifiers to analytics; browser storage and project configuration match the notice. The selected provider’s retention/region are documented.

## H1-05 — Medium: the newsletter promise leaves the pulse-survey collection gate unresolved

**Evidence:** [Page structure](homepage.md#page-structure) promises “the fixed newsletter sections from spec 01”; [website spec, Newsletter](../specs/01-website-and-feedback.md#newsletter) says “Every issue ends with a one-question pulse survey.” Its [newsletter readiness row](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03) requires “Pulse-survey answers can be deleted”. The homepage checklist covers subscriber deletion only and declares surveys out of scope without explicitly deferring newsletter pulses.

**Failure case:** the first issue follows spec 01 and links a provider poll or reply collection with no defined answer retention/deletion procedure. Subscriber deletion passes while survey answers remain elsewhere.

**Required change:** explicitly defer pulses until their own collection/deletion checks pass, or name the answer store, scope, retention, provider copies and deletion route and add its fixture acceptance. Reconcile this with the newsletter spec rather than silently omitting a gate.

**Acceptance:** either the Phase 0 welcome/issue template has no pulse collection and the spec records that deferral, or a synthetic answer is deleted from every defined answer store before the first real response.

## H1-06 — Medium: research cards lose essential limits and an asserted common pattern lacks a public source

**Evidence:** [Page structure](homepage.md#page-structure) says “Assisted practice went up while the unaided exam went down” with only an RCT tag, and “Approving diffs you skimmed. Stated as a common pattern, without a statistic”. The [research evidence table](../../research/ai-for-junior-devs.md#1-what-the-evidence-says) describes a high-school mathematics study, unrestricted versus safeguarded assistance, and an exam in the same session. The homepage’s own [constraints](homepage.md#constraints-from-the-plan) say “The page cites only the public research.” No public source is supplied for the third card.

**Failure case:** junior readers infer the exam result tested coding tools or all AI tutoring. The third card asserts prevalence without evidence; removing a statistic does not supply support or satisfy the public-research-only rule.

**Required change:** identify the math population, unrestricted-assistance condition, same-session endpoint and safeguarded-tutor nuance in compact visible copy; label coding application as extrapolation. Supply a public source for the diff-review claim or frame it as a reader self-check/recommendation rather than a common observed pattern. Give the first card its primary research link and keep post-hoc association limits. Do not reuse private early-signal material.

**Acceptance:** each factual card has a public supporting source and retains domain/design limits; no card presents a suggested habit as a demonstrated learning intervention. A reader can distinguish coding evidence, non-coding extrapolation and advice without opening a paper.

## H1-07 — Low: confirmation metrics lack a counting contract

**Evidence:** [Goal](homepage.md#goal) measures “visit → confirmed rate”, while [Stack](homepage.md#stack) counts “visits and the submit click”; [After launch](homepage.md#after-launch) calls for “the confirm rate”. The form leaves the site for provider-hosted confirmation.

**Failure case:** imports, existing subscribers or delayed confirmations are divided by unrelated visits; submit clicks include invalid or abandoned submissions. A low ratio is attributed to the confirmation email without evidence.

**Required change:** define the time window, confirmed-new-subscriber numerator and visit denominator, and distinguish an aggregate acquisition ratio from submit-to-confirm conversion. Use provider counts where available; do not introduce email-based analytics joining for this baseline.

**Acceptance:** a worked fixture with repeat clicks, a delayed confirmation and an existing subscriber produces unambiguous counts and a report that states attribution limits.

## H1-08 — Low: ongoing newsletter work is absent from the capacity estimate

**Evidence:** [Constraints](homepage.md#constraints-from-the-plan) budgets “about one week of site time, then near zero upkeep”; [Page structure](homepage.md#page-structure) promises four sections “every two weeks”. The [roadmap](../jrdev-ai-plan.md#5-roadmap) assigns Ivan about two hours per week for site, newsletter and feedback together.

**Failure case:** a two-hour build allowance is mistaken for the whole launch cost; research checking, drafting, accessibility checks, subscriber requests and recurring issues compete with Phase 0 discovery.

**Required change:** separate build time from an explicit small recurring issue/admin budget and define a fallback cadence or launch deferral when it cannot fit. A first issue/template drafted before launch would make the commitment reviewable.

**Acceptance:** a four-week allocation includes two issues and subscriber administration within the existing owner budget, or the advertised cadence is revised before signups.

## Coverage and limits

Read all five requested source files and the round-17 format. Relative Markdown file targets and heading anchors in the homepage were checked against the working files; no broken target was found. Astro’s zero-client-JavaScript default and Cloudflare Pages push deployments are consistent with their [component](https://docs.astro.build/en/basics/astro-components/) and [Git integration](https://developers.cloudflare.com/pages/configuration/git-integration/) documentation. Buttondown’s [embed documentation](https://docs.buttondown.com/building-your-subscriber-base) supports plain form POSTs and tags; it also warns that users may encounter hosted validation/CAPTCHA flows. Optional checkboxes are therefore feasible, not a finding by themselves.

No running site, account configuration, provider erasure or accessibility test was executed. The existing keyboard/screen-reader and honeypot fixture gates are appropriate; the plan must verify server-side honeypot handling rather than assume a hidden field enforces rejection. No unsupported claim of a missing provider rate limit is made. Backup retention remains an explicit unresolved launch gate, not a discovered numeric guarantee. The public early-signal prohibition is retained; no private feedback is reproduced here. Email-law applicability and any sender-address requirements should be confirmed for the actual newsletter purpose/jurisdictions; a commercial-email violation is not asserted merely because this free project has a newsletter. Only this review file was created; no source plan was edited and no commit was made.
