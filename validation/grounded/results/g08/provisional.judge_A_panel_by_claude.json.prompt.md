You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G08 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (the orchestrator's framing was over-weighted; the top risk is not the version)
- **correct_call:** **proceed-with-changes.** Upgrade all three nodes to
  2.14.5 by a rehearsed rolling procedure (non-leaders first, meta leader
  last; drain via lame-duck `SIGUSR2`, not `systemctl restart`; a per-node
  health gate read from the meta leader's `/jsz` from another host; pin the
  version with a SHA256 check). The newsletter's other items are correctly
  ignored. The highest-severity finding is NOT the version: both live
  workloads authenticate as the full-privilege `control` user, EVENTS has no
  byte cap, three consumers have infinite redelivery, an ad-hoc stream was
  created on prod, something tried the route user on the client port, and
  route creds sit in plaintext URLs. Fix those as a separate, sequenced
  change.
- **oracle:** production rolling upgrade applied 2026-08-25 19:50-20:07 UTC
  in exactly this order; drains 15-18 s each with clients attached (lab: 2 s);
  gate green in 0-5 s per node; clients re-homed; no data or JetStream
  errors; canary round-trip OK. A rehearsal on byte-copies of the live store
  proved 2.14.2 -> 2.14.5 -> 2.14.2 loses nothing (downgrade safe). Vendor
  docs: "upgrade the non-leaders first, and the meta-leader last"; peer
  `current`/`pending` are only reported by the meta leader's `/jsz`.
- **correct fix:** as correct_call; plus replace `latest` in
  `install-nats.sh:6` with a pinned tag + SHA256; extend monitoring beyond
  store bytes (version skew, missing peer/current, leaderless stream, client
  count, redelivery growth); then the least-privilege change.
- **must_catch:**
  1. Top risk is the captured state, not the version: `live-state.md:25-30`
     both clients (the serving-host python client and the support-intake client) show
     `authorized_user=control` while `rtx`/`arm` least-privilege users exist
     (`nats-server.conf.sanitized:29-38`, `control` = publish/subscribe `>`).
  2. `live-state.md:21` EVENTS `max_bytes=-1`; `:18,20,22` `max_deliver=-1` on
     `rtx-serve`, `the eval collector`, `support-intake`; `:39` ad-hoc
     `FIX2SCRATCH` stream on prod; `:38` route-user auth error on the client
     port from a host; `nats-server.conf.sanitized:16-20` route password in
     plaintext URLs.
  3. Which fixes apply: the Raft/catch-up class (offline node rejoining:
     `release-notes-v2.14.5.md:29` #8449 and the 2.14.3/2.14.4 catch-up,
     phantom-stream, removed-peer fixes) and the 2.14.3 pre-auth panic/data
     race fixes (`release-notes-v2.14.3.md:47`) plus queue-subscription
     permission enforcement (`release-notes-v2.14.4.md:37`); NOT the TLS
     `verify_and_map`, JWT, MQTT, leafnode, encrypted-filestore items (the
     config has none of those). The orchestrator's "auth-bypass CVEs" framing
     is over-weighted; "CVEs not applicable at all" is the mirror error.
  4. Order and drain: followers first, meta leader (`live-state.md:13` =
     nats3) last; `nats.service:9-12` has no KillSignal/ExecStop so
     `systemctl restart` is SIGTERM, not lame-duck; use `kill -USR2 <pid>`
     and let `Restart=always` relaunch the new binary.
  5. Health gate must not be `is-active`: all peers current + 0 pending +
     every stream/consumer has a leader + both clients reconnected + a
     canary; polled from another host; peer status only visible via the meta
     leader's `/jsz`.
  6. No version pin: `install-nats.sh:6` fetches GitHub `latest`; no rollback
     path documented (`live-state.md:10`); rollback = reinstall the saved
     2.14.2 binary, restart, same gate.
  7. Redelivery after restart is harmless only if CT 116/CT 201 handlers are
     idempotent, which the artifact does not establish (a legitimate
     SPECULATIVE with the test: stop the consumer before ACK on a test event,
     restart, confirm one ticket).
  8. Monitoring gap: `nats-store-monitor.sh:5-9` checks store bytes only.
- **landmine (confident wrong answers):** "do not upgrade, the CVEs do not
  apply to bcrypt-only auth" (pre-auth panics and queue-permission fixes do);
  "`systemctl restart nats` one node at a time is the rolling upgrade" (no
  lame-duck drain); "downgrade is unsafe, file-store format changed" (the
  2.14 downgrade cautions are 2.14 -> 2.12 only; rehearsal proved patch-level
  downgrade safe); "the `latest`-fetching installer is fine, it is the
  official source" (unpinned).
- **unverifiable:** CT 116 handler idempotency (code not in the artifact).
- **outcome_source:** `2026-08-25-nats-spine-upgrade-review/ledger.md` (A1,
  A2, A3, A7-part, B3, B4, B5 verified; A5/B7, A6/B6 agreed; A4 surviving),
  `oracle-notes.md` O1-O6, `verdict.md`, `retro.md` T1 (outcome; verdict_held
  yes; missed_entirely "LDM drain took 15-18 s with clients attached").


=== MEMO (author unknown) ===
# Verdict - g08: upgrade the 3-node NATS JetStream spine 2.14.2 -> 2.14.5?

**ARM STATUS: INCOMPLETE.** Round 1 ran for one seat only: the other seat's three Round 1 attempts returned 0 bytes with a provider usage-limit error (retry window opens 2026-09-19 09:12). Under the protocol's missingness rule this arm counts as a seat death unless Round 1 is resumed from the unchanged packet before scoring. What follows is the mechanical fill from the ledger as it stands; buckets that depend on the dead seat's rebuttal are labelled.

Built exclusively from ledger rows. No new arguments.

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before exposure:
- The upgrade is justified by the JetStream/Raft/filestore fixes, above all v2.14.5 #8449 (offline-node catch-up from a metalayer snapshot) and the 2.14.3/2.14.4 Raft fixes; the "auth/permission CVE" items need TLS/JWT/MQTT/leafnodes this config does not enable (A1, B1; oracle-verified against the release notes and the config).
- Roll nats2 -> nats1 -> nats3, meta leader last, outside the :00 vzdump window, after saving the 2.14.2 binary and a per-node store copy; pin the installer, which currently fetches "latest" (A2, B3, A5 pin half; oracle-verified).
- Downgrade of the 2.14.5 file store and prolonged mixed-version operation are SPECULATIVE from the notes; rehearse on cloned data; roll back a failed node alone; never rewind all three stores (A5, A6, B5).
- Both live workloads authenticate as the `control` superuser; fixing that is a real defect and a separate change after the roll (A7, B7; oracle-verified).
- The ignored newsletter items are correctly ignored (A8, B8).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):** every cited release-note fix exists (A1's four line numbers off by 1-4, corrected by the seat in R1); config has no TLS/MQTT/leafnode/JWT blocks; installer unpinned; nats.service Restart=always/RestartSec=3; support-intake ack_wait=30s max_deliver=-1 pending=0; all four client connections are `control`; the store monitor defaults a missing field to 0 and only logs.

**Cross-examination-settled (`conceded` by Seat A only; Seat B dead in R1):**
- A4: redelivery is not to be called harmless; pending=0 at each stop is necessary, not sufficient for poll/event overlap (Seat A conceded toward B2).
- A3: the gate reads BOTH surviving monitor ports, halts on timeout, and separates Raft pending from consumer backlog (Seat A conceded toward B4); hard stop 300 s.
- A5/A6: cloned-R3 rehearsal before production; mixed state only for attended minutes; store restored only if the new binary opened it (Seat A conceded toward B5).

## 3. Surviving dissent (unrebutted by the dead seat; recorded as dissent, not resolved)

- **B2 vs A4 — precondition strength.** B: prove ticket idempotency and client recovery before ANY disruption. A: pending=0 per node plus the 10-min poll is the gate; a product-level interruption test is outside "simplest thing". Cheapest discriminating test: one deliberate nats1 bounce in a window with `support-intake pending=0`, then check CT 116 for a duplicate ticket and the poll/event overlap.
- **B4 vs A3 — what belongs in the gate and what ranks highest.** B: application publish/pull/ack and support processing in the gate; highest severity = restarting without proven replica recovery. A: monitor-port predicates only; highest severity = both workloads on `control`. Cheapest test: a rehearsal on cloned data where the monitor predicate passes but a controlled publish/ack fails (or never does).
- **B6 — monitor before rollout.** B: fix `.get("store",0)` and add quorum/version/lag checks first. A: EXTERNAL field name; 0% is also integer division at ~1 MB; logs-only footgun, follow-up not gate. Cheapest test: `curl :8222/jsz | python3 -c 'import sys,json; print(list(json.load(sys.stdin)))'` on one node.

## Recommendation

**Mode:** `decide-after-check` on Q3 (the three dissents above each name a cheap check) with `ship`-as-proceed-with-changes on Q1 and Q2 from bucket 1 (A1/B1, A2/B3, A5, A7/B7, A8/B8). The arm is incomplete: one seat's cross-examination is missing, so bucket 2 rests on one seat's concessions and bucket 3 is unrebutted dissent, not surviving-after-debate dissent.

## Record

- Open (unexamined, not endorsed): B8's ranking of capacity/retry hygiene (agreed with A7's ordering; nothing turned on it).
- Verified vs merely agreed: verified = A1/B1 fixes, config blocks, installer, service unit, consumer fields, control creds, monitor default; agreed-only = order (A2/B3), downgrade/mixed SPECULATIVE (A5/A6/B5), newsletter items (A8/B8).
- Rounds run / stop rule hit: R0 + R1 (Seat A only; Seat B dead: provider usage limit, 3 attempts, 0 bytes); no load-bearing flip; Round 2 not due.
- Transformations applied to relayed text: neutral Peer labels, order shuffle, ORCH-SUMMARY ledger note; none else.

VERDICT INPUT: Q1 proceed-with-changes (2.14.5; nats2->nats1->nats3; pinned checksummed binary; per-node store copy; both-survivor gate) · Q2 proceed-with-changes (`control` -> per-workload users as a separate change after the roll; newsletter extras ignored) · Q3 decide-after-check (bounce-and-check for duplicates; rehearsal on cloned data for downgrade/mixed; jsz field-name probe) · Highest-severity: dissent — both workloads on `control` (A) vs restarting without proven replica recovery (B) — INCOMPLETE ARM, one R1 seat dead.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}