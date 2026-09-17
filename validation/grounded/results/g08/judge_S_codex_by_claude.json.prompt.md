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
Plausible framings are patch/security applicability, preservation of ticket processing, and operational readiness. I choose operational readiness: the captured monitor checks capacity alone (`artifact/live-state.md:45`), so installing the patch is only part of the decision. All paths below are under `artifact/`. Prescriptions and risk rankings are proposed judgments.

**CLAIM 1:** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.  
**EVIDENCE:** `release-notes-v2.14.3.md:67,72,74,87` fixes shutdown races, voting after write errors, post-snapshot commit indexing, and membership rollback; `release-notes-v2.14.4.md:59,63,67` fixes filestore races and stale-term proposals; `release-notes-v2.14.5.md:29` specifically requires **idempotent stream creates** during offline-node metalayer catch-up. R3 file streams: `live-state.md:14,19,21`. JWT, TLS, MQTT, leafnode and no-auth fixes have no demonstrated configured trigger (`nats-server.conf.sanitized:1–41`); queue-permission fixes are conditional, whereas JetStream null-JSON and monitoring-input guards remain relevant (`release-notes-v2.14.4.md:37–39`; `release-notes-v2.14.3.md:50,63`).  
**CONFIDENCE:** 0.93  
**FALSIFIER:** Reproduction shows these clustered fixes cannot affect this configuration, or additional configuration establishes the excluded authentication paths.

