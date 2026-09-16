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
