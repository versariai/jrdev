# Phase 0 homepage: email capture

*Part of the [jrdev.ai plan](../jrdev-ai-plan.md), Phase 0. Draft 2026-10-28. Owner: Ivan (site, ~2 h/week). Implements the **Home** and **Newsletter** rows of [spec 01](../specs/01-website-and-feedback.md#pages) at the smallest size that does the job.*

## Goal

One page at `jrdev.ai` that explains the problem and collects newsletter subscribers through double opt-in. Nothing else ships with it.

**Success measures** (counted per calendar week, no target before launch; the first four weeks give the baseline):
- **Confirmed new subscribers:** Buttondown subscribers tagged `src-home` whose confirmation falls in the week. Imports and existing subscribers have no `src-home` tag and aren't counted.
- **Acquisition ratio:** confirmed new subscribers ÷ unique visitors (analytics). Aggregate only; confirmations lag visits, so a single week is noisy.
- **Confirm rate:** confirmed ÷ all `src-home` sign-ups created in the week, both from Buttondown, read two weeks later so late confirmations count. Submit clicks aren't used: they include invalid and repeat submits.

Nothing joins analytics to email addresses.

**Out of scope here:** guides, catalog, install pages, the feedback form and the onboarding survey. Each is a separate collection channel with its own [readiness checks](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03). Phase 1 adds the install CTA.

## Constraints from the plan

- **The newsletter gate (P6-03):** no real subscriber until double opt-in, unsubscribe and **subscriber deletion** are tested on team fixtures, and the provider's backup retention is written on the privacy page.
- **Capacity:** the [roadmap](../jrdev-ai-plan.md#5-roadmap) gives site, newsletter and feedback about 2 h/week together, and puts "the website beyond a landing page" last. Budget:
  - **Build and launch checks:** about 8 h once, spread over four weeks.
  - **Recurring:** one issue every two weeks (about 1.5 h each) plus subscriber requests, so about 1 h/week.
  - **Fallback:** if a four-week window can't fit two issues, the cadence becomes monthly. The page, confirmation and welcome email all say so, under a new consent version, *before* more sign-ups. The first issue is drafted before launch, so the commitment is visible.
- **No overclaiming (principles 3 and 4):** nothing exists to install yet. The copy promises a newsletter and an open project, never learning outcomes.
- **Early signals stay private:** the [early-signal themes](../jrdev-ai-plan.md#early-signals-2026-10-02) lack publication consent (P13-01). The page cites only the public research.
- **No pulse survey in Phase 0:** [spec 01](../specs/01-website-and-feedback.md#newsletter) ends each issue with a one-question pulse, but its answers are a separate store with a deletion check. Phase 0 issues and the welcome email collect no answers, polls or reply surveys; pulses start when that check passes.
- **The name is not settled:** the interviews test "jrdev" ([decision: brand](../jrdev-ai-plan.md#7-decisions)). Keep the wordmark to plain text so a rename costs one find-and-replace.

## Stack

| Part | Choice | Why |
|---|---|---|
| Page | Astro, one `index.astro` plus `privacy.astro`, static output | It's in the [suggested stack](../specs/01-website-and-feedback.md#suggested-stack-lightweight-swappable), ships zero JS by default, and MDX guides can land in the same repo in Phase 1 |
| Hosting | Cloudflare Pages (or Vercel) | Free tier, deploy on push |
| Newsletter | **Buttondown** | Built-in double opt-in, tags for track segmentation, full export, API deletion. The form posts straight to its embed endpoint, so there's **no backend and no database of ours** |
| Analytics | **Plausible**, script only, no custom properties, query strings stripped with its `transformRequest` option | No cookies and no form contents; reports are aggregate. It still processes IP and User-Agent to make a daily visitor hash (see the privacy page). The script is the page's only client JS. PostHog would need cookieless mode, person profiles off, autocapture and replay off and URL allowlisting; not worth it for one page |
| Spam | Buttondown's own checks plus a honeypot field | Matches the feedback form's abuse controls |

**Skipped:** a custom signup API, our own subscriber table, a CMS, A/B testing, social login.

## Page structure

Single column, readable on a phone, one form repeated at top and bottom.

1. **Hero**
   - Headline (draft): *Use AI coding tools without skipping the learning.*
   - Sub (draft): *A free, open-source project for junior developers. Practical habits, backed by research, every two weeks.*
   - Email field + **Subscribe** button. Under it: *Double opt-in. Unsubscribe in one click. [What we store](/privacy).*
2. **The problem**, three short cards. Each factual card links its source and keeps its limits visible:
   - **Debugging by pasting errors back:** in an [Anthropic study of developers learning a new library](https://www.anthropic.com/research/AI-assistance-coding-skills), people who looped errors back to the AI without forming a hypothesis scored lowest on the later quiz. *Tag: RCT, post hoc. These patterns weren't randomized and the groups were small (n = 4), so it's an association.*
   - **Assisted gains, unaided losses:** in a [math study with high-school students](https://doi.org/10.1073/pnas.2422633122), unrestricted AI help raised practice scores, but the same students did worse on a closed-book exam later that session. A tutor with safeguards mostly avoided the drop. *Tag: RCT, math, not coding. Applying it to coding is our extrapolation.*
   - **A self-check, not a statistic:** *"Could you explain the last diff you approved?"* This is a question for the reader, with no claim about how common it is.
3. **Three ways to use AI**: learning, onboarding to a codebase, delivery. One line each. These double as the signup tracks.
4. **What you get**: the fixed newsletter sections from [spec 01](../specs/01-website-and-feedback.md#newsletter) (one practice, one tool tip, a stuck-of-the-week theme, the changelog), every two weeks.
5. **Built in the open**: MIT, link to `github.com/versariai/jrdev`, one line on "enforced vs. advisory".
6. **Repeat form** with the optional track checkboxes.
7. **Footer**: privacy, GitHub, contact email.

### Form fields

| Field | Required | Goes to |
|---|---|---|
| Email | Yes | Buttondown subscriber |
| Tracks: learning / onboarding / delivery (checkboxes) | No | Buttondown tags. Purpose, stated next to the boxes: pick which newsletter sections you get first |
| Source and consent version (hidden) | — | Tags `src-home` and `consent-vN`. Check that the embed accepts several hidden `tag` fields |
| Honeypot (hidden) | — | Rejected by Buttondown's checks; verified with a fixture, not assumed |

**Consent record:** Buttondown keeps the signup, confirmation and unsubscribe dates and the source. The `consent-vN` tag links that to the exact text the person saw. A **consent bundle** is the set of consent-relevant copy: the signup promise and form text (including cadence), the track purpose, the privacy page and the confirmation email. Rules:
- any change to the bundle bumps `N`, including the monthly-cadence fallback
- `consent/vN.md` in the site repo holds a copy of the whole bundle and the commit it shipped in; it never contains subscriber data Subscribing is the consent action, so no extra checkbox.

No name, role or company. The onboarding survey collects those later, under its own consent.

## Privacy page (Phase 0 version)

Plain language, one screen. It adds the newsletter rows to the [data-flow inventory](../specs/05-data-flows-and-privacy.md#data-flow-inventory). The final numbers come from the launch checks, not from this draft.

- **Who is responsible:** the controller's name and a contact email for privacy requests.
- **What is collected when you sign up:** email, chosen tracks, the source and notice-version tags, signup, confirmation and unsubscribe dates, and the IP address and referrer that Buttondown records. Open and click tracking is **off**.
- **Why, and on what basis:** to send the newsletter, on your consent (the confirmation click). Withdraw any time with the unsubscribe link.
- **Who receives it:** Ivan, and Buttondown with its listed subprocessors, as processors under Buttondown's DPA. The site host (Cloudflare) sees request logs. Plausible (EU-hosted) receives each page view: the URL without its query string, the referrer, and the IP address and User-Agent. From those it derives approximate location (country, region, city), browser, OS and device type, and a visitor hash that changes daily. It stores event and session records with those derived fields, not the raw IP or User-Agent, and we see only aggregate reports. Retention: the period confirmed for this account, stated here.
- **Transfers:** Buttondown is in the US. Name the safeguard its DPA uses (standard contractual clauses) and how to get a copy.
- **How long:**
  - unconfirmed sign-ups: permanently deleted after 30 days. Buttondown's cleanup soft-deletes by default, so support is asked to switch this account to hard deletion. If that isn't possible, this line states what the soft-deleted record keeps and for how long, as Buttondown confirms in writing
  - unsubscribed: kept on a suppression record so you aren't re-added, for the period the provider states, unless you ask for erasure
  - erasure on request: hard deletion through Buttondown, within 30 days of the request
  - provider backups: the period the provider confirms in writing
- **What can't be undone:** emails already sent; backups until they expire.
- **Your rights:** access, correction, erasure, withdrawal, and complaint to your data protection authority.

## Launch checklist

Run on team fixtures (test inboxes) before the domain is announced:

- [ ] Domain bought; trademark search on "jrdev" done ([Phase 0 scope](../jrdev-ai-plan.md#5-roadmap))
- [ ] Double opt-in: an unconfirmed address gets no newsletter
- [ ] Tracks arrive as tags on the subscriber
- [ ] Hidden tags (`src-home`, `consent-vN`) arrive on the subscriber. Two fixtures signed up either side of a copy change resolve to their own `consent/vN.md`
- [ ] Three fixtures, tested separately:
  - **Unsubscribe:** no further sends; record what stays (suppression) and for how long
  - **Unconfirmed expiry:** cleanup is enabled and the fixture is gone after 30 days (start this one first). Buttondown confirms in writing that it was hard-deleted, or what remains; the privacy line matches
  - **Erasure:** a hard delete through the API or support. Written confirmation from Buttondown says what remains afterwards. Absence from the export is not enough, because cleanup soft-deletes by default
- [ ] Any CSV export made during admin is deleted after use; erasure requests include it
- [ ] Buttondown's DPA covers this account; its transfer safeguard and subprocessor list are linked
- [ ] Provider backup retention confirmed in writing and written on the privacy page
- [ ] Honeypot submission is rejected
- [ ] Page works with JS off (plain form POST) and passes a keyboard-only and screen-reader pass
- [ ] Analytics: no cookies or storage set. Visits with `?utm_source=fixture&e=test@example.com` and from a referring page send no query string; a signup and an error send no email or tracks (DevTools → Application and Network). The privacy text matches what's sent and the derived fields and retention in Plausible's current data policy and account settings
- [ ] Copy check: every factual card has a public source and its limits, no early-signal content, no names
- [ ] First issue drafted; the welcome email and issue template have no poll or survey

The [readiness table](../specs/01-website-and-feedback.md#per-channel-readiness-p6-03) records the pass date.

## After launch

- **Where traffic comes from:** the GitHub README, the research reports, and Ivan's own posts. No paid ads in Phase 0.
- **Welcome email:** one short email after confirmation linking the two research reports. It states the current cadence, the same one as the page and confirmation email.
- **Review after 4 weeks:** the [success measures](#goal) and the track split. A low confirm rate is a reason to look at the confirmation email, not proof it's the cause. The split feeds newsletter segmentation and gives a weak demand signal next to the interviews. It doesn't replace them.

## Open decisions

1. **Newsletter provider:** Buttondown is recommended; Resend or ConvertKit are the alternatives in spec 01. A switch redoes the privacy page and the three deletion fixtures.
2. **Website code public or private** ([still open](../jrdev-ai-plan.md#7-decisions)). Public by default, since there's no secret in a static page.
3. **Launch before or after the name test:** launching now starts the list sooner, but a rename later means migrating the domain and telling subscribers.
