# Retrospective - repo-monorepo-vs-polyrepo

<!-- This is a demonstration run published as a worked example. T0 is filled from
     the panel record; T1 (real-world outcome) is illustrative here since no team
     actually shipped from this run - in a real panel you would stamp it when the
     decision's outcome is known. -->

## T0 - at verdict (process facts)

verdict_mode:            decide-after-check   <!-- run the spike oracle before committing A or B -->
verdict_date:            2026-08-21
rounds_run:              R0, R1
stop_rule:               none - seats were assigned opposing sides, so R0 could not agree; ran R1, dissent survived, routed to an oracle
panel_type:              role-incentivized stress-test (Seat A championed monorepo, Seat B championed polyrepo; roles as incentives, not personas)
seats:                   codex-cli (GPT-5-class), grok-cli (Grok-4-class)  <!-- 2 vendors, heterogeneous -->
seat_deaths:             0
claims_total:            11   <!-- Seat A: 5, Seat B: 6 -->
agreed_r0:               0    <!-- opposing sides by construction; nothing agreed pre-exposure. The common ground emerged only in R1, via concession -->
disputed:                0    <!-- terminal status: every contested claim resolved to `conceded` (6) or `surviving-dissent` (5); none left transiently `disputed` at verdict -->
conceded:                6    <!-- contradicting seat yielded in R1: A1, A3, A4, A5, B3, B5 (confidence also revised on A3, A4, B3; B2 revised but stayed surviving-dissent) -->
overturned:              0    <!-- neither seat's verdict was overturned -->
verified:                0    <!-- no claim was oracle-checked in-panel -->
dropped:                 0
surviving_dissent:       5    <!-- 5 claims carry terminal `surviving-dissent` status (A2, B1, B2, B4, B6); they cluster into one load-bearing dispute in the verdict (D2: can THIS org run a monorepo without a bespoke build system / dedicated build team), with B6/reversibility a minor tail -->
resolved_by_oracle:      0    <!-- the discriminating spike is a follow-up test, not run in-panel -->
resolved_by_debate:      6    <!-- the 6 `conceded` claims, resolved by R1 cross-examination -->
citations_dropped:       0    <!-- no citation was checked-and-failed -->
citations_unverified:    0    <!-- no citation was relayed with the UNVERIFIED stamp. NB: no in-panel citation audit ran at all, so this 0 means "none formally flagged", not "all verified"; the seats cited named public sources (DORA, Google CACM, Netflix, Nx, Merino, Backstage, Brito/Terra/Valente, Bazel, CODEOWNERS, AWS) that this demo did not independently check -->
cost_notes:              4 CLI invocations (2 Round 0 + 2 Round 1), a few minutes wall-clock each
friction:                grok needed all inputs staged into one cwd + read-only allow-rules (else narration-only exit); codex needed stdin closed (`< /dev/null`). Both produced clean, claim-shaped, well-cited output.

## T1 - when the outcome is known

outcome_date:            (illustrative example - not a shipped decision)
outcome:                 n/a
verdict_held:            no-signal
dissent_confirmed_for:   none   <!-- would be set by running the two-week spike + the PR-frequency read -->
missed_entirely:         none observed

## TEMPLATE DELTA (mandatory)

template_delta:          Both seats, arguing opposite sides, **independently proposed nearly the identical discriminating test** (a ~two-week monorepo spike on 1 shared lib + 2 services, measuring affected-CI time and unrelated-red hours with no dedicated owner). That convergence is a strong signal: when both adversarial seats name the same falsifying experiment, the *intersection of their two falsifiers* is an unusually trustworthy oracle - stronger than either seat's own confidence. Worth a one-line note in the r1-seat and verdict templates: **"if opposing seats each name a settling test, adopt their intersection as the oracle and pre-register its pass/fail before running it."** Second observation (recorded, not yet a core change): a role-incentivized panel on a genuinely no-consensus question reliably produces the ideal decision-grade output - a *preserved* dissent plus a *named cheap test* - rather than a smoothed average; this is the mode to reach for when the honest answer is "it depends, and here's exactly what it depends on."
