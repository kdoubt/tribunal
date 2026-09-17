You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: release-notes-v2.14.3.md:67
OPENED TEXT:
- A data race on the cluster meta node during JetStream shutdown has been fixed (#8260)
- Meta proposal inflight tracking is now kept consistent during stream moves and related operations (#8261)
- Stream catchup is no longer skipped when limits are exceeded, preventing possible stream desync (#8265)
- Malformed TTL and schedule state is now rejected during decode (#8269)

[2] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: release-notes-v2.14.4.md:59
OPENED TEXT:
- Fixed string ownership when handling the expected last sequence per subject in a batch (#8377)
- Fixed a race condition between concurrent message removals via limits that could unexpectedly disable writes into a filestore (#8378)
- Fixed a bug in the filestore which could prevent some filestore block cache references from being weakened correctly, which could result in unexpected memory usage and GC pressure (#8380)
- Attempting to update the consumer storage type now correctly returns an error (#8382)
- Stream publish checks will now correctly reject messages that exceed the maximum store size before proposal (#8389)

[3] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: release-notes-v2.14.5.md:29
OPENED TEXT:
- Fix a bug that could result in potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot, caused by an incorrect update to the create time in the stream assignment (#8449)

### Complete Changes

[4] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: live-state.md:14
OPENED TEXT:
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0
    - consumer hale-eval-workers: filter=jobs.hale.eval ack_wait=300s max_deliver=4 max_ack_pending=8 pending=0 redelivered=0 waiting_pulls=0

[5] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: release-notes-v2.14.4.md:37-39
OPENED TEXT:
- JWT validation no longer crashes the server with whitespace-only permissions
- Several paths that enforce the permissions of queue subscriptions no longer treat the whole permission as a subject literal
- Several JetStream and MQTT endpoints now correctly guard against null values in JSON
- Fixed an authentication bypass with TLS `verify_and_map` authenticating users with blank passwords

Monitoring

[6] CLAIM: ** Upgrade to 2.14.5 is justified by applicable clustered-storage fixes, but the owner overstates both blanket authentication exposure and the snapshot-loss trigger.
POINTER: release-notes-v2.14.3.md:50
OPENED TEXT:
- Leaf connections no longer bypass `Nats-Trace-Dest` publish permission checks
- `CONNZ` and `SUBSZ` pagination now guard against `Offset` and `Limit` integer overflow panics
- Fixed a nil pointer panic when starting up when the resolver parent directory is missing (#8329)

MQTT

[7] CLAIM: ** A2 requires proving ticket idempotency and client recovery before disruption, because redelivery cannot be assumed harmless.
POINTER: live-state.md:22
OPENED TEXT:
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2

[8] CLAIM: ** The proposed rollout is nats2 → nats1 → nats3, re-evaluated against live leadership, with pinned binaries and a cold per-node recovery copy.
POINTER: live-state.md:13-21
OPENED TEXT:
## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0
    - consumer hale-eval-workers: filter=jobs.hale.eval ack_wait=300s max_deliver=4 max_ack_pending=8 pending=0 redelivered=0 waiting_pulls=0
    - consumer rtx-serve: filter=jobs.serve.> ack_wait=30s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM RESULTS: subjects=['results.>'] retention=limits max_age=3d max_bytes=2147483648 discard=old R3 storage=file | msgs=380 bytes=63214 leader=nats1
    - consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['e

[9] CLAIM: ** The proposed rollout is nats2 → nats1 → nats3, re-evaluated against live leadership, with pinned binaries and a cold per-node recovery copy.
POINTER: install-nats.sh:6-10
OPENED TEXT:
apt-get install -y -qq curl ca-certificates >/dev/null 2>&1
NV=$(curl -fsSL https://api.github.com/repos/nats-io/nats-server/releases/latest | grep '"tag_name"' | head -1 | sed -E 's/.*"(v[^"]+)".*/\1/')
cd /tmp
curl -fsSL "https://github.com/nats-io/nats-server/releases/download/${NV}/nats-server-${NV}-linux-amd64.tar.gz" -o n.tgz
tar xzf n.tgz
install -m755 nats-server-*-linux-amd64/nats-server /usr/local/bin/nats-server
id nats >/dev/null 2>&1 || useradd --system --home-dir /var/lib/nats --shell /usr/sbin/nologin nats
mkdir -p /var/lib/nats/js /etc/nats && chown -R nats:nats /var/lib/nats
echo "nats-server $(/usr/local/bin/nats-server --version)"

[10] CLAIM: ** The proposed rollout is nats2 → nats1 → nats3, re-evaluated against live leadership, with pinned binaries and a cold per-node recovery copy.
POINTER: live-state.md:44
OPENED TEXT:
## Backup / maintenance windows
- PVE hourly `vzdump` (all guests, snapshot mode, to iscsi-fast) runs at the top of every hour. Estate rule: check cluster tasks and avoid the :00 windows before any disruptive op (a reboot during vzdump once locked a live-trading CT for 20 h).
- `nats-store-monitor.timer` on node1 polls `/jsz` on all three nodes every 15 min and alerts at >80% of 15 GB; the last runs show 0%. It only checks store bytes — not version, quorum, leader, or consumer lag.
- Cross-node `estate-deadman` watchdog exists (pings node1/2/3), unrelated to NATS health.

[11] CLAIM: ** A5’s prolonged mixed-version safety and direct file-store downgrade remain **SPECULATIVE**, so neither should be an unattended assumption.
POINTER: release-notes-v2.14.3.md:4
OPENED TEXT:
Refer to the [2.14 Upgrade Guide](https://docs.nats.io/release-notes/whats_new/whats_new_214) for backwards compatibility notes with 2.12.x. Please note that the 2.13.x version was skipped.

### Go Version

[12] CLAIM: ** A5’s prolonged mixed-version safety and direct file-store downgrade remain **SPECULATIVE**, so neither should be an unattended assumption.
POINTER: release-notes-v2.14.4.md:4
OPENED TEXT:
Refer to the [2.14 Upgrade Guide](https://docs.nats.io/release-notes/whats_new/whats_new_214) for backwards compatibility notes with 2.12.x. Please note that the 2.13.x version was skipped.

### Go Version

[13] CLAIM: ** A5’s prolonged mixed-version safety and direct file-store downgrade remain **SPECULATIVE**, so neither should be an unattended assumption.
POINTER: release-notes-v2.14.5.md:4
OPENED TEXT:
Refer to the [2.14 Upgrade Guide](https://docs.nats.io/release-notes/whats_new/whats_new_214) for backwards compatibility notes with 2.12.x. Please note that the 2.13.x version was skipped.

### Go Version

[14] CLAIM: ** Monitoring needs correction before rollout because its missing-field default can report zero usage.
POINTER: nats-store-monitor.sh:5
OPENED TEXT:
for n in <lan-address> <lan-address> <lan-address>; do
  used=$(curl -s --max-time 6 "http://$n:8222/jsz" | python3 -c 'import sys,json;print(json.load(sys.stdin).get("store",0))' 2>/dev/null)
  [ -z "$used" ] && { echo "$(date -u +%FT%TZ) WARN $n unreachable" | tee -a "$LOG"; continue; }
  pct=$(( used*100/LIMIT ))
  msg="$(date -u +%FT%TZ) $n store ${pct}% ($((used/1024/1024))MB/15GB)"

[15] CLAIM: ** After immediate continuity safeguards, shared unrestricted credentials are the highest-priority security correction and should remain a separate change.
POINTER: live-state.md:25-30
OPENED TEXT:
## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=hale-support lang=nats.js lib=2.29.3 subs=1
- via nats2: client_ip=<lan-address> authorized_user=control name=hale-support-events lang=nats.js lib=2.29.3 subs=2

(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

Observed config-vs-doctrine deviations (for seats to weigh, not conclusions):
- The build doctrine said "explicit stream limits or silent disk-full". Stream EVENTS has `max_bytes=-1` (unlimited) with max_age 30d; the 15 GB `max_file_store` is the only backstop.

[16] CLAIM: ** After immediate continuity safeguards, shared unrestricted credentials are the highest-priority security correction and should remain a separate change.
POINTER: live-state.md:38-39
OPENED TEXT:
## Recent journal signals (from `journalctl -u nats` on the nodes)
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).
- 2026-08-15 20:35–20:42: stream `ESTATE > FIX2SCRATCH` with consumers `s2` and `scratch-serve` got leaders — a scratch stream created ad hoc on the production spine during some fix session; it is not in the current stream list (deleted since).
- 2026-08-15 06:03: nats3 became meta leader ("Self is new JetStream cluster metadata leader") after a node event; the last meta snapshot is from 2026-08-15 20:43.
- No restarts of nats.service on any node since each CT's last boot (NRestarts=0).

[17] CLAIM: ** Capacity/retry hygiene ranks below continuity and credential exposure, while the other newsletter products can presently be deferred.
POINTER: live-state.md:21-22
OPENED TEXT:
- consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }