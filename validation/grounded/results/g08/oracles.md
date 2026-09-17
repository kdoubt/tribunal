# Oracles — g08 (orchestrator, non-LLM: pointer resolution + inspection)

grounding.py: grok 40/40 pointers open; codex 17/17 open (one claim, B4, carries EXTERNAL docs only).

Inspection results (file:line in ./artifact):
- v2.14.5.md:29 — "#8449 ... potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot" — supports A1/B1; B1's qualifier (trigger = idempotent stream creates) is literally in the text. VERIFIED.
- v2.14.4.md:54 removed-peer votes (A cited :56); :67 stale proposals (A cited :68); :59 filestore write race, :63 compaction race (B); :37-39 queue-permission/null-JSON/TLS verify_and_map (B). v2.14.3.md:72 Raft after write errors (A cited :76); :81 compaction corruption (A cited :80); :85 phantom streams; :67,:74,:87 (B); :50,:63 (B). All texts present; A's four line numbers are off by 1-4 inside the same section. VERIFIED (content), noted (numbering).
- nats-server.conf.sanitized:1-41 — no tls/mqtt/leafnodes/jwt/websocket/gateway blocks; control user allows ">" (:29-30); rtx/arm restricted (:31-38); routes with plaintext route password (:16-20). VERIFIED.
- live-state.md:6-8 seed = nats3; :13 meta leader nats3, last snapshot 2026-08-15; :14 JOBS leader nats3; :19 RESULTS leader nats1; :21 EVENTS leader nats1, max_bytes=-1; :22 support-intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0; :25-28 four conns, three via nats1, all authorized_user=control; :30 support user designed, never added; :38 route user on client port; :39 scratch stream; :44 :00 vzdump rule; :45 monitor checks store bytes only; :49 CT116 10-min polling fallback. VERIFIED.
- install-nats.sh:6-10 — fetches releases/latest, no pin, wildcard extraction. VERIFIED.
- nats.service:9-12 — ExecStart /usr/local/bin/nats-server, Restart=always, RestartSec=3. VERIFIED.
- nats-store-monitor.sh:5 — `json.load(sys.stdin).get("store",0)`; :6-9 only logs/tee. VERIFIED that the script defaults a missing field to 0. Whether the live jsz JSON names the field `store` or `storage` is NOT in the artifact (EXTERNAL); live-state.md:45 says the last runs show 0%, which is consistent with either a ~1 MB store or a missing field — not settled here.
- Not in the artifact (SPECULATIVE by both seats): downgrade safety of the 2.14.5 file store; mixed-version safety; client reconnect defaults; desk idempotency on redelivery.

## Addendum 2026-09-17 - pointers cited in Seat B's Round 1 (opened by the orchestrator)
- live-state.md:12-22 - the jsz section is a one-line summary plus per-stream/consumer lines (api level, msgs, leaders, consumer fields); it is summarized state, not an endpoint schema. VERIFIED (supports B's "summarized state" reading; settles nothing about the live endpoint).
- nats.service:11-12 - Restart=always / RestartSec=3. VERIFIED (already recorded).
- release-notes-v2.14.5.md:29 - "#8449 ... idempotent stream creates when an offline node catches up from a metalayer snapshot". VERIFIED (already recorded; B's qualifier is literal).
- live-state.md:25-30 - four connections, all authorized_user=control. VERIFIED (already recorded). live-state.md:22 - support-intake pending=0 is a field value; the snapshot does not define the counter's semantics. VERIFIED as B states. live-state.md:39 scratch stream; :49 CT 116 10-min polling fallback. VERIFIED (already recorded).
- nats-store-monitor.sh:5-9 - `.get("store",0)`, `[ -z "$used" ]` only catches an empty response, output is tee/echo to a log. VERIFIED that a missing field becomes 0 and that alerts only reach the log.
- No new file:line was introduced by either R1 beyond those above; nothing in the artifact settles the three remaining disputes (interruption-test necessity, halt-on-missing vs fallback, store-restore condition) - all rest on behaviour the bundle does not record.

