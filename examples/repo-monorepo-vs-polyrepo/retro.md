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
disputed_axes:           6    <!-- D1..D6 -->
agreed_r0:               0    <!-- opposing sides by construction; common ground emerged only in R1, via concession -->
converged_in_r1:         3    <!-- D1 atomic-change value, D4 blast-radius tradeoff, D5 onboarding/catalog -->
partly_resolved:         1    <!-- D3 autonomy: A conceded isolation-by-default; residual magnitude rolled into D2 -->
claims_revised:          4    <!-- A3 0.91->0.84, A4 0.87->0.82, B2 0.90->0.85, B3 0.84->0.82 -->
overturned:              0    <!-- neither seat's verdict was overturned -->
surviving_dissent:       1    <!-- D2: can THIS org run a monorepo without a bespoke build system / dedicated build team; +D6 minor (reversibility), low weight -->
resolved_by_debate:      3    <!-- the converged axes -->
resolved_by_oracle:      0    <!-- the discriminating spike is a follow-up test, not run in-panel -->
verified:                0    <!-- no claim was oracle-checked in-panel -->
dropped:                 0
citations_unverified:    all  <!-- seats cited named, checkable public sources (DORA, Google CACM, Netflix, Nx, Merino, Backstage, Brito/Terra/Valente, Bazel, CODEOWNERS, AWS two-pizza); none were independently fact-checked in-panel - this run demonstrates method, not a citation audit -->
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
