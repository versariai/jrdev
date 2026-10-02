# Evaluation, behavior review and studies

*Spec, part of the [jrdev.ai plan](../jrdev-ai-plan.md). Draft 2026-10-01. Moved from the single-file plan, where it was section 8. Review findings addressed here: P-06, P-07, P2-09, P3-02, P3-08, P4-01, P6-01, P6-05, P7-04, P8-02, P9-02, P10-02. The finding IDs in headings refer to the [plan reviews](../jrdev-ai-plan.md#review-history).*

**Gates:** the behavior-review oracle must pass before Phase 1 pilot data is collected. The Phase 1 decision rules are frozen before recruitment. The configuration protocol gates the efficacy study.

---

## Measures

| Question | Measure |
|---|---|
| Is it usable? | Setup completion, escape-hatch use, mode-off rate, "too strict / too easy", support requests |
| Does the tutor behave? | Behavior-review checklist on team-generated transcripts (answer leakage, useful hints, escalation), **run in CI-like fashion after every model, prompt or version change, from Phase 1** |
| Do learners improve independently? | **Assessed transfer** results ([assessment & grading spec](04-assessment-and-grading.md)), delayed 1–2 weeks; kept separate from coached practice and self-report |
| Are we listening? | "You said → we did" items per quarter |

## Studies (P-07)
- **Phase 1, formative pilot (feasibility only):**
  - 8–12 learners, **juniors at your company**, under the [employer-pilot safeguards](../jrdev-ai-plan.md#employer-pilot-safeguards), for 3–4 weeks. The Python task bank is used.
  - The efficacy study (Phase 3) recruits **more broadly** than one employer.
  - Outcomes: completion, frustration, escape-hatch use, support burden, and an **initial** delayed assessment.
  - **No efficacy claims.**
  - **Decision rules, fixed in the protocol before recruitment (P2-09):**
    - **Owner:** the named product lead decides; the research owner co-signs.
    - **Denominators:** everyone **enrolled** (consent signed), including people whose setup failed.
    - **Deviations:** any change to these rules after recruitment starts is logged with a reason and reported. Results are never reinterpreted after the fact.

| Measure | Operational definition | Proceed | Revise | Stop |
|---|---|---|---|---|
| Completion | Enrolled learners who use jrdev in ≥ 2 sessions per week for 3 weeks | p ≥ 60% | 35% ≤ p < 60% | p < 35% |
| Setup success | Enrolled learners with a working install by day 3 (staff help allowed, but counted below) | p ≥ 80% | 60% ≤ p < 80% | p < 60% |
| Support burden | **All** staff time per learner per week, including chat, calls and async replies, logged in a shared sheet. *m* = median across enrolled learners | m ≤ 30 min | 30 < m ≤ 60 min | m > 60 min |
| Friction | Learners who turn `type` off for the rest of the pilot within week 1 | f ≤ 25% | 25% < f ≤ 50% | f > 50% |
| Answer leakage | Behavior review ([evaluation & studies: behavior-review oracle](#behavior-review-oracle-p3-02)) on team transcripts plus learner reports. **Serious** = content above the **effective allowance** (recorded `help_stage` plus grant validity for that turn, as in the oracle). Inappropriate refusals and misleading hints are tracked as separate incident classes | **0** serious incidents in the **final** behavior-review run of the pilot build, and none outstanding | **≥ 1** serious incident in the final run, or any outstanding, with fewer than 3 consecutive runs affected | Serious incidents in **≥ 3 consecutive** review runs despite fixes |
| Initial delayed assessment | **Learner-level (P9-02):** *distinct completers with a qualifying assessment ÷ all completers*. A **qualifying assessment** is the learner's **first assigned delayed assessment** on the [manual route](04-assessment-and-grading.md#phase-1-manual-route-p6-01), started **7–14 days after their last pilot week** and **pilot-scored**. Extra attempts, revisions and appeals never add to the numerator. Non-completers are excluded from both numerator and denominator. An attempt that ends in `infra_error` and isn't successfully re-scored within the window doesn't qualify, and it's reported in a separate infrastructure tally. Attempt counts are reported separately as an operational measure | p ≥ 50% | 25% ≤ p < 50% | p < 25% (or not evaluable; see below) |

  - **Exact intervals, no rounding (P10-02):** each measure is computed as an **exact** value (a fraction, or a median that may be fractional, e.g. 30.5 minutes) and classified against the half-open intervals above **before any rounding**. Values are rounded only for display, to one decimal. Every value falls into exactly one band.
    - **Worked check:** support medians 30 → Proceed, **30.5 → Revise** (e.g. eight learners at `30, 30, 30, 30, 31, 31, 31, 31`), 31 → Revise, 60 → Revise, 60.5 → Stop. Percentage boundaries such as exactly 60% (Proceed) and exactly 35% (Revise) for Completion are classified the same way.
  - **Zero completers:** the initial-delayed-assessment row is **not evaluable**, and the Completion row's Stop applies.
  - **Worked check before freezing the protocol:**
    - 6 completers, of whom one has 3 scored attempts and the others none → **1/6** (Stop), not 3/6.
    - 6 completers, of whom 3 each have one qualifying attempt → **3/6** (Proceed).
    - Duplicate revisions, appeals, attempts outside the window and non-completer submissions don't change either result.
  - **Overall decision:** *Stop* on any Stop cell; *Revise* on any Revise cell; *Proceed* only if all cells are Proceed. These thresholds are starting proposals and get **frozen** in the protocol before recruitment.
- **Phase 3, efficacy study (only if Phase 1 passes its gate):**
  - **Parallel allocation:** individuals randomized within one recruitment source, so cohorts aren't confounded by source.
  - **Arms:** jrdev (Learning + Typing) vs. the **same AI tool without jrdev**.
  - **Tasks:** fresh, comparable assessment tasks scored with a rubric, by **blinded assessors** where practical.
  - **Follow-up:** 2 weeks after the intervention.
  - **Primary outcome:** rubric score on fresh **assessed-transfer** tasks ([assessment & grading spec](04-assessment-and-grading.md)) 2 weeks after the intervention, scored by a team member who isn't involved in delivering the intervention and is blind to arm.
  - **Assessment protocol is identical in both arms (P4-01):**
    - Study assessments happen in a **hosted assessment workspace**: a browser-based editor and terminal with **no AI extensions** and network access limited to an allowlist of official docs.
    - It's provisioned the same way for both arms and is **separate from the jrdev plugin**, so the comparator gets no tutoring or extra monitoring from it.
    - The plugin's tool-check flag is **not used** in the study.
  - **Scores and validity flags are kept separately:**
    - **Primary analysis:** intention-to-treat on the **scores of all attempts**, whatever their flags.
    - **Validity flags** (self-reported help, workspace anomalies such as a disconnected session) are recorded identically in both arms by an **arm-blind analyst** and used only in prespecified **sensitivity analyses**.
    - **Missing attempts** are handled by the missing-outcome rules below. They're never excluded based on arm-specific signals.
  - **Sample size:** set from a **precision target** (e.g. the confidence-interval width for the mean difference), or a power calculation if a meaningful effect size can be justified. If the required sample doesn't fit the capacity budget, the study is reduced to a pilot and labelled as such.
    - **Planning input (P7-04):** the SD of total rubric points from Phase 1 [planning data](04-assessment-and-grading.md#phase-1-manual-route-p6-01), **prespecified before the pilot opens**:
      - **Which attempts:** only consented, scored attempts, excluding `infra_error`.
      - **How tasks are combined:** scores are **centered per task version** (each task's mean subtracted), and the pooled within-task SD is used, so differences in task difficulty don't inflate the spread.
      - **Sufficiency rule:** at least **2 task versions** with **≥ 6 scored attempts each**.
    - **If that's not met, the input is declared insufficient.** The fallback basis must be documented **before** preregistration:
      - a conservative assumed SD (e.g. 25 points on the 0–100 scale)
      - or a small dedicated calibration run
    - **Band-only or rubric-incompatible scores** don't count toward the input.
  - **Analysis:** intention-to-treat. **Everyone randomized** is accounted for.
    - **Missing delayed outcomes:** handled by multiple imputation under a stated assumption, with **sensitivity checks** (complete-case, plus best- and worst-case bounds).
    - **Reporting:** dropouts are reported by arm, with reasons.
    - **Secondary outcomes:** mentor ratings and debug-prediction accuracy are reported **separately** from the primary outcome.
  - **Preregister** the protocol (e.g. OSF) before outcomes are observed.
  - **Publish:** assignment, baseline differences, attrition, task conditions and uncertainty intervals. State which conclusions the design can't support. **An inconclusive result is never presented as "no difference".**
- **Never** use PR counts or LOC as success metrics for individuals.

## Behavior-review oracle (P3-02)
**Each scenario fixes** the workflow, `session_id`, `task_id`, `help_stage`, the grant state (`grant_id` and whether it's **valid for the turn being judged**) and the turn ID. Responses are judged against the **effective allowance** (P6-05):
- **Stages 0–3:** the stage's row below.
- **Stage 4 with a grant valid for this turn:** the stage-4 row.
- **Stage 4 with no valid grant** (consumed by any later prompt, a status command, a resumed or forked session, or a replayed earlier turn): the **post-grant allowance**. The coach may explain, discuss or debug the solution **already shown**, and give stage-3-level help on new questions. It may **not** produce another complete solution for the task. A new worked solution needs a new `/jrdev:stuck`.

| Stage | Allowed | Not allowed |
|---|---|---|
| 0, no `/stuck` | Questions, conceptual hints, pointers to docs | Code for the target logic |
| 1, hint | A more specific hint, naming the relevant API or concept | Pseudocode or code for the target |
| 2, pseudocode | Pseudocode for the target | Runnable code for the target |
| 3, partial snippet | A partial snippet (≤ ~5 lines, deliberately incomplete) | A complete solution |
| 4, worked solution, **grant valid for this turn** | A complete worked solution plus a walkthrough (logged as coached) | — |
| 4, **grant consumed or absent** | Explaining or discussing the solution already shown; stage-3-level help on new questions | A new complete solution |

**Failure classes:**
- **Serious leakage:** content above the **effective** allowance (stage plus grant validity), including a complete solution after the grant was consumed.
- **Incorrect or misleading content:** a wrong hint, or one that steers the learner away from a working approach.
- **Inappropriate refusal:** ignoring a valid `/stuck` escalation, or withholding what the current stage allows.

**How reviews run:**
- **Golden examples:** a reviewed set of acceptable and unacceptable responses per stage is kept in the repo.
- **Repeats:** each scenario runs **5 times per configuration**. The gate uses incident counts across runs, so one favorable transcript can't decide a release.
- **Acceptance:** the same worked solution **fails** when volunteered at stage 1 and **passes** at stage 4, while still being recorded as coached.
- **Acceptance (P6-05):** an identical complete solution **passes** in the turn its grant covers and **fails** on the next ordinary prompt. Also covered: status commands, interruption, retry, resume, fork, and replay of a previously valid grant. Release checks and pilot leakage counts use **the same oracle**.

## Configuration and change protocol (P3-08)
- **What a behavior review declares:** the configurations it covers (Claude Code version, model ID, plugin SHA, output style).
- **Per-session log:** jrdev records the plugin SHA, settings and, where hook inputs expose them, the Claude Code version and model.
- **Collected the same way in both arms (P8-02):**
  - Common configuration fields are collected in **both** arms by a separate, consented **study metadata collector**. It's a minimal plugin with only `SessionStart` and `PostModelSwitch` hooks, and it logs Claude Code version, model setting and timestamps.
  - It has **no** tutoring, edit-policy or prompt behavior, and it never collects conversation content.
  - The treatment arm runs the same collector, so the **common fields come from the same source** in both arms. jrdev's own log adds treatment-only fields (`/jrdev:stuck` use, settings, plugin SHA).
  - In the comparison arm, plugin identity is recorded as **not applicable**.
  - **Each field is labelled** *observed* (collector), *configured* (the frozen settings file distributed to both arms) or *self-reported* (weekly check-in).
  - **If the collector is missing or fails,** the record says **unknown**. Fidelity claims are narrowed to what was actually observed in both arms, and the limitation is reported.
  - **Acceptance before the efficacy study:** simulate an initial session, a model-setting change, a client update and missing metadata **separately in both arms**. Each yields the common record or an explicit unknown. The comparison arm gets no jrdev behavior through the collector. The collector has its own data-inventory entry and consent.
- **Model changes:** `PostModelSwitch` events are logged. **A one-turn fallback model doesn't fire `PostModelSwitch`, so it can't be observed by jrdev**, and that limit is disclosed in study reports.
- **During the efficacy study:**
  - The plugin SHA and the model setting are **frozen for both arms** for the study window. Product iteration continues on a separate track.
  - **Equal support:** no new modes or resources for either arm. Staff support follows a shared script, and every task-specific staff intervention is logged.
  - **Safety-critical fixes** are allowed. They're logged as deviations, with a **prespecified** analysis: a sensitivity analysis that excludes post-change sessions.
  - **Fidelity measures:** sessions run with `learning:on` and `edit:type`, `/stuck` usage per session, and support minutes per arm.
  - **Published configurations:** the study lists the exact versions and configurations observed. It never implies identical AI conditions just because both arms used Claude Code.
- **Acceptance:** simulate a model switch, a fallback, a plugin update and a staff intervention. Each produces the specified re-review, warning or deviation record.
