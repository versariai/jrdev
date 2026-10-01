# Public repository and release trust

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 7. Review findings addressed here: P-09, P2-08, P5-01. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the independent verification procedure and the verify-before-enable flow must pass before the Phase 1 pilot build is distributed.

---

**Why public:**
- Anyone can read every hook and script before installing.
- Contributing is itself a learning path.
- Feedback becomes visible and traceable.

## What lives where
| Public repo | Private (never committed) |
|---|---|
| Plugin source, packs, tests | Raw feedback, survey responses, interview notes |
| Research reports, review history, evidence ledger | Subscriber list |
| Website content (and optionally code) | Pilot participant data (only aggregated, anonymized results get published) |
| Public roadmap, changelog, `docs/data-flows.md` | Telemetry, any consented transcripts, secrets |

## Layout (monorepo)
```
jrdev/
├── .claude-plugin/marketplace.json   # entries pinned by git sha
├── plugins/jrdev/  plugins/packs/…
├── community/                        # reviewed contributed packs
├── research/  docs/  site/  tests/
├── .github/{ISSUE_TEMPLATE,workflows,CODEOWNERS}
└── CONTRIBUTING.md  CODE_OF_CONDUCT.md  SECURITY.md  LICENSE  README.md
```

## Contribution and review
- **First-time contributors:** "junior-friendly" issues that come with mentoring notes.
- **PRs touching hooks, policy or scripts need:**
  - decision-table tests
  - a disclosure update (enforced/advisory and data-flow changes)
  - a CODEOWNER review
- **Community packs** go through a checklist before landing in `community/`:
  - licence
  - every file read
  - no remote instruction fetching
  - disclosed network use
  - minimal permissions
  - data-flow entry
  - stated learning intent and limits
- **`SECURITY.md`:** private advisories and a response-time target.

## Release trust: from signing to verified installation (P-09)
- **Immutable identity:** each release has a **commit SHA**. Marketplace entries pin plugins by `sha` (Claude Code supports `ref`/`sha` pinning for git sources), so **a moved tag can't change what's installed**.
- **Dependencies bundled** in `vendor/` with a lockfile. No install-time fetches beyond the pinned source.
- **Version reporting and integrity checking are separate things (P2-08):**
  - **Version reporting:** `jrdev verify-info` (bundled) prints the installed source and commit SHA. It's a **convenience only**: a tampered installation could make it print anything, so it's **never the check for a suspect install**.
  - **Integrity manifest:** every release publishes `MANIFEST.sha256`, the SHA-256 of **every distributed file**: hooks, handlers, skills, output styles, `plugin.json`, libs, `vendor/` dependencies and the bundled `bin/` scripts, including the verifier.
  - **Signing the manifest:** the manifest is signed, using Sigstore/cosign keyless signing tied to the repo's release workflow identity, or minisign with a published key.
  - **Establishing trust:** the signing identity (workflow identity or key fingerprint) is published in the README, on the site and in `SECURITY.md`. Users pin it after first use.
- **Independent verification procedure**, documented on the "Install & verify" page:
  1. Locate the installed plugin directory in the plugin cache (`claude plugin details jrdev` shows its source; the exact cache path is **to be confirmed in Phase 0**).
  2. Verify the manifest signature with **your own** `cosign` or `minisign`.
  3. Run **your own** `sha256sum -c` (or `shasum -a 256 -c`) against the manifest, from inside the installed directory.
  4. Report any extra or missing files: anything listed in neither direction fails.

  The procedure **doesn't execute any bundled code** and **doesn't need git metadata** in the installed cache. A copied SHA label can't make altered file contents pass.
- **We don't assume Claude Code verifies signatures at install.** If it ever does, we'll document it as an additional layer.
- **Verify before any jrdev code runs (P5-01):**
  - **Default off:** the plugin manifest sets **`defaultEnabled: false`**, so a fresh install is **installed but off** until `claude plugin enable`. Plugin hooks run only inside sessions in which the plugin is enabled.
  - **First install:**
    1. `claude plugin install jrdev@<marketplace>` (the plugin stays disabled).
    2. Run the independent procedure above on the **installed on-disk copy**: signature, then `sha256sum -c`, then the extra and missing file check.
    3. Only then run `claude plugin enable jrdev`.

    No jrdev code runs before step 3. The `Setup` hook event doesn't fire on normal startup, and jrdev doesn't use it.
  - **Updates:**
    - **Keep marketplace auto-update off.** It's off by default for third-party marketplaces, and the install page says not to turn it on when manual verification is your trust mechanism.
    - **Update flow:** `claude plugin disable jrdev`, update, verify the new on-disk copy, then `claude plugin enable jrdev`.
    - **Don't run `/reload-plugins`** in a session until the new copy is verified.
    - **Rollback** follows the same flow, pinned to the previous SHA.
  - **Stated limitation:** Claude Code doesn't enforce this gate. It's a **documented procedure**. If a user turns on auto-update, an updated copy loads in the **next session without verification**, and jrdev can't prevent that from inside itself. The install page states this plainly.
  - **Acceptance:**
    - On a clean machine, an altered artifact detected in step 2 never reaches a jrdev handler, because the plugin stays disabled.
    - After a valid install, change the marketplace SHA, then exercise update, reload and restart with the documented flow. New code is verified before it's enabled, and the auto-update caveat is shown.
    - No bundled verifier is executed to decide whether the bundle is trusted.
- **Updates and rollback:**
  - An update is a marketplace change to a new SHA, announced in the changelog, and installed with the disable → update → verify → enable flow above.
  - A rollback pins the previous SHA, using the same flow.
  - Both paths are documented and tested.
- **CI:**
  - decision-table and hook tests on Linux and macOS
  - a **clean-install test** that adds the marketplace, installs, runs every command and checks state and hook decisions
  - a compatibility matrix against the supported Claude Code versions
  - secret scanning, dependency audit, link check
- **Acceptance:**
  - a clean install shows the exact SHA
  - re-pointing a tag doesn't alter an installation pinned by SHA
  - update and rollback both work, including the bundled runtime
  - the independent procedure **detects tampering** with a skill, a lib, a vendored dependency and the bundled verifier itself, and detects an added file

## GitHub as a feedback channel
- **Issue templates:** "I got stuck" (no code), "too strict / too loose", "pack proposal", "bug". Each warns against pasting proprietary code or secrets.
- **Discussions:** Q&A and learning tracks.
- **Monthly triage** merges GitHub items with site feedback. Private feedback is never linked to public issues ([website & feedback: feedback system](01-website-and-feedback.md#feedback-system)).
