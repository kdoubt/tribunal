A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

===== Memo 1 =====

## Framing

Plausible framings: (a) *is the perimeter airtight?* — pure edge/identity review; (b) *will the build/window break production?* — ops-first; (c) *assume the guest session is already compromised and ask what the blast radius is.* I pick (c): the brief explicitly names "an attacker who compromises the external party's session," and the owner's tie-break ranks confirmed lateral reach and co-tenant outage above everything, so the trust model must be "hostile root inside both bench endpoints."

---

CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
EVIDENCE: `ct205.conf:14-19` (`lxc.cgroup2.devices.allow: c 195:* rwm` / `c 510:* rwm` + five `/dev/nvidia*` bind mounts) with `ct205-services.conf:5` `the external party ALL=(ALL) NOPASSWD:ALL`; GB10 mirror at `gb10_setup.sh:15` (`echo 'the external party ALL=(ALL) NOPASSWD:ALL'`) plus `--gpus all` (`gb10_setup.sh:54`) and no `--cap-drop`/`--security-opt no-new-privileges`. The runbook concedes it: `RUNBOOK.md:95-96` "GPU shared with CT 201 — isolation is fs/process-level, not GPU-level (a driver-crashing workload would still hit vLLM)."
CONFIDENCE: 0.9 that this is the top-ranked exposure under the owner's criteria; 0.25 that a *kernel escape* (as opposed to GPU DoS/VRAM starvation) is actually achieved in a time-boxed window.
FALSIFIER: A MIG partition or a separate GPU for 205, or evidence that CT 201's vLLM is stopped and the RTX is unshared for the entire engagement, plus an equivalent on GB10.

CLAIM: The Cloudflare identity gate is load-bearing **only for the WAN path**; both bench SSH endpoints are directly reachable from the flat production LAN, so any co-tenant dev box or compromised prod container bypasses Access entirely and faces only `authorized_keys`.
EVIDENCE: `bastion-nftables.conf:8` `tcp dport 22 accept comment "sshd (cloudflared localhost + LAN mgmt)"` — no `ip saddr` restriction on CT 204's `<lan-address>`; `gb10-docker-inspect.txt:3` `ports={"2222/tcp":[{"HostIp":"<lan-address>","HostPort":"2222"}]}` with `gb10_setup.sh:5` "no host iptables/systemd changes." Contrast `RUNBOOK.md:92` "Identity gate at CF edge (email allowlist) BEFORE any packet reaches sshd."
CONFIDENCE: 0.88
FALSIFIER: A host firewall on GB10 or an nft `iif eth0` restriction on CT 204 limiting :22/:2222 to the loopback/admin source.

CLAIM: Both egress denylists cover **IPv4 private ranges only** — no IPv6, no link-local — so if either segment has IPv6 (SLAAC/RA), the "no private egress" and "denies RFC1918" guarantees do not hold on that family.
EVIDENCE: `bastion-nftables.conf:24` `ip daddr { <lan-address>/16, 10.0.0.0/8, 172.16.0.0/12 } drop` inside `table inet filter` with `policy accept` on output (`:16`) — `ip daddr` never matches IPv6; `gb10-squid-dockerfile.txt:3` `acl lan dst <lan-address>/16 10.0.0.0/8 172.16.0.0/12` then `:9` `http_access allow benchnet`. Settling probe: `pct exec 204 -- ip -6 addr show eth0` and `curl -x http://172.30.99.3:3128 'http://[<lan-host-v6>]:80/'`.
CONFIDENCE: 0.6 (the ruleset gap is certain from the file; exploitability depends on unverified IPv6 presence — that half is SPECULATIVE)
FALSIFIER: `ip -6 addr` shows no global/ULA v6 on CT 204 eth0 or on the GB10 docker bridge.

CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
EVIDENCE: `ct205.conf:8` has no `gw=` (no default route off `cirub0`); `bastion-nftables.conf:14` `chain forward { ... policy drop; }` so the bastion never routes; tinyproxy's unrestricted destinations (`bastion-services.conf:8-17` — `Allow <island-address>`, `ConnectPort 443/80`, no dest filter) are backstopped by `bastion-nftables.conf:24`. On GB10 the equivalent backstop is squid ACLs alone, and `the external party-proxy` is dual-homed onto the NAT'd default bridge (`gb10-docker-inspect.txt:2` `"bridge"..."IPAddress":"172.17.0.6"`).
CONFIDENCE: 0.8
FALSIFIER: A successful `curl -x http://<island-address>:8888 http://<lan-address>X:80/` from CT 205 returning non-error.

