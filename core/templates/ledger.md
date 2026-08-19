# Claim ledger - [decision name]

<!-- Copy this file at the start of every panel. One row per claim; update
     statuses after every round. Field semantics and the dispute rule live
     in core/LEDGER.md. The `claim` field is the seat's own sentence,
     verbatim - never your paraphrase. -->

| id | author | claim (verbatim) | evidence | provenance | confidence | falsifier | status | decision-relevant | rebuttals |
|----|--------|------------------|----------|------------|------------|-----------|--------|-------------------|-----------|
| A1 | A | "…" | file.py:42 | PANELIST-CLAIM | high | "…" | agreed-r0 | yes | - |
| A2 | A | "…" | quoted snippet | PANELIST-CLAIM | med | "…" | disputed | yes | B-r1 §1 |
| A3 | A | "…" | ASSUMPTION | PANELIST-CLAIM | low | "…" | open | no | - |
| B1 | B | "…" | doc URL | PANELIST-CLAIM (EXTERNAL) | med | "…" | conceded | yes | A-r1 §2 |
| B2 | B | "…" | file.md:7 → check failed | - | high | "…" | dropped | - | - |
| B3 | B | "…" | file.md:9 (VERIFIED: test run) | VERIFIED | high | "…" | verified | yes | - |
| B4 | B | "…" | file.md:12 | PANELIST-CLAIM | high | "…" | overturned | yes | A-r1 §1 |
| B5 | B | "…" | file.md:20 | PANELIST-CLAIM | med | "…" | surviving-dissent | yes | A-r1 §3 |

## Oracle record

<!-- every verification run: claim id → check performed → result -->

## Stop-rule record

<!-- which stop rule ended the panel, after which round -->
