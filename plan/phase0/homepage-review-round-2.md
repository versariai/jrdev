# Adversarial review — homepage round 2

Reviewed 2026-10-28. Scope: the current uncommitted [Phase 0 homepage plan](homepage.md), its round-1 remediation and the corresponding changes to [spec 01](../specs/01-website-and-feedback.md), checked against the [overview](../jrdev-ai-plan.md), [privacy spec](../specs/05-data-flows-and-privacy.md) and [practices research](../../research/ai-for-junior-devs.md). Current provider documentation and consent guidance were checked on the web. This reviews the plan, not a deployed site or configured provider account.

**Result: two medium findings, two low findings, no high findings.**

## Round 1 disposition

| Previous finding | Disposition in the current working files |
|---|---|
| **H1-01 — erasure** | **Addressed at design level.** The homepage separates unsubscribe, unconfirmed expiry and hard erasure; requires written provider confirmation rather than export absence; includes administrative CSV copies and backup retention. Actual provider behavior remains a launch gate, not an established result. Cleanup still defaults to soft deletion according to the [current documentation](https://docs.buttondown.com/subscriber-cleanup); the final notice must describe retained soft-deleted records as well as active-list removal. |
| **H1-02 — inventory and transfers** | **Substantially addressed.** Controller/contact, IP/referrer, processors/subprocessors, consent basis, rights, complaint route, DPA applicability and US-transfer safeguards are now specified. The new analytics statement has a separate factual/configuration gap, H2-01. |
| **H1-03 — consent evidence** | **Partially addressed.** Source and dated notice tags, provider lifecycle dates and historical privacy notices are specified. The newsletter promise/form version is still not explicitly part of the archived consent evidence; see H2-02. |
| **H1-04 — analytics** | **Original provider-configuration problem addressed by selecting Plausible.** Script-only collection and payload/storage checks are sensible. The replacement's default behavior does not fully match the new wording or query-parameter acceptance; see H2-01. |
| **H1-05 — pulse gate** | **Collection deferral addressed in both documents.** Homepage constraints prohibit Phase 0 polls/reply surveys, its launch checklist checks the templates, and spec 01's Newsletter section makes pulses conditional. The readiness row was not reconciled; see the Low inconsistency H2-03. |
| **H1-06 — research limits** | **Addressed.** Cards identify the coding study's post-hoc association and small subgroup, and the math population, unrestricted/safeguarded distinction and same-session exam. Coding extrapolation is explicit. The unsupported prevalence claim is replaced by a self-check. Public primary links are present; private early-signal publication remains prohibited. |
| **H1-07 — metrics** | **Addressed at design level.** Calendar-week confirmed-new counts, an explicitly noisy acquisition ratio and a signup-cohort confirm rate replace submit clicks; no email/analytics join is introduced. Provider counting, including repeats and existing addresses, still needs fixture verification. |
| **H1-08 — capacity** | **Addressed at design level.** Eight build/check hours over four weeks, about one recurring hour/week, a drafted first issue and a monthly fallback are explicit. These are estimates, not evidence of delivery capacity; they no longer assume newsletter work is free. |

## H2-01 — Medium: the selected analytics defaults do not match the disclosure and query-parameter check

**Evidence:** [Stack](homepage.md#stack) selects “Plausible, script only, no custom properties” and describes it as “Aggregate by design”. The [privacy section](homepage.md#privacy-page-phase-0-version) says “Plausible gets aggregate page visits with no identifiers.” The launch checklist requires that “the network payloads for a signup and an error contain no email, tracks or query parameters”. Plausible's current [data policy](https://plausible.io/data-policy), updated September 2026, describes individual event/session records, daily visitor identifiers derived from IP and User-Agent, session identifiers and retained campaign/referral parameters. Its [query-stripping instructions](https://plausible.io/docs/stop-tracking-utm-tags) prescribe `transformRequest` to remove the supported attribution parameters too.

**Failure case:** a normal homepage visit carrying `?utm_source=fixture` uses the selected standard script and retains attribution data despite the stated no-query-parameters check. A misguided personalized campaign value could consequently reach analytics. Even with harmless URLs, the public “no identifiers” description remains inaccurate: aggregate dashboard output does not mean the processor receives only aggregates or uses no identifiers. This does not establish that Plausible requires a consent banner or stores raw IP addresses; its policy expressly says it does neither under its documented setup.

**Required change:** describe transient IP/User-Agent processing, daily visitor/session identifiers and aggregate reporting accurately. Specify the documented URL transformation for the current script if all query parameters are to be excluded, and check referrer payloads as well. Record the selected service's region and retention. Keep the simple analytics choice and no custom properties; no replacement analytics system is necessary.

**Acceptance:** fixtures with an ordinary query, campaign parameters containing a synthetic address/track value, and a referring URL show that the actual outgoing URL/referrer fields meet the declared policy. Inspect browser storage and provider-held fields as well as the dashboard. The final privacy notice matches the documented identifiers, processing, region and retention; the default script is not assumed to satisfy stricter requirements merely because it is cookieless.

## H2-02 — Medium: the consent version identifies the privacy notice but not the signup promise

**Evidence:** [Form fields](homepage.md#form-fields) defines `notice-YYYY-MM-DD` as “the privacy page version shown” and promises that “Every published version of the privacy page stays in the site repo's git history with its date”. The [capacity fallback](homepage.md#constraints-from-the-plan) changes the advertised cadence to monthly and says “the page copy says so before more sign-ups”. The version contract never says to preserve or version the newsletter promise/form copy alongside that privacy notice. [ICO consent-record guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/lawful-basis/consent/how-should-we-obtain-record-and-manage-consent/) distinguishes the dated data-capture form/consent statement from the separate privacy information and requires evidence linking the submission to the applicable versions.

**Failure case:** the advertised cadence or newsletter scope changes while the privacy page still says only “to send the newsletter”. Subscribers before and after the change share a notice tag. The specified archive establishes which privacy information existed, but does not guarantee a retrievable copy of the signup promise that each person accepted. An operator following the documented privacy-only version rule can pass the hidden-tag check without closing H1-03's original what-was-consented-to gap.

**Required change:** extend the existing version identifier to cover an archived consent bundle: signup promise/form text, optional-track purpose, privacy notice and relevant confirmation wording. Bump it when any consent-relevant copy changes, including the fallback promise; map it to an immutable repository revision. Keep subscriber records at the provider and historical public copy free of addresses. Add retrieval of the bundle to launch acceptance, rather than checking only that a tag arrives.

**Acceptance:** two synthetic subscriptions across a cadence/copy change resolve to their exact historical signup and notice text, signup method, confirmation evidence and withdrawal status. A shared unchanged privacy notice cannot obscure the differing promises. This can use the existing static repository and provider metadata; a new database or redundant consent checkbox is unnecessary.

## H2-03 — Low: the readiness table still makes pulse deletion an unconditional newsletter-opening check

**Evidence:** [Spec 01, Newsletter](../specs/01-website-and-feedback.md#newsletter) now says pulses start “once the pulse-answer deletion check ... passes” and “Until then, issues carry no survey”. Its [P6-03 readiness table](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03), under “Consent and deletion checks before opening”, still requires “Pulse-survey answers can be deleted” in the newsletter row. The homepage explicitly defers pulses until their own check passes.

**Failure case:** a launch reviewer follows the table literally and blocks the survey-free newsletter until an unused survey store is built and tested. Another reviewer follows the new conditional paragraph and signs off. The conflict encourages needless work within the two-hour weekly allocation, although the explicit no-survey instruction prevents an immediate unauthorized collection route.

**Required change:** qualify the table's pulse check as required before the first pulse response, and allow newsletter readiness to record “deferred; no pulse collection in templates”. Keep subscriber consent/deletion checks mandatory before the first subscriber.

**Acceptance:** the homepage checklist, Newsletter paragraph and readiness row give the same answer for a survey-free launch; enabling pulses later requires a separately recorded pass before any real response.

## H2-04 — Low: the launch-checklist heading is embedded in a privacy bullet

**Evidence:** the [privacy section](homepage.md#privacy-page-phase-0-version) ends with the exact text “complaint to your data protection authority.## Launch checklist”. There is no standalone `## Launch checklist` line in the file.

**Failure case:** Markdown renders the checklist title as part of the rights bullet instead of a heading. Heading navigation omits the launch checks, and `homepage.md#launch-checklist` does not resolve to a generated heading. This also makes the prior review's checklist evidence link unusable against the revised document.

**Required change:** place the heading on its own line with a blank line before it.

**Acceptance:** rendered Markdown exposes a level-two Launch checklist heading and a working `#launch-checklist` anchor, separate from the privacy rights bullet.

## Coverage and limits

The overview's P6-03 and P13-01 gates, publication/enforcement principles and capacity allocation were checked against the remediation. No new private early-signal material is needed for the proposed public cards. The linked [coding-study summary](https://www.anthropic.com/research/AI-assistance-coding-skills) supports the iterative-debugging subgroup and its lowest average quiz score; the [math paper](https://doi.org/10.1073/pnas.2422633122) supports the revised domain, assistance-condition and same-session distinctions. No tested learning benefit is claimed for the project's own proposed habits.

Buttondown's [embed documentation](https://docs.buttondown.com/building-your-subscriber-base) supports standard form POSTs and hidden tag inputs, and warns that hosted validation/CAPTCHA navigation can be necessary. Its [subscriber API reference](https://docs.buttondown.com/api-subscribers-introduction) documents lifecycle transitions, source and tags; the exact confirmation evidence and repeat-submission behavior still need account fixtures. Its [current DPA](https://marketing.buttondown.com/legal/data-processing-agreement), section 8, provides SCC arrangements for US transfers. None of these documents proves that this account's erasure, backup retention or honeypot check has passed. Those remain explicit prelaunch checks, including reconciliation of the final retention wording with soft deletion.

Astro's [component documentation](https://docs.astro.build/en/basics/astro-components/) supports its zero-client-JavaScript default, and Cloudflare Pages documents [automatic deployments from Git](https://developers.cloudflare.com/pages/configuration/git-integration/). Adding the analytics script intentionally introduces client JavaScript; the framework-default claim alone does not promise a script-free page. The unselected analytics alternative is not a launch configuration to validate.

Relative file targets and the homepage's existing Markdown link fragments were checked; no missing target was found among those links. The absent checklist anchor is recorded separately above. No deployed form, screen-reader journey, CSS contrast, account setting, deletion fixture or provider retention clock was tested. Keyboard/screen-reader, JS-off, opt-in and abuse fixtures remain appropriate gates; a hidden honeypot is not assumed to enforce rejection. The actual sender identity, newsletter purpose and applicable jurisdictions still govern email-law details; no commercial-email violation is inferred merely from a free newsletter. Only this review file was created; no source plan was edited and no commit was made.

## Reviewed snapshot

SHA-256 identifies the two uncommitted remediation sources reviewed here.

| Source | SHA-256 |
|---|---|
| `plan/phase0/homepage.md` | `3213e8a72615520864a429108163f4cc7a3f9e3c45f8ad8c4abc92c70bac23ff` |
| `plan/specs/01-website-and-feedback.md` | `942d00eb06f028b08fb5d30313393447e5e1421efb394f547bb7a16a2e71851a` |