CLAIM: `gb10_setup.sh`'s verification block **cannot fail the build** — every isolation assertion is echo-only or `|| true`, so a squid misconfiguration that permits LAN egress still exits 0.
EVIDENCE: `gb10_setup.sh:66-71`; line 69 ends `&& echo "LAN REACHABLE (BAD)" || echo "LAN direct: blocked (good)"`, line 71 ends `|| true`, line 68 ends `|| true`. `set -euo pipefail` (`:6`) is defeated by each.
CONFIDENCE: 0.95
FALSIFIER: A non-zero exit from the script when squid is replaced with a permit-all config.

CLAIM: Teardown touches two shared mutable stores with blind read-modify-write, matching the estate's own lost-update precedent: `pvesh set /cluster/sdn` applies **all** pending cluster-wide SDN state, and the backup-exclude restore hardcodes `114`.
EVIDENCE: `RUNBOOK.md:82` "`pvesh delete /cluster/sdn/vnets/cirub0`, `.../zones/cirub`, then `pvesh set /cluster/sdn`"; `RUNBOOK.md:86` "restore exclude to `114` only"; current value `backup-job.json:1` `"exclude":"114,205"`. Symptom if a third party added `"114,205,301"` meanwhile: 301 silently resumes being backed up.
CONFIDENCE: 0.7
FALSIFIER: The teardown is scripted as a read-filter-write (remove only `205`) and SDN pending state is inspected via `pvesh get /cluster/sdn` before apply.

