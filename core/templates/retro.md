# Retrospective - [decision name]

<!-- The flywheel. Two passes: T0 at verdict time, T1 when the real-world
     outcome is known (days/weeks later). A retrospective with an empty
     TEMPLATE DELTA is data collection, not a flywheel - the delta field
     is mandatory, even if it says "no change".

     Evaluation principles (borrowed from evidence-cited session-eval
     practice): every finding here must point at a run artifact (ledger
     row, seat file, verdict section) - unpointed impressions are struck;
     conclusions roll up from the recorded facts, not from anyone's
     overall vibe; where the record can't show something, write
     "no signal", never a guess; keep the field names below stable and
     one per line so retrospectives stay comparable ACROSS runs with
     nothing but grep - your panel directories are the analytics store:

         grep -h '^verdict_held:' */retro.md | sort | uniq -c
         grep -h '^overturned:'   */retro.md
         grep -h '^dissent_confirmed_for:' */retro.md | sort | uniq -c

     No service, no signup. If you separately run your own metrics stack,
     mirroring these counts into it is fine (metadata only - never claim
     text, question text, or file paths); the mirror never closes the
     loop: only TEMPLATE DELTA does. -->

## T0 - at verdict (process facts)

verdict_mode:            <!-- ship | dont | decide-after-check | human-call -->
verdict_date:
rounds_run:              <!-- e.g. R0+R1 -->
stop_rule:               <!-- which stop rule ended the panel -->
seats:                   <!-- vendor/model/effort per seat, comma-separated -->
seat_deaths:             <!-- count, incl. narration-only failures; name the seat -->
claims_total:
agreed_r0:
disputed:
conceded:
overturned:
verified:
dropped:
surviving_dissent:
resolved_by_oracle:
resolved_by_debate:
citations_dropped:
citations_unverified:
cost_notes:              <!-- wall-clock and/or spend, per seat if known -->
friction:                <!-- free text: anything that cost time besides the debate -->

## T1 - when the outcome is known

outcome_date:
outcome:                 <!-- free text: what actually happened -->
verdict_held:            <!-- yes | no | no-signal -->
dissent_confirmed_for:   <!-- seat name(s) whose surviving dissent proved right; or none | no-signal -->
missed_entirely:         <!-- claims nobody made that reality surfaced; or none -->

## TEMPLATE DELTA (mandatory - the loop-closer)

<!-- One concrete proposed edit per line: target file (brief wording, a
     core/templates/ file, seat selection, tier/effort practice) + a
     one-sentence change + which T0/T1 field above motivates it.
     Or exactly: "no change" + why.
     Repo policy: a recurring delta (≈3 similar) backed by these run
     records is exactly the evidence CONTRIBUTING.md asks for to change
     core/ - turn it into a PR or your local overlay. -->

template_delta:
