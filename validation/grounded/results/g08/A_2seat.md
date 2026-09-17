A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== Memo 1 ===

I'll read the artifacts first and ground every claim in the files before taking a position on the upgrade.**Framings considered:** (1) CVE rush — land “auth CVEs” immediately; (2) R3 Raft/filestore integrity patch whose hard part is a gated binary rollout; (3) freeze because the desk is LIVE; (4) hygiene-first (fix `control` before the binary). **Picked (2):** this config has no TLS/JWT/MQTT/leafnodes, so most security-shaped notes do not apply, while R3 + metalayer snapshots + file store do; criteria still require the patches that apply and a procedure that cannot drop quorum.

---

CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
EVIDENCE: Applies: `release-notes-v2.14.5.md:29` “data loss when … an offline node catches up from a metalayer snapshot”; `v2.14.4.md:56` “Raft elections now correctly ignore votes from removed peers”; `v2.14.4.md:68` “stale proposals from a previous term”; `v2.14.3.md:76` “Raft nodes no longer participate in voting or candidacy after write errors”; `v2.14.3.md:85` “Meta recovery snapshots no longer leave phantom streams”; `v2.14.3.md:80` filestore compaction corruption; `live-state.md:15-21` `R3 storage=file` + `meta leader nats3` + `last meta snapshot 2026-08-15T20:43:08`. Does not apply: `v2.14.4.md:39` TLS `verify_and_map` blank-password bypass; `v2.14.3.md:53-59` MQTT; `v2.14.3.md:41-42` JWT claims; `v2.14.3.md:49` leaf `Nats-Trace-Dest`; `nats-server.conf.sanitized` has no `tls`/`mqtt`/`leafnodes`/`jwt`/`websocket`/`gateway` blocks (file is 42 lines). Notes credit “CVE advisory notices” (`v2.14.3.md:91`) but list no CVE IDs.
CONFIDENCE: 0.88
FALSIFIER: A 2.14.3/2.14.4 advisory that assigns a CVE to bcrypt-user / clustered-core auth on a non-TLS, non-MQTT, non-JWT server; or `nats-server.conf` on nats2/nats3 containing those blocks.

CLAIM: Roll followers first, meta leader last: nats2 → nats1 → nats3; freeze stream/consumer creates; avoid PVE `:00` `vzdump`; copy the 2.14.2 binary aside and tar `/var/lib/nats/js` on the node before replacing it.
EVIDENCE: `live-state.md:7-8,13-21,28` nats3 is seed + meta leader + JOBS leader; nats1 leads RESULTS+EVENTS (desk stream) and holds 3/4 client conns; nats2 leads nothing and has one `hale-support-events` conn. `live-state.md:44` “avoid the :00 windows”. `live-state.md:10` store “~0.5–1.1 MB”. June HA “node rejoined” is the #8449 path — catch-up must run on the **new** binary, so replace-then-start, and do not create streams mid-roll (`live-state.md:39` already did that once on prod).
CONFIDENCE: 0.9
FALSIFIER: At cutover, `/jsz` shows nats2 as meta or EVENTS leader, or a vzdump task running against CT 121–123.

CLAIM: `systemctl is-active` is not “back and current”; gate the next node from a **surviving** `:8222` with a 2s poll, 3 consecutive passes, 120s hard stop.
EVIDENCE: Precedent “Active is not healthy”; `nats.service:11-12` `Restart=always` / `RestartSec=3` so a crash loop still looks active. Predicate from a node that was **not** just restarted: (1) `curl -sf --max-time 3 http://<restarted>:8222/healthz` → HTTP 200 `"status":"ok"`; (2) restarted `/varz` contains version `2.14.5` and JetStream enabled; (3) survivor `/routez` includes the restarted IP and 2 routes; (4) survivor `/jsz` has a meta leader and JOBS/RESULTS/EVENTS each have a `leader` (same fields as `live-state.md:13-21`); (5) `GET /jsz?raft=true` on a survivor — if present, every replica `current` and `pending==0` (exact keys not in the snapshot: treat missing keys as “use 1–4 only”, labeled SPECULATIVE); (6) restarted `journalctl -u nats` shows listen on 4222/6222 and no repeating `[ERR]`; (7) `connz?auth=true` shows the clients that were on that node (`live-state.md:26-28`). Timeout 120s is ASSUMPTION given ~1 MB store (`live-state.md:10`).
CONFIDENCE: 0.82
FALSIFIER: A node with `is-active=active` and `/healthz=ok` while survivor `/jsz` shows a stream with empty leader or a replica not `current`.

CLAIM: After each restart, Raft catch-up, possible stream/meta leader moves, and ack-wait redelivery are expected; with 4 connections this is not a reconnect storm; duplicate desk events are possible because `support-intake` has `max_deliver=-1`, while loss of the NATS path is covered by the 10-minute poll.
EVIDENCE: `live-state.md:22` `ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0`; `live-state.md:25-28` four conns, three on nats1; `live-state.md:49` “falls back to 10-min polling”. Restart nats2 first so the desk’s EVENTS leader on nats1 stays up. Clients `python3 lib=2.15.0` / `nats.js lib=2.29.3` reconnect by default — ASSUMPTION (library default, not in artifact). Harmless iff pending=0 at each stop and the desk is idempotent on redelivery; criterion (1) forbids silent duplicates — verify pending=0 on `support-intake` before signaling the next node.
CONFIDENCE: 0.78
FALSIFIER: After nats1 restart, `connz` never again shows `hale-support` / `.13` python clients, or `support-intake redelivered` increases and CT 116 creates a duplicate ticket.

CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
EVIDENCE: `install-nats.sh:6-10` `releases/latest` → `install -m755 … /usr/local/bin/nats-server` with no pin; `live-state.md:10` “no documented rollback path”. Notes only point at a 2.12.x upgrade guide (`v2.14.5.md:5`); 2.14.3–2.14.4 change filestore compaction/key-files (`v2.14.3.md:80`, `v2.14.4.md:56-59`) but never say “on-disk format bumped” or “downgrade-safe”. Discriminating test: scratch R3, write known seqs, run 2.14.5, write more, start 2.14.2 against that store **without** restore — if 2.14.2 exits or seqs diverge vs the other replicas, downgrade-without-restore is unsafe.
CONFIDENCE: 0.7 (pin: 0.95; downgrade: 0.45 SPECULATIVE)
FALSIFIER: `install-nats.sh` already taking a version argument; or that scratch test loading 2.14.5 blocks on 2.14.2 with matching seqs.

CLAIM: A mixed 2.14.2/2.14.5 cluster after 1 of 3 is acceptable for minutes (the intended roll), not for hours/days; if the new binary fails to start, roll back **that node only** and do not touch the other two.
EVIDENCE: Notes never state mixed-patch safety (SPECULATIVE). #8449 (`v2.14.5.md:29`) remains on any remaining 2.14.2 node that later catches up from a metalayer snapshot (`live-state.md:13` last snapshot 2026-08-15). Exact rollback: confirm survivors still have meta+stream leaders via their `:8222`; `install -m755` the saved `nats-server.v2.14.2`; if 2.14.5 never opened JS (no listen / immediate exit), start 2.14.2; if it opened JS, restore the pre-step `/var/lib/nats/js` tarball then start 2.14.2 and wait for the A1 predicate. `nats.service:8-9` still points at `/usr/local/bin/nats-server`.
CONFIDENCE: 0.62
FALSIFIER: Official 2.14 notes/advisory that mixed 2.14.2+2.14.5 is unsupported, or a mixed-cluster probe where publish/pull/ack fails while 2/3 are up.

CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
EVIDENCE: `live-state.md:26-30` `authorized_user=control` for `.13` and `.16`; `nats-server.conf.sanitized:29-30` `control` allows `">"`; `rtx`/`arm` exist (`:31-38`) but are unused; “`support` user was DESIGNED … but never added” (`live-state.md:30`). Bundling couples a HUP+credential push (`nats.service:10`) to a binary replace, so a bad binary rollback also requires client credential rollback. Rank next: `support-intake` `max_deliver=-1` (`live-state.md:22`) can duplicate tickets on redelivery; EVENTS `max_bytes=-1` (`live-state.md:21,33`) with only the 15 GB `max_file_store` backstop (`nats-server.conf.sanitized:10`); `nats-store-monitor.sh:2-9` checks only store bytes; route user tried on client port 4222 (`live-state.md:38`); dead `rtx-serve` (`live-state.md:35`).
CONFIDENCE: 0.86
FALSIFIER: `connz?auth=true` showing CT 116 on a restricted user, or a `support` user already in the live conf.

CLAIM: Newsletter items the owner ignored (OpenTelemetry collector, agent-SDK session store, “coordinating AI agents”, file-vs-memory KV mirror) are correctly ignored; seed designation, credentials, and client reconnect config must not change in this change-set.
EVIDENCE: USER-FACT newsletter list (newsletter file not in `./artifact/`). `live-state.md:15-22` only JOBS/RESULTS/EVENTS — no KV, no OTel, no agent SDK. A4 must-change: binary on three nodes; `install-nats.sh:6` pin; keep `nats-server.v2.14.2`; record version next to `live-state.md:6` inventory. Must-not: `cluster.routes` / nats3 seed (`nats-server.conf.sanitized:16-20`, `live-state.md:8`); passwords; client libs. Optional follow-up: extend `nats-store-monitor.sh` to `/healthz` + version + meta leader.
CONFIDENCE: 0.8
FALSIFIER: A live KV bucket or OTel exporter on this spine, or a newsletter item that documents a 2.14.2 data-loss requiring more than the binary bump.

**VERDICT INPUT**
- Q1: proceed-with-changes (upgrade to 2.14.5; nats2→nats1→nats3; pin installer; do not treat auth CVEs as the reason).
- Q2: proceed-with-changes (fix `control` after the roll; ignore OTel/agent-SDK/KV-mirror).
- Q3: proceed-with-changes (A1 surviving `:8222` gate; A5 no mixed-for-days; rollback = old binary + `js` tarball).
- Highest-severity: CT 116 and CT 201 both use the `control` superuser on the LIVE desk bus.


=== Memo 2 ===

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