CLAIM: A1 readiness is weak on the LXC side and racy on GB10: CT 205 is `onboot: 0` while the bastion is `onboot: 1`, so a node reboot leaves the guest with a live entry point and a dead target; the GB10 build uses a fixed `sleep 3` as its only readiness predicate.
EVIDENCE: `ct205.conf:10` `onboot: 0` vs `ct204.conf:10` `onboot: 1`; `gb10_setup.sh:60` `sleep 3` before `docker exec the external party-bench ...` (`:61`); `bastion-cloudflared.service:8-11` is the only component with `Restart=on-failure` — `Type=notify` + `TimeoutStartSec=15` will also loop-restart if cloudflared fails to notify in 15s.
CONFIDENCE: 0.85 for the config facts; 0.3 for the cloudflared notify-timeout loop specifically (EXTERNAL: cloudflared's packaged unit does use sd_notify, so 15s is likely adequate).
FALSIFIER: `pct config 205 | grep onboot` showing `1`, or a documented "start 205 first" step in the window procedure.

CLAIM: CT 204 is **not** excluded from the hourly vzdump job, so the cloudflared tunnel token and the guest's installed pubkey are copied hourly to shared `iscsi-fast` and survive teardown for up to three months.
EVIDENCE: `backup-job.json:1` `"all":1,...,"exclude":"114,205",...,"prune-backups":{"keep-daily":"7",...,"keep-monthly":"3"},"storage":"iscsi-fast"` — 204 absent from `exclude`; token lives in `bastion-cloudflared.service:9` `--token=REDACTED` inside CT 204. `RUNBOOK.md:78-87` teardown has no backup-purge step.
CONFIDENCE: 0.85 (fact); 0.35 that it matters materially, since teardown step 1 deletes the tunnel, revoking the token.
FALSIFIER: The tunnel is deleted at teardown (it is, `RUNBOOK.md:80`), or 204 appears in a per-guest exclusion elsewhere.

---

## VERDICT INPUT

- **Q1 Architecture — harden-first.** The shape is right and the asymmetry is defensible (the SDN island is the stronger model; the bastion is a real chokepoint, not theatre), but the GB10 side achieves the same goal with a strictly weaker, ACL-only backstop and a LAN-bound published port. Converge them or accept that GB10 is the soft side.
- **Q2 Security — harden-first.** No confirmed IPv4 lateral path from CT 205; the identity gate is real but WAN-only.
- **Q3 Systems/operational — harden-first.** Non-asserting verification, `onboot: 0`, blind RMW on two shared stores, and a runbook that has already drifted from live state (`RUNBOOK.md:13` "[NOT YET BUILT]" vs three running containers in `gb10-docker-inspect.txt:1-3`) — the drift matters because the runbook is the teardown authority.

**Highest-severity finding overall:** hostile root inside both bench endpoints holds unmediated ioctl access to GPU character devices shared with live production vLLM (`ct205.conf:14-19` + `ct205-services.conf:5`; `gb10_setup.sh:15,54`). Network isolation is sound; the GPU is the shared resource no rule in this artifact set constrains.

===== Memo 2 =====

I'll ground this review in the actual artifact files first, then form an independent position on architecture, security, and operations.**Framings considered:** (1) ship the island — RTX topology is the template and GB10 is “good enough docker”; (2) don’t ship until GB10 matches RTX invariants and GPU exclusion is mechanical; (3) shape is right, ops (A1–A5) are not; (4) CF Access is theatre because two of three sshds are on the LAN. **Picked (2)+(3):** owner criteria rank confirmed co-tenant outage / lateral reach over nits; the family (edge identity + bastion + isolated compute) is right under the no-re-VLAN constraint, but GB10 is not a chokepoint and GPU exclusion is process-only.

---

CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
EVIDENCE: `ct205.conf:8` `bridge=cirub0` only; `sdn.cfg` zone `cirub` / vnet `cirub0` with no uplink; `ct204.conf:8-9` dual-home `vmbr0` + `cirub0`; `bastion-nftables.conf:14` `forward` policy `drop`; `gb10-docker-inspect.txt:3` `ports={"2222/tcp":[{"HostIp":"<lan-address>","HostPort":"2222"}]}`; `gb10_setup.sh:5` “no host iptables/systemd changes”.
CONFIDENCE: 0.93
FALSIFIER: `cirub0` has an uplink/NAT path to vmbr0, or GB10 `:2222` is bound only to a non-LAN address and filtered to `<lan-address>`.

CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
EVIDENCE: `cloudflare-objects.json:4-6` ingress `ssh://localhost:22` for `the bench hostname`; `cloudflare-objects.json:26-34` Access allow email `<email>` only; `gb10-docker-inspect.txt:3` host publish `.225:2222`; `bastion-nftables.conf:8` `tcp dport 22 accept` with no `ip saddr`; `bastion-services.conf:6-7` `AllowTcpForwarding yes`.
CONFIDENCE: 0.92
FALSIFIER: A packet to `<lan-address>` from a non-bastion LAN source is dropped before socat, or bastion sshd is bound only to localhost/`<island-address>`.

CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
EVIDENCE: `ct205.conf:14-20` `devices.allow c 195:*` / `510:*` and nvidia bind-mounts; `ct205.conf:9` `onboot: 0` but no window-close stop; `RUNBOOK.md:67-75` close path is `start vllm` / `docker update --memory 20g` + start three vLLMs, not GPU detach; `gb10-docker-inspect.txt:1` `mem=21474836480 restart=unless-stopped` with `--gpus all` in `gb10_setup.sh:53-54`; `RUNBOOK.md:12,32` still says GB10 `[NOT YET BUILT]`.
CONFIDENCE: 0.88
FALSIFIER: Window close stops/destroys CT 205 or drops nvidia cgroup allows, and `the external party-bench` is not running with GPU devices while the three GB10 vLLMs are up.

CLAIM: Guest→LAN from CT 205 is blocked by construction; guest→LAN from GB10 is only a squid RFC1918 deny on a dual-homed proxy that already has a docker0 default route, and the only LAN probe never tested the internal-net host gateway `172.30.99.1`.
EVIDENCE: `gb10-docker-inspect.txt:2` `the external party-proxy` `bridge` `Gateway":"172.17.0.1"` + `cirubench` `172.30.99.3`; `gb10-squid-dockerfile.txt:3,6` `acl lan dst <lan-address>/16 10.0.0.0/8 172.16.0.0/12` / `http_access deny lan`; `gb10-docker-inspect.txt:4` `cirubench internal=true` `Gateway":"172.30.99.1"` vs bench `Gateway":""`; `gb10_setup.sh:69-71` probes only `<lan-address>` direct and via squid.
CONFIDENCE: 0.78
FALSIFIER: `docker exec the external party-bench` cannot open TCP to `172.30.99.1` or any GB10-host/published port, and squid has no path to `<lan-address>/24` even after ACL bypass attempts (IPv6, `169.254.0.0/16`).

CLAIM: A1: only cloudflared has a readiness/restart story; GB10 bring-up is racy and the island daemons have no units in-tree.
EVIDENCE: `bastion-cloudflared.service:3-11` `After=network-online.target`, `Type=notify`, `Restart=on-failure`; `gb10_setup.sh:48-51` socat starts with `tcp:172.30.99.10:22` then `docker network connect cirubench`; `gb10_setup.sh:43-46` squid starts on default bridge then connect `--ip 172.30.99.3`; `gb10_setup.sh:60` `sleep 3` then exec; `bastion-services.conf` / `ct205-services.conf` are drop-in fragments with no systemd restart/health stanza.
CONFIDENCE: 0.86
FALSIFIER: Healthchecks/`Requires=` exist live for tinyproxy, dnsmasq, both sshds, squid, and socat, and socat is attached to `cirubench` before it listens.

CLAIM: A2/A3: immediate checks can false-pass, and no cache-proof DNS/Access/TLS validation is recorded.
EVIDENCE: `gb10_setup.sh:60-71` verify runs after `sleep 3` with `|| true` on GPU and LAN-via-squid; `gb10_setup.sh:5` isolation is docker iptables only (ordering vs other engine rules); `cloudflare-objects.json:20` `session_duration: "24h"`; `RUNBOOK.md:40-46` onboarding is dashboard/script with no `dig +trace` / uncached Access probe. A3 cache-fooling: no artifact records a validation method at all (`ASSUMPTION` that undocumented operator checks may exist).
CONFIDENCE: 0.74
FALSIFIER: A logged probe that failed closed until CF DNS/Access were globally visible and until squid/socat listened on `172.30.99.0/24`.

CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
EVIDENCE: `backup-job.json:1` `"exclude":"114,205"` on hourly job `cd5e47ba-…`; `sdn.cfg` `/etc/pve/sdn/vnets.cfg` + `zones.cfg`; `gb10-docker-inspect.txt:3` host DNAT `:2222`; `gb10_setup.sh:5` explicitly skips `DOCKER-USER`; `ct205.conf:16-20` vs CT 201’s same nvidia nodes (runbook `RUNBOOK.md:25-26`).
CONFIDENCE: 0.84
FALSIFIER: Exclude/SDN/DOCKER-USER updates are atomic/owned by one writer, and CT 201’s nvidia nodes are not simultaneously accessible to CT 205.

CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
EVIDENCE: `gb10_setup.sh:38-41`; `RUNBOOK.md:78-87` deletes CF app/policy/CNAME/tunnel, `pct destroy 204 205`, SDN vnet+zone, three containers + `cirubench` + image + home dir, exclude back to `114`; omits restart of CT 201 vLLM and GB10’s three vLLMs, pulled `ubuntu/squid`/`alpine/socat`/nvcr base, PBS snapshots of CT 204 (token in `bastion-cloudflared.service:9`), and `DOCKER-USER`; `ct205-services.conf:5` and `gb10_setup.sh:15` `NOPASSWD:ALL`; `RUNBOOK.md:40-45` keys go in 204, 205, and GB10 with no `from="<lan-address>"`.
CONFIDENCE: 0.9
FALSIFIER: A teardown drill that leaves no CF objects, no CTs/SDN, no GB10 containers/network/image/home, exclude=`114`, vLLMs healthy, and no `:2222` listener.

**VERDICT INPUT**
- Q1 Architecture: harden-first
- Q2 Security: harden-first
- Q3 Systems/operational: harden-first
- Highest-severity finding: production GPUs remain attachable to the guest outside exclusive windows (CT 205 nvidia mounts persist; GB10 `the external party-bench` already `--gpus all`/`unless-stopped`), recreating the vLLM-over-fraction crashloop; GB10 `:2222` on the LAN makes Access/bastion non-gating for that sshd.

Must-be-true before keys/email: Access email matches the key holder; `authorized_keys` on all three sshds with GB10 `from=<lan-address>`; bind/filter `:2222` off the broadcast LAN; window close stops GPU access (stop 205 / `docker update` + drop `--gpus` or stop `the external party-bench`); single-writer apply for PBS exclude and SDN; tear down includes vLLM restore.
