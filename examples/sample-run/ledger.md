# Claim ledger - methodology-design panel (historical)

Abbreviated to decision-relevant claims. A = Codex CLI, B = Grok CLI.
This run predates `core/LEDGER.md`'s finalized field list and status enum;
kept as a period artifact, not rewritten. Status mapping to the current
enum: `agreed` → `conceded`; `disputed → overturned` → `overturned`;
`disputed → adopted-with-modification` → `conceded` (with the revision
recorded in the seat's R1); all others match. The compliant reference is
`core/templates/ledger.md`.

| id | author | claim (compressed here as ORCH-SUMMARY; verbatim text in the r0 files) | status |
|---|---|---|---|
| A1 | A | Panel = structured dispute; value from independent error processes + explicit adjudication | agreed-r0 (B: "claim-stress test, not a salon") |
| A2 | A | 3 rounds default (independent / rebuttal / revision) + optional adjudication round | disputed → overturned (B's R1 attack "ceremony priced as rigor"; A's schema point folded into R1 structure) |
| A3 | A | Claim ledger outside panelist prose; context = contested evidence, not memory | agreed (B conceded R1: "must ship") |
| A4 | A | Four-way provenance tags; repetition never becomes evidence | agreed (B conceded R1) |
| A5 | A | Blinded relay legitimate where prestige bias risks judgment | surviving-dissent (B: style deanonymizes in 2-seat; bias-discounting needs attribution) |
| A6 | A | If evidence doesn't resolve, orchestrator chooses via explicit decision criteria | surviving-dissent (B: authority laundering; adopted only as pre-delegation exception) |
| B1 | B | Default 2 rounds, cap 3, never 4; consensus is a failure mode not a goal | disputed → adopted-with-modification (verification steps don't count as rounds - B revised) |
| B2 | B | Isolated parallel R0 is the only uncontaminated signal; stop if R0 agrees | agreed-r0 (A: "do not expose another panelist's answer yet") |
| B3 | B | Attributed relay always; never blind | surviving-dissent (see A5) |
| B4 | B | Oracles before debate; repo-checkable claims preferred | agreed-r0 (A: "use that oracle before convening a debate") |
| B5 | B | Orchestrator = switchboard not chair; never picks a winner | surviving-dissent (see A6) |
| B6 | B | "If wrong answer costs >~15 min, panel" | overturned (A's R1: fake precision; replaced by expected-loss factors) |

Stop rule hit: R1 added no new claim IDs; remaining deltas adjudicated as
surviving dissent.
