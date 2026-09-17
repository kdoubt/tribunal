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


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}