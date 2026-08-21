# Retrospective - api-auth-jwt-vs-sessions

<!-- This is a demonstration run published as a worked example. T0 is filled
     from the panel record; T1 (real-world outcome) is illustrative here since
     no team actually shipped from this run - in a real panel you would stamp
     it when the decision's outcome is known. -->

## T0 - at verdict (process facts)

verdict_mode:            decide-after-check   <!-- adopt B, then gate ship on the chaos/load oracle -->
verdict_date:            2026-08-21
rounds_run:              R0
stop_rule:               a - Round 0 already agreed on everything decision-relevant
seats:                   codex-cli (GPT-5-class), grok-cli (Grok-4-class)  <!-- 2 vendors, heterogeneous -->
seat_deaths:             0
claims_total:            7   <!-- after dedup across both seats -->
agreed_r0:               5
disputed:                0
conceded:                0
overturned:              0
verified:                0   <!-- the discriminating chaos/load test is a follow-up oracle, not run in-panel -->
dropped:                 0
surviving_dissent:       0
resolved_by_oracle:      0
resolved_by_debate:      0
citations_dropped:       0
citations_unverified:    0    <!-- none relayed with an UNVERIFIED stamp; NB no in-panel citation audit ran, so 0 means "none formally flagged", not "all verified" -->
cost_notes:              two single-shot Round 0 CLI invocations, ~a few minutes wall-clock; no Round 1
friction:                none - both seats returned clean, claim-shaped, well-cited output on the first try

## T1 - when the outcome is known

outcome_date:            (illustrative example - not a shipped decision)
outcome:                 n/a
verdict_held:            no-signal
dissent_confirmed_for:   none
missed_entirely:         none observed

## TEMPLATE DELTA (mandatory)

template_delta:          A hard, well-specified MUST in the brief (here: "revoke within seconds") did most of the adjudication by eliminating a whole design branch in isolation - worth a one-line note in brief.md that front-loading the single hardest constraint often collapses a seemingly-open choice to near-consensus and earns an honest early stop. (Recorded as an observation; not a core rule change on the strength of one run.)