**CLAIM 2:** A2 requires proving ticket idempotency and client recovery before disruption, because redelivery cannot be assumed harmless.  
**EVIDENCE:** `live-state.md:22,25–30,49–50` identifies support’s 30-second ack wait, three connections on nats1, one on nats2, and polling fallback; EXTERNAL: [acknowledgment semantics](https://docs.nats.io/learn/jetstream/acknowledgment). Client implementation is unavailable: harmlessness is **SPECULATIVE**. Proposed discriminating test: interrupt after ticket creation but before acknowledgment, overlap polling and event delivery, and require exactly one ticket per source ID plus eventual processing of every accepted event. Test CT201 side effects similarly. Raft catch-up, elections, reconnection and ack-state convergence take time; affected connections move at each restart, potentially repeatedly. Verify all-three-server reachability, reconnect/backoff/jitter settings, durable resubscription and disconnect alerts; library versions alone establish none of these.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** Existing implementation and interruption tests establish these guarantees already.

**CLAIM 3:** The proposed rollout is nats2 → nats1 → nats3, re-evaluated against live leadership, with pinned binaries and a cold per-node recovery copy.  
**EVIDENCE:** `live-state.md:13–21` places metadata/JOBS leadership on nats3 and EVENTS/RESULTS on nats1; EXTERNAL: [rolling-upgrade guidance](https://docs.nats.io/learn/deployment/rolling-upgrades). `install-nats.sh:6–10` selects latest and wildcard extraction; `nats.service:9–12` specifies the binary and automatic restart. Proposed procedure: check PVE tasks and avoid backup overlap (`live-state.md:44`); pass claim 4 before each stop; save the original binary/config; stage the exact v2.14.5 architecture tarball, verify published checksum, and run its `-t` against the real config. Through host `pct exec`, explicitly stop `nats`, verify termination, back up that node’s store preserving ownership, `install -m755` the exact staged binary, start, then pass claim 4. Update all three binaries, installer pin/checksum, inventory and runbook; preserve identities/routes and document nats3’s seed designation separately from elected leadership (`nats-server.conf.sanitized:2,17–20`).  
**CONFIDENCE:** 0.88  
**FALSIFIER:** Fresh leadership or rehearsal results require another order or reveal incompatible configuration.

**CLAIM 4:** A1/A3 should gate every subsequent stop on independently observed replication and application health, never systemd activity alone.  
**EVIDENCE:** Proposed probes, from a PVE host against all three addresses, including both surviving nodes: `/varz`, `/healthz?js-enabled-only=true`, `/routez`, `/jsz?accounts=true&streams=true&consumers=true&raft=true`, `/raftz`, and `/connz?auth=true&subs=true`; EXTERNAL: [monitoring endpoints](https://docs.nats.io/learn/monitoring/monitoring-endpoints), [Raftz](https://docs.nats.io/reference/system/monitor/raftz). Proposed predicate: expected version/identity; successful health response; both distinct route peers; metadata and every stream/consumer leader present; expected membership, all followers current/online with zero lag; zero pending Raft work and committed/applied convergence; expected clients/subscriptions restored; controlled publish/pull/ack and support processing succeed. Distinguish Raft pending from consumer backlog. Poll every five seconds with three-second request deadlines; require three consecutive passes within five minutes. Retry reads until deadline; thereafter halt and diagnose, with no automatic restart loop or next-node action.  
**CONFIDENCE:** 0.87  
**FALSIFIER:** A rehearsal passes this predicate while a required replica or support client remains unavailable.

**CLAIM 5:** A5’s prolonged mixed-version safety and direct file-store downgrade remain **SPECULATIVE**, so neither should be an unattended assumption.  
**EVIDENCE:** The compatibility pointers concern 2.12.x (`release-notes-v2.14.3.md:4`, `release-notes-v2.14.4.md:4`, `release-notes-v2.14.5.md:4`); none supplies the requested downgrade guarantee. Discriminating rehearsal: cloned R3 data, mixed-version soak, elections, offline snapshot catch-up/idempotent creates, writes/acks, then downgrade and verify payloads, sequences and consumer state. Proposed interruption rule: pause only after claim 4 passes, record versions, retain alerting and hand over; do not plan days mixed. Failed-start rollback: explicitly stop `nats`, quarantine the possibly modified store, restore that node’s pre-upgrade cold store/config and saved 2.14.2 binary at `/usr/local/bin/nats-server`, preserve ownership, start and repeat claim 4—only after rehearsing this recovery; never rewind all three stores or disturb the surviving quorum.  
**CONFIDENCE:** 0.93  
**FALSIFIER:** Version-specific guarantees and successful representative recovery tests establish compatibility.

**CLAIM 6:** Monitoring needs correction before rollout because its missing-field default can report zero usage.  
**EVIDENCE:** `nats-store-monitor.sh:5` uses `.get("store",0)`; EXTERNAL [JSz schema](https://docs.nats.io/reference/system/monitor/jsz) names `storage`. Lines 6–9 only log warnings/threshold messages. Proposed changes: require valid `storage` and configured capacity, reject missing fields/HTTP errors, deliver tested alerts, and monitor quorum/currentness, leadership, version drift, client absence and lag independently every 30 seconds.  
**CONFIDENCE:** 0.98  
**FALSIFIER:** Captured responses contain a valid `store` field and a tested external alert path covers these failures.

**CLAIM 7:** After immediate continuity safeguards, shared unrestricted credentials are the highest-priority security correction and should remain a separate change.  
**EVIDENCE:** `live-state.md:25–30`; `nats-server.conf.sanitized:29–38`. Proposed sequence: add workload-specific permissions on all nodes, validate/reload individually, deliver credentials to CT116/201, verify actual authorized users and API/ack access, then retire shared application credentials. Investigate route-user misuse and production scratch creation (`live-state.md:38–39`); neither proves compromise.  
**CONFIDENCE:** 0.96  
**FALSIFIER:** Fresh connection evidence shows workloads already use appropriately restricted credentials.

**CLAIM 8:** Capacity/retry hygiene ranks below continuity and credential exposure, while the other newsletter products can presently be deferred.  
**EVIDENCE:** `live-state.md:21–22,33–35`: unlimited EVENTS bytes, infinite retries and an unused demo consumer. Propose an explicit EVENTS budget with retention/backpressure safeguards; poison-message alerting, quarantine and replay before limiting retries; confirm ownership before retiring `rtx-serve`. **ASSUMPTION:** OTel, agent sessions/orchestration and KV-mirror changes lack a demonstrated requirement; actual newsletter bodies are unavailable.  
**CONFIDENCE:** 0.85  
**FALSIFIER:** Growth, poison-message incidents or an active integration establishes immediate need.

**VERDICT INPUT**

Q1: **proceed-with-changes**.  
Q2: **proceed-with-changes**.  
Q3: **proceed-with-changes**, contingent on gates and recovery rehearsal.  
Highest severity: proceeding to another restart without independently proving replica recovery, risking quorum loss (claim 4).


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}