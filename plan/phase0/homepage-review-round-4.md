# Adversarial review — homepage round 4

Reviewed 2026-10-28. Scope: the current uncommitted [homepage plan](homepage.md), the [round 3 findings](homepage-review-round-3.md) and earlier dispositions, and [spec 01](../specs/01-website-and-feedback.md), checked against the [overview](../jrdev-ai-plan.md), [privacy spec](../specs/05-data-flows-and-privacy.md) and [practices research](../../research/ai-for-junior-devs.md). Primary research and current provider documentation were checked on the web. This reviews the plan, not a deployed site or configured provider account.

**Result: no medium or high findings; one low finding.**

## Previous findings disposition

| Finding | Disposition in the current working files |
|---|---|
| **H3-01 — analytics disclosure** | **Addressed at design level.** The [privacy section](homepage.md#privacy-page-phase-0-version) now names location, browser, OS and device derivations, the daily visitor hash and retained event/session records; it distinguishes aggregate reports from underlying storage. The account-specific retention period must be stated. The checklist explicitly compares the notice with server-derived fields and account retention, rather than treating browser payloads as complete evidence. This matches the categories in the [current data policy](https://plausible.io/data-policy). |
| **H3-02 — unconfirmed retention** | **Addressed at design level.** The notice and expiry fixture now require provider confirmation of hard deletion or disclosure of what soft-deleted data remains and for how long. The draft's initial “permanently deleted after 30 days” promise is expressly conditional on enabling hard cleanup; otherwise it must be replaced with the confirmed retained state. Requested erasure and backup expiry remain separate checks. [Provider documentation](https://docs.buttondown.com/subscriber-cleanup) still describes soft deletion by default and support-enabled hard deletion. No actual hard cleanup is claimed here. |
| **H2-01 — analytics configuration** | **Remains addressed subject to launch fixtures.** URL transformation and outgoing query/referrer tests remain explicit. The documentation supports [transforming the event URL](https://plausible.io/docs/stop-tracking-utm-tags); that alone is not proof that the referrer is sanitized. The separate referrer fixture remains necessary. H3-01's narrower disclosure issue is now closed above. |
| **H2-02 / H1-03 — consent evidence** | **Remain addressed at design level.** The bundle covers signup promise, cadence, form, track purpose, privacy and confirmation wording; changes bump the version, archived with its deployed commit. Two fixtures must retrieve their separate historical bundles. Provider dates/source and actual confirmation evidence still require implementation verification. |
| **H2-03 / H1-05 — pulse readiness** | **Remain addressed in both documents.** Homepage constraints and templates prohibit Phase 0 pulses. Spec 01's Newsletter paragraph and P6-03 row both place answer deletion before the first pulse response and permit “pulses deferred; no collection in templates”. Subscriber consent/deletion checks remain mandatory before subscribers. |
| **H2-04 — heading** | **Remains addressed.** The standalone Launch checklist heading and its fragment resolve. |
| **H1-01 — erasure** | **Remains addressed at design level, including the H3-02 completion above.** Separate unsubscribe, expiry and requested-erasure fixtures, provider confirmation, administrative exports and backup retention remain in scope. Active-list/export absence is expressly insufficient evidence of erasure. |
| **H1-02 — inventory and transfers** | **Remains addressed at design level.** Controller/contact, signup IP/referrer, recipients/subprocessors, US transfer safeguard, account DPA coverage and retention evidence remain specified. The analytics disclosure correction is accounted for above. |
| **H1-04 — analytics choice** | **Remains addressed.** The selected script-only configuration and storage/payload checks remain; the unselected alternative is not the launch configuration. |
| **H1-06 — research limits** | **Remains addressed.** The coding card identifies post-hoc association and the small subgroup; the math card identifies its population, same-session timing and coding extrapolation. The self-check makes no prevalence claim. No private early-signal material is added to the cards. |
| **H1-07 — metrics** | **Remains addressed at design level.** Confirmed-new counts and the delayed signup-cohort confirmation rate replace submit clicks; the acquisition ratio is explicitly noisy and does not join analytics to addresses. Repeat/existing-address behavior must still be checked when implementing the provider counts. |
| **H1-08 — capacity** | **Remains addressed at design level.** Build hours, recurring allowance, a drafted first issue and monthly fallback remain explicit. The fallback's wording has a minor consistency issue below; no additional system is required. |

## H4-01 — Low: the monthly fallback leaves an unconditional two-week welcome promise

**Evidence:** [Constraints → Capacity](homepage.md#constraints-from-the-plan): “if a four-week window can't fit two issues, the cadence becomes monthly and the page copy says so *before* more sign-ups.” [After launch](homepage.md#after-launch): “**Welcome email:** one short email after confirmation linking the two research reports. It sets the two-week cadence.” [Spec 01 → Newsletter](../specs/01-website-and-feedback.md#newsletter): “**Cadence:** every 2 weeks.” The [consent bundle](homepage.md#form-fields) requires versioning cadence changes, but the fallback instruction explicitly updates only page copy.

**Failure case:** Ivan invokes the documented capacity fallback and updates the signup page and consent version. A new subscriber sees a monthly promise on the page, then receives the separately specified welcome email promising every two weeks. An implementer can avoid this by interpreting the bundle rule broadly, but the welcome-email instruction and parent spec still prescribe the old cadence.

**Required change:** make the welcome email use the current advertised cadence and explicitly include it in the fallback copy update. Qualify spec 01's default cadence with the Phase 0 capacity fallback or a link to it. Keep one selected cadence across the signup, confirmation and welcome copy; no scheduling system is needed.

**Acceptance:** review a synthetic monthly-fallback signup from page through confirmation and welcome. Every cadence statement says monthly, the archived consent bundle matches, and spec 01 permits that documented fallback. The ordinary two-week version remains consistent too.

## Coverage and limits

P6-03 remains a pre-collection gate, including requested subscriber erasure and documented provider backup retention. The revised unconfirmed-record fallback does not waive that gate. P13-01's prohibition and the distinction between proposed habits and demonstrated learning outcomes remain explicit. The one-page launch is scoped separately from feedback, interviews, surveys and install pages; deferring those channels does not open them without their own checks. The approximately two-hour weekly budget is an estimate with a fallback, not verified delivery capacity.

The [coding-study summary](https://www.anthropic.com/research/AI-assistance-coding-skills) supports the iterative-debugging subgroup size and lowest mean quiz score; the [math paper](https://doi.org/10.1073/pnas.2422633122) supports the assisted-practice versus same-session unaided-exam distinction and safeguarded condition. Neither establishes a learning benefit for this project's proposed newsletter or tools.

[Buttondown forms](https://docs.buttondown.com/building-your-subscriber-base) support standard HTML POST and tag inputs and warn that hosted validation/CAPTCHA navigation may be needed. [Double opt-in](https://docs.buttondown.com/double-opt-in) is supported, and the [subscriber schema](https://docs.buttondown.com/api-subscribers-introduction) exposes lifecycle transitions, tags and source. Exact multi-tag, confirmation-record and honeypot behavior remain fixture checks, not assumed passes. The [DPA](https://marketing.buttondown.com/legal/data-processing-agreement) describes US processing, subprocessors and SCC arrangements; it does not independently establish this account's backup expiry or successful erasure.

[Astro components](https://docs.astro.build/en/basics/astro-components/) support the zero-client-JavaScript default; the intentionally added analytics script does not contradict that framework-default claim. [Cloudflare Pages Git integration](https://developers.cloudflare.com/pages/configuration/git-integration/) supports automatic deployments on push. The selected analytics configuration is appropriately narrower than the unselected alternative; the latter's [cookieless documentation](https://posthog.com/tutorials/cookieless-tracking) also distinguishes storage-free operation from person-profile settings. No unsupported vendor defect or mandatory vendor switch was found.

[Consent guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/lawful-basis/consent/how-should-we-obtain-record-and-manage-consent/) supports active opt-in and retrievable consent statements; it does not require an extra checkbox for this single newsletter purpose. The [commercial-email guidance](https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business) turns on the actual message's primary purpose. No commercial-email violation is inferred from a draft free-project newsletter; actual sender identity, message content and applicable jurisdictions still govern its requirements.

Homepage relative Markdown file targets and heading fragments were checked against the working files; none was unresolved. Keyboard/screen-reader, JS-off, hosted error/CAPTCHA, abuse rejection, tracking-off settings, account retention and deletion-clock behavior were not exercised. Those remain implementation and launch acceptance, not observed successes. Only this review file was created; no source plan was edited and no commit was made.

## Reviewed snapshot

SHA-256 identifies the uncommitted sources reviewed; context hashes identify the comparison snapshot, not a new audit of every unrelated requirement.

| Source | SHA-256 |
|---|---|
| `plan/phase0/homepage.md` | `65ae8db550f572dac8ee35743bdb9cedab4adb3ac01706a7347f16d98ccd297a` |
| `plan/specs/01-website-and-feedback.md` | `9d2f8bc1bcd1a0a3aa9196213949acd58e9e914812fbc5d3c2e641b9674c9b59` |
| `plan/specs/05-data-flows-and-privacy.md` | `a79db9644cb3f960913db69572a16daf92e44f0cb9f97314f0c8a4b1d446ffd4` |
| `plan/jrdev-ai-plan.md` | `44c07b74d7086bc1a2045117a0ff674ec6248a4512b736eca56566a9d5a83011` |
| `research/ai-for-junior-devs.md` | `3d1a0bb5a7b8b185cf62ef8e5d761929a37f1c802f80663445a8c068df1090de` |
