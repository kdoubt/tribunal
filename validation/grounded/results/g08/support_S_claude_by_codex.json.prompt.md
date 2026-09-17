You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.4:39
OPENED TEXT:
- Several JetStream and MQTT endpoints now correctly guard against null values in JSON
- Fixed an authentication bypass with TLS `verify_and_map` authenticating users with blank passwords

Monitoring

[2] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.3:55-59
OPENED TEXT:
- Partial `CONNECT` packets can no longer exhaust pre-authentication memory
- `PUBLISH` remaining-length underflow no longer causes a server panic
- Subscriptions to internal `$MQTT.deliver.pubrel` subjects are now rejected
- Subscribe deny rules are now enforced on retained message and QoS replay paths
- WebSocket `/mqtt` upgrades no longer panic when MQTT is disabled

Monitoring

[3] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.4:71-76
OPENED TEXT:
MQTT

- Packet identifiers for QoS1 and QoS2 are now issued by a monotonic counter, avoiding accidental ID reuse (#8358, thanks to @nberlee)
- Pending QoS1 and QoS2 deliveries should no longer leak when the subscription is downgraded to QoS0 (#8359, thanks to @nberlee)
- QoS2 messages released on a resumed sessions should no longer lose their QoS or packet ID (#8414)
- MQTT clients can no longer subscribe to `$MQTT.>` subjects, closing a potential permission bypass

### Complete Changes

[4] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.3:48-49
OPENED TEXT:
- Several panic, fatal and data race conditions in authentication, routing, monitoring and clustered request handling have been fixed
- `NoAuthUser` now checks connection restrictions
- Leaf connections no longer bypass `Nats-Trace-Dest` publish permission checks
- `CONNZ` and `SUBSZ` pagination now guard against `Offset` and `Limit` integer overflow panics
- Fixed a nil pointer panic when starting up when the resolver parent directory is missing (#8329)

[5] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.3:40-41
OPENED TEXT:
- Long-running reconnect and OCSP loops no longer retain unused timers, reducing memory pressure over time (#8204)
- Inherited JWT default permissions are now refreshed when account claims are updated (#8276)
- External auth configuration is now cleared correctly when account claims are updated (#8275)
- PROXY protocol detection, TLS sniffing with `allow_non_tls` and PROXY v1 address-family parsing have been fixed (#8302)
- A race in gateway `CONNECT` handling has been fixed (#8306)
- Trusted proxy tracking no longer leaks closed clients during concurrent updates (#8307)

[6] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.4:36
OPENED TEXT:
- Combining `no_auth_user` with auth callouts will no longer skip authentication checks when no `CONNECT` message is sent
- JWT validation no longer crashes the server with whitespace-only permissions
- Several paths that enforce the permissions of queue subscriptions no longer treat the whole permission as a subject literal
- Several JetStream and MQTT endpoints now correctly guard against null values in JSON
- Fixed an authentication bypass with TLS `verify_and_map` authenticating users with blank passwords

[7] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.3:72
OPENED TEXT:
- Zero consumer limits are now treated as unlimited during stream updates (#8286)
- Raft nodes no longer participate in voting or candidacy after write errors (#8290)
- Raft checkpoint handling now aborts if the node is closed (#8296)
- Raft `ApplyCommit` now handles the post-snapshot index correctly (#8321)
- Consumer ack subscriptions now match correctly when consumer names contain `%` (#8301)

[8] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.3:74
OPENED TEXT:
- Raft checkpoint handling now aborts if the node is closed (#8296)
- Raft `ApplyCommit` now handles the post-snapshot index correctly (#8321)
- Consumer ack subscriptions now match correctly when consumer names contain `%` (#8301)
- Observer state is now cleared correctly during `js_cluster_migrate` when a leaf remote is removed (#8304)
- Atomic batch end-of-batch max-size checks and R1 message rewrites have been fixed (#8305)

[9] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.4:54
OPENED TEXT:
- Oversized cluster replicated messages are now validated correctly on 32-bit systems (#8357, thanks to @uwezkhan)
- Raft elections now correctly ignore votes from removed peers (#8353)
- Filestore blocks with unsynced or truncated key files are now removed and counted as lost data instead of failing to recover altogether (#8365)
- Filestore encryption key files are now synced to disk more aggressively (#8366)
- Raft now handles the append entry iterator returning no more entries correctly (#8372)

[10] CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
POINTER: v2.14.4:67
OPENED TEXT:
- Creating a clustered consumer immediately after creating a clustered stream should no longer respond with a `stream not found` error (#8410)
- Raft proposals now require the term to be passed down from JetStream, preventing situations where stale proposals from a previous term could make changes in a new term after a fast election (#8370)
- Replicated streams that were recreated while a node was down are no longer treated as an update by a returning node processing a snapshot, avoiding stale Raft groups from continuing to run and unexpected behaviour with consumers (#8413)
- Stream snapshot endpoints now more strictly check the reply subject for validity

[11] CLAIM: The single strongest reason to go to 2.14.5 is that the rolling upgrade itself is the exact scenario the #8449 data-loss fix covers, and one #8413 hazard has already occurred on this spine.
POINTER: v2.14.5:29
OPENED TEXT:
- Fix a bug that could result in potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot, caused by an incorrect update to the create time in the stream assignment (#8449)

### Complete Changes

[12] CLAIM: The single strongest reason to go to 2.14.5 is that the rolling upgrade itself is the exact scenario the #8449 data-loss fix covers, and one #8413 hazard has already occurred on this spine.
POINTER: live-state.md:13
OPENED TEXT:
## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0

[13] CLAIM: The single strongest reason to go to 2.14.5 is that the rolling upgrade itself is the exact scenario the #8449 data-loss fix covers, and one #8413 hazard has already occurred on this spine.
POINTER: v2.14.4:68
OPENED TEXT:
- Raft proposals now require the term to be passed down from JetStream, preventing situations where stale proposals from a previous term could make changes in a new term after a fast election (#8370)
- Replicated streams that were recreated while a node was down are no longer treated as an update by a returning node processing a snapshot, avoiding stale Raft groups from continuing to run and unexpected behaviour with consumers (#8413)
- Stream snapshot endpoints now more strictly check the reply subject for validity

MQTT

[14] CLAIM: The single strongest reason to go to 2.14.5 is that the rolling upgrade itself is the exact scenario the #8449 data-loss fix covers, and one #8413 hazard has already occurred on this spine.
POINTER: live-state.md:39
OPENED TEXT:
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).
- 2026-08-15 20:35–20:42: stream `ESTATE > FIX2SCRATCH` with consumers `s2` and `scratch-serve` got leaders — a scratch stream created ad hoc on the production spine during some fix session; it is not in the current stream list (deleted since).
- 2026-08-15 06:03: nats3 became meta leader ("Self is new JetStream cluster metadata leader") after a node event; the last meta snapshot is from 2026-08-15 20:43.
- No restarts of nats.service on any node since each CT's last boot (NRestarts=0).

[15] CLAIM: Order must be nats2 → nats1 → nats3, not "seed first"; the "seed" label is documentation-only and carries no config meaning.
POINTER: live-state.md:13
OPENED TEXT:
## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0

[16] CLAIM: Order must be nats2 → nats1 → nats3, not "seed first"; the "seed" label is documentation-only and carries no config meaning.
POINTER: live-state.md:8
OPENED TEXT:
| nats2 | CT 122 | node2 (lvm) | same binary/hash | v2.14.2 | 10349df4… | 2026-07-26 21:07 UTC (CT rebooted then) | 0 | Debian 12 |
| nats3 | CT 123 | node3 (lvm), seed | same binary/hash | v2.14.2 | 10349df4… | 2026-07-28 13:28 UTC (CT rebooted then) | 0 | Debian 12 |

All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

[17] CLAIM: A1/A3 — `systemctl is-active` and even the restarted node's own `/healthz` are insufficient; the gate must be read from a peer that was not restarted.
POINTER: nats-store-monitor.sh:5
OPENED TEXT:
for n in <lan-address> <lan-address> <lan-address>; do
  used=$(curl -s --max-time 6 "http://$n:8222/jsz" | python3 -c 'import sys,json;print(json.load(sys.stdin).get("store",0))' 2>/dev/null)
  [ -z "$used" ] && { echo "$(date -u +%FT%TZ) WARN $n unreachable" | tee -a "$LOG"; continue; }
  pct=$(( used*100/LIMIT ))
  msg="$(date -u +%FT%TZ) $n store ${pct}% ($((used/1024/1024))MB/15GB)"

[18] CLAIM: A2 — duplicate ticket events are possible on restart, so decision criterion (1) "never duplicates" is unachievable by the spine and must be met by CT 116 idempotency; additionally `support-intake` can be head-of-line blocked forever by one poison message.
POINTER: live-state.md:22
OPENED TEXT:
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2

[19] CLAIM: A downgrade of the on-disk file store from 2.14.5 back to 2.14.2 is not establishable from the notes — SPECULATIVE — so the rollback path must not depend on it.
POINTER: v2.14.5:4
OPENED TEXT:
Refer to the [2.14 Upgrade Guide](https://docs.nats.io/release-notes/whats_new/whats_new_214) for backwards compatibility notes with 2.12.x. Please note that the 2.13.x version was skipped.

### Go Version

[20] CLAIM: A downgrade of the on-disk file store from 2.14.5 back to 2.14.2 is not establishable from the notes — SPECULATIVE — so the rollback path must not depend on it.
POINTER: v2.14.3:81
OPENED TEXT:
- Counter stream staging no longer corrupts the committed running total (#8311)
- Filestore compaction no longer corrupts compressed or encrypted blocks (#8312)
- Memory store `NumPending` no longer overcounts for `DeliverLastPerSubject` consumers (#8313)
- Consumer inactive-delete grace period handling and pull request `MaxBytes` budgeting have been fixed (#8314)
- `MultiLastSeqs` no longer reorders stream config subjects through `filterIsAll` handling (#8315)

[21] CLAIM: A downgrade of the on-disk file store from 2.14.5 back to 2.14.2 is not establishable from the notes — SPECULATIVE — so the rollback path must not depend on it.
POINTER: v2.14.4:51
OPENED TEXT:
- Empty cluster replicated entries are now correctly ignored (#8347, thanks to @uwezkhan)
- Decoded AVL sequence set node counts are now validated correctly on 32-bit systems (#8355, thanks to @uwezkhan)
- Stale error responses on source or mirror creation are now dropped by recreating the subscription (#8356)
- Oversized cluster replicated messages are now validated correctly on 32-bit systems (#8357, thanks to @uwezkhan)
- Raft elections now correctly ignore votes from removed peers (#8353)

[22] CLAIM: A downgrade of the on-disk file store from 2.14.5 back to 2.14.2 is not establishable from the notes — SPECULATIVE — so the rollback path must not depend on it.
POINTER: v2.14.4:27
OPENED TEXT:
- Inserts, iterations and deletes in AVL sequence sets are now faster in many cases, which speeds up the tracking of interior deletes (#8406)
- Stream snapshots now attempt to determine the correct encode buffer size up front, avoiding many unnecessary allocations on streams with large numbers of interior deletes (#8405)
- Reduced memory usage of the structure that is used to track subjects within a stream (#8412)

### Fixed

[23] CLAIM: A4 — the highest-value non-binary change is pinning `install-nats.sh`, which today re-installs whatever GitHub calls "latest" and would silently produce a 4th version on any re-run.
POINTER: install-nats.sh:6
OPENED TEXT:
apt-get install -y -qq curl ca-certificates >/dev/null 2>&1
NV=$(curl -fsSL https://api.github.com/repos/nats-io/nats-server/releases/latest | grep '"tag_name"' | head -1 | sed -E 's/.*"(v[^"]+)".*/\1/')
cd /tmp
curl -fsSL "https://github.com/nats-io/nats-server/releases/download/${NV}/nats-server-${NV}-linux-amd64.tar.gz" -o n.tgz
tar xzf n.tgz

[24] CLAIM: A4 — the highest-value non-binary change is pinning `install-nats.sh`, which today re-installs whatever GitHub calls "latest" and would silently produce a 4th version on any re-run.
POINTER: live-state.md:10
OPENED TEXT:
All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08

[25] CLAIM: A4 — the highest-value non-binary change is pinning `install-nats.sh`, which today re-installs whatever GitHub calls "latest" and would silently produce a 4th version on any re-run.
POINTER: live-state.md:6
OPENED TEXT:
|---|---|---|---|---|---|---|---|---|
| nats1 | CT 121 | node1 (zfs subvol) | /usr/local/bin/nats-server (17968728 B, dated Jun 22) | v2.14.2 | 10349df4… | 2026-06-22 03:31 UTC | 0 | Debian 12 |
| nats2 | CT 122 | node2 (lvm) | same binary/hash | v2.14.2 | 10349df4… | 2026-07-26 21:07 UTC (CT rebooted then) | 0 | Debian 12 |
| nats3 | CT 123 | node3 (lvm), seed | same binary/hash | v2.14.2 | 10349df4… | 2026-07-28 13:28 UTC (CT rebooted then) | 0 | Debian 12 |

[26] CLAIM: Q2's top risk is not in the newsletter at all: both live workloads authenticate as `control`, which holds publish+subscribe `>` — including full `$JS.API` destructive access to EVENTS/JOBS/RESULTS.
POINTER: live-state.md:25-28
OPENED TEXT:
## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=hale-support lang=nats.js lib=2.29.3 subs=1
- via nats2: client_ip=<lan-address> authorized_user=control name=hale-support-events lang=nats.js lib=2.29.3 subs=2

(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

[27] CLAIM: Q2's top risk is not in the newsletter at all: both live workloads authenticate as `control`, which holds publish+subscribe `>` — including full `$JS.API` destructive access to EVENTS/JOBS/RESULTS.
POINTER: live-state.md:30
OPENED TEXT:
(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

Observed config-vs-doctrine deviations (for seats to weigh, not conclusions):
- The build doctrine said "explicit stream limits or silent disk-full". Stream EVENTS has `max_bytes=-1` (unlimited) with max_age 30d; the 15 GB `max_file_store` is the only backstop.

[28] CLAIM: Q2's top risk is not in the newsletter at all: both live workloads authenticate as `control`, which holds publish+subscribe `>` — including full `$JS.API` destructive access to EVENTS/JOBS/RESULTS.
POINTER: live-state.md:38
OPENED TEXT:
## Recent journal signals (from `journalctl -u nats` on the nodes)
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).
- 2026-08-15 20:35–20:42: stream `ESTATE > FIX2SCRATCH` with consumers `s2` and `scratch-serve` got leaders — a scratch stream created ad hoc on the production spine during some fix session; it is not in the current stream list (deleted since).
- 2026-08-15 06:03: nats3 became meta leader ("Self is new JetStream cluster metadata leader") after a node event; the last meta snapshot is from 2026-08-15 20:43.
- No restarts of nats.service on any node since each CT's last boot (NRestarts=0).

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }