# Retrospective - fixture: multiple template_delta lines

<!-- Fixture for the flywheel-export regression test. Repeated
     `template_delta:` lines are the documented house format ("one concrete
     proposed edit per line"). This fixture carries three real deltas AND a
     trailing "no change" line: before v1.1.9 the exporter read only the LAST
     occurrence, matched /no change/, and emitted had_template_delta:false,
     silently discarding the three real deltas. -->

## T0 - at verdict (process facts)

verdict_mode:            ship
rounds_run:              R0
seats:                   codex, grok
seat_deaths:             0
claims_total:            4
agreed_r0:               4
disputed:                0
conceded:                0
overturned:              0
verified:                0
dropped:                 0
surviving_dissent:       0
resolved_by_oracle:      0
resolved_by_debate:      0
citations_dropped:       0
citations_unverified:    0

## TEMPLATE DELTA (mandatory - the loop-closer)

template_delta: adapters/example - a real proposed edit; motivated by friction.
template_delta: brief-writing - a second real proposed edit; motivated by friction.
template_delta: core/templates/retro.md - a third real proposed edit; motivated by friction.
template_delta: no change to core/ - the method itself needs no edit from this run.
