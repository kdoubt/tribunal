# Verdict - g08: upgrade the 3-node NATS JetStream spine 2.14.2 -> 2.14.5?

Built exclusively from ledger rows. No new arguments. Round 0 + Round 1 (both seats; Seat B's Round 1 resumed from the unchanged packet on 2026-09-17 after three provider-limit failures on 2026-09-16).

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before exposure:
- The upgrade is justified by the JetStream/Raft/filestore fixes, above all v2.14.5 #8449 (offline-node catch-up from a metalayer snapshot) and the 2.14.3/2.14.4 Raft fixes; the "auth/permission CVE" items need TLS/JWT/MQTT/leafnodes this config does not enable (A1, B1; oracle-verified against the release notes and the config).
- Roll nats2 -> nats1 -> nats3, meta leader last, outside the :00 vzdump window, after saving the 2.14.2 binary and a per-node store copy; pin the installer, which currently fetches "latest" (A2, B3, A5 pin half; oracle-verified).
- Downgrade of the 2.14.5 file store and prolonged mixed-version operation are SPECULATIVE from the notes; rehearse on cloned data; roll back a failed node alone; never rewind all three stores (A5, A6, B5).
- Both live workloads authenticate as the `control` superuser; fixing that is a real defect and a separate change after the roll (A7, B7; oracle-verified).
- The ignored newsletter items are correctly ignored (A8, B8).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):** every cited release-note fix exists (A1's four line numbers off by 1-4, corrected by the seat in R1); config has no TLS/MQTT/leafnode/JWT blocks; installer unpinned; nats.service Restart=always/RestartSec=3; support-intake ack_wait=30s max_deliver=-1 pending=0 (the snapshot does not define the counter's semantics); all four client connections are `control`; the store monitor defaults a missing field to 0 and only logs; live-state.md:12-22 is summarized state, not an endpoint schema. Nothing in the artifact settles the three disputes in bucket 3.

**Cross-examination-settled (`conceded` / `revised`):**
- A4 (by A toward B2): redelivery is not to be called harmless; pending=0 at each stop is necessary, not sufficient (R1_grok, REVISE).
- A3 / B4 (both ways): the gate reads BOTH surviving monitor ports, halts on timeout, separates Raft pending from consumer backlog, hard stop 300 s (A conceded); missing currentness, membership, pending-work or consumer-leadership evidence blocks the next restart and clients are verified across the cluster, while controlled publish/pull/ack is no longer in B's gate text (B revised, 0.87 -> 0.80).
- A5 / B5 (both ways): cloned-R3 rehearsal before production (A revised 0.70 -> 0.76; B conceded); single-node rollback, never rewind all three stores, explicit uncertainty about prolonged mixed versions (A conceded; B conceded).
- B6 (by B): the claim that the live jsz field must be `storage` is withdrawn; what stands is that the monitor turns a missing field into 0 and only logs, so capture live /jsz to establish the field, reject absent metrics, test the alert path (B revised 0.98 -> 0.95). A had conceded the logs-only footgun.
- B7 / A7 (by B): credential rollback is independent of binary rollback; separating them stays sensible (R1_codex, ATTACK Peer-4).

## 3. Surviving dissent

- **B2 vs A4 - precondition strength.** B: run an interruption test covering ticket creation before ack and polling overlap before ANY disruption. A: pending=0 per node plus the 10-min poll is the gate; the residual duplicate risk is speculative and not a roll blocker. Cheapest discriminating test: one deliberate nats1 bounce in a window with `support-intake pending=0`, then check CT 116 for a duplicate ticket and the poll/event overlap.
- **B4 vs A3 - halt-on-missing vs fallback, and what ranks highest.** B: unavailable replication or consumer-leader evidence must halt progression; highest severity = advancing without proof the restored replica is current. A: fall back to predicates 1-4 and 7 when raft fields are absent; highest severity = both workloads on `control`. Cheapest test: a cloned-data rehearsal in which the raft fields are withheld and the gate must halt rather than advance.
- **A6 vs B5 - the store-restore condition on a failed start.** A: restore the js tarball only if the new binary opened the store. B: stop systemd explicitly before any replace, and require positive evidence the store was untouched or use the rehearsed cold-copy recovery. Cheapest test: in the rehearsal, start a deliberately broken 2.14.5 binary and inspect the store's mtime/contents before and after.
- **B6 - monitor before rollout.** B: fix the monitor (field capture, reject absent metrics, delivered alerts) before the roll. A: logs-only footgun, follow-up not gate. Cheapest test: `curl :8222/jsz | python3 -c 'import sys,json; print(list(json.load(sys.stdin)))'` on one node.

## Recommendation

**Mode:** `decide-after-check` on Q3, with `ship`-as-proceed-with-changes on Q1 and Q2 from bucket 1 (A1/B1, A2/B3, A5, A7/B7, A8/B8) and bucket 2 (A3/B4 gate shape, A5/B5 rehearsal and rollback scope, B6 monitor fix). The checks that gate Q3 are the four named in bucket 3: the bounce-and-duplicate check (B2/A4), the withheld-raft-fields rehearsal (B4/A3), the broken-binary store inspection (A6/B5), and the jsz field probe (B6). Both seats condition the roll on executable gates plus a rehearsal; they differ on whether an application-level interruption test must precede any disruption and on which of two defects ranks highest. The brief pre-delegated no tie-break; the ranking question goes to the owner.

## Record

- Open (unexamined, not endorsed): B8's ranking of capacity/retry hygiene (agreed with A7's ordering; nothing turned on it).
- Verified vs merely agreed: verified = A1/B1 fixes, config blocks, installer, service unit, consumer fields, control creds, monitor default, jsz summary; agreed-only = order (A2/B3), downgrade/mixed SPECULATIVE (A5/A6/B5), newsletter items (A8/B8).
- Rounds run / stop rule hit: R0 + R1 (both seats; Seat B resumed on 2026-09-17 from the unchanged packet after three zero-byte provider-limit attempts); no load-bearing flip; Seat B's Q3 wording moved from "proceed-with-changes contingent on gates + rehearsal" to "do-not-upgrade-yet until the checks and tests exist", recorded as the same condition relabelled; Round 2 not triggered.
- Transformations applied to relayed text: neutral Peer labels, order shuffle, ORCH-SUMMARY ledger note; none else.

VERDICT INPUT: Q1 proceed-with-changes (2.14.5; nats2->nats1->nats3; pinned checksummed binary; per-node store copy; both-survivor fail-closed gate) · Q2 proceed-with-changes (monitor field capture + delivered alerts; `control` -> per-workload users as a separate change after the roll; newsletter extras ignored) · Q3 decide-after-check (bounce-and-duplicate check; withheld-raft-fields rehearsal; broken-binary store inspection; jsz field probe; interruption-test-before-any-disruption is surviving dissent) · Highest-severity: dissent - both workloads on `control` (A) vs advancing to another restart without proof the restored replica is current (B) - owner's call.
