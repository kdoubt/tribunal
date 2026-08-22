# Claim ledger - [decision name]

<!-- Copy this file at the start of every panel. One row per claim; update
     statuses after every round. Field semantics and the dispute rule live
     in core/LEDGER.md. The `claim` field is the seat's own sentence,
     verbatim - never your paraphrase. -->

| id | author | claim (verbatim) | evidence | provenance | confidence | falsifier | status | decision-relevant | rebuttals |
|----|--------|------------------|----------|------------|------------|-----------|--------|-------------------|-----------|
| A1 | A | "…" | file.py:42 | PANELIST-CLAIM | 0.85 | "…" | agreed-r0 | yes | - |
| A2 | A | "…" | quoted snippet | PANELIST-CLAIM | 0.6 | "…" | disputed | yes | B-r1 §1 |
| A3 | A | "…" | ASSUMPTION | PANELIST-CLAIM | 0.3 | "…" | open | no | - |
| A4 | A | (no stable position on Q2) | evidence reviewed; why undecided | - | - | cheapest discriminating test | no-stable-position | yes | - |
| B1 | B | "…" | EXTERNAL: doc URL | PANELIST-CLAIM | 0.6 | "…" | conceded | yes | A-r1 §2 |
| B2 | B | "…" | file.md:7 → check failed | - | 0.85 | "…" | dropped | - | - |
| B3 | B | "…" | file.md:9 (VERIFIED: test run) | VERIFIED | 0.9 | "…" | verified | yes | - |
| B4 | B | "…" | file.md:12 | PANELIST-CLAIM | 0.85 | "…" | overturned | yes | A-r1 §1 |
| B5 | B | "…" | file.md:20 | PANELIST-CLAIM | 0.6 | "…" | surviving-dissent | yes | A-r1 §3 |

## Oracle record

<!-- every verification run: claim id → check performed → result -->

## Stop-rule record

<!-- which stop rule ended the panel, after which round -->
