I'll read the frozen artifacts and peer evidence pointers so I can cross-examine each disputed claim against the brief, not against the other seats.I'll check the cited artifact lines against each peer claim so I can attack overconfidence and missing pointers, not just the weakest wording.1. **ATTACK**

**Peer-1 (0.94):** Harmlessness of redelivery is SPECULATIVE — client code is absent — but A2 does not require a CT116/CT201 interruption test before any stop. A2 asks what is eventually consistent, not a new product-test gate. `support-intake` is `pending=0 max_ack_pending=1` (`live-state.md:22`); waiting for `pending=0` before each stop closes in-flight redelivery without that test. The 10-min poll (`:49`) covers path loss, not duplicates; poll/event overlap is a residual **nats1** bounce risk, not a 0.94 blocker. Reconnect/backoff/jitter/alerts are not in the artifact; demanding them as pre-roll work violates “simplest thing.” 0.94 is uncalibrated for a SPECULATIVE conclusion.

**Peer-2 (0.93):** Core (mixed/downgrade unattended = no) is sound. “Pause only after claim 4 passes” has no artifact pointer. Always-restore of `js` when 2.14.5 never opened the store is stricter than needed. Rehearsal-as-hard-gate is argued, not a vendor requirement. Confidence is acceptable for “do not assume.”

**Peer-3 (0.87):** Systemd ≠ health is correct. `/raftz`, `/healthz?js-enabled-only=true`, and the long `jsz` query string are EXTERNAL; missing keys must not fail the gate. “Controlled publish/pull/ack and support processing” couples the binary roll to CT116/ClickUp. “No automatic restart loop” contradicts `nats.service:11-12` `Restart=always`; A4 does not change the unit. 5 min vs 120s: both ASSUMPTION; 1 MB catch-up is not the slow path — client reconnect on nats1 is.

**Peer-4 (0.98):** Worst-calibrated claim. Whether `/jsz` uses `store` or `storage` is EXTERNAL (not in the snapshot). `live-state.md:45` “0%” is also explained by integer `used*100/LIMIT` at ~1 MB / 15 GB (`nats-store-monitor.sh:7`) even if `store` is correct. Script only logs (`:6–9`); no tested pager. This cannot lose a desk ticket and is not a pre-roll gate. A 30s quorum/version/lag monitor is a new project, not a 2.14.5 prerequisite.

2. **CONCEDE**

Redelivery/idempotency of CT116 is SPECULATIVE; `pending=0` is a necessary per-node gate, not sufficient for poll-overlap. Mixed 2.14.2/2.14.5 and 2.14.5→2.14.2 without restore stay SPECULATIVE: attended minutes only; rollback **that node**; never rewind all three stores; cloned-data rehearsal before production. A1/A3 from **both** survivors, not `is-active`; halt on timeout; distinguish Raft pending vs consumer backlog. Monitor `.get("store",0)` is a logs-only footgun — follow-up, not a roll gate. Settled: nats2→nats1→nats3, pin installer, do not bundle `support`.

3. **REVISE**

- **Own-1:** Claim text unchanged. Evidence lines: Raft write-error candidacy `v2.14.3.md:72` (not :76); filestore compaction `:81` (not :80). Confidence 0.88 unchanged.
- **Own-3:** Gate from **both** surviving `:8222`; keep predicates 1–4 and 7; raft keys still SPECULATIVE-if-absent; 2s poll, 3 consecutive, **300s** hard stop; no `/raftz` or app publish/ack. Confidence 0.82→0.80.
- **Own-4:** Do **not** call redelivery harmless. With `pending=0` at each stop, `support-intake` has nothing to redeliver; residual duplicate risk is nats1 bounce vs poll overlap (SPECULATIVE, not a roll blocker). Confidence 0.78→0.74.
- **Own-5:** Run the cloned-R3 discriminating test **before** production; pin remains. Confidence 0.70→0.76 (pin 0.95; downgrade 0.45 SPECULATIVE).
- **Own-6:** Mixed only for attended minutes; interrupted 1/3 → hand off, do not soak; failed-start: stop that node, restore saved 2.14.2; restore `js` tarball iff the new binary opened the store. Confidence 0.62→0.68.
- Own-2, Own-7, Own-8: no revision (monitor stays follow-up).

4. **VERDICT INPUT**

Q1 proceed-with-changes; Q2 proceed-with-changes (`control` after roll; newsletter extras ignored); Q3 proceed-with-changes. Highest-severity: CT116+CT201 as `control` (`">"`), then unproven mixed/downgrade.
