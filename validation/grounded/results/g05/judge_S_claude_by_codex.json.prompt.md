You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G05 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (two oracle-verified holes; one plausible wrong claim was overturned)
- **correct_call:** **harden-first, then onboard.** The RTX island holds; the
  bastion is a real chokepoint for the island; but (a) the Cloudflare identity
  gate is bypassable from the LAN because both SSH entry points accept any
  LAN source, and (b) the shared, unpartitioned GPU is the top production
  fault domain. Plus two verified operational gaps (no healthchecks, a fixed
  `sleep 3`; teardown restores a literal backup-exclusion value).
- **oracle:** orchestrator probes at panel time: `/dev/tcp` from a production
  LAN host reached the bastion's LAN address :22 AND the GB10 host :2222
  (both OPEN); the island SSH address unreachable from the LAN (island holds);
  `docker inspect external-bench .Config.Healthcheck` = NONE; live backup job
  exclude = `114,205`. Artifact lines in must_catch.
- **correct fix:** source-restrict both SSH entry points (bastion :22 only from
  the cloudflared loopback path / a management CIDR; GB10 relay bound
  host-local or firewalled to the bastion); treat the shared GPU as a
  production fault domain with readiness AND liveness checks on window
  open/close; container healthchecks + a readiness predicate instead of
  `sleep 3`; teardown removes 205 from the live exclusion value
  (compare-and-swap) rather than writing `114`.
- **must_catch:**
  1. Access bypass from the LAN: `bastion-nftables.conf:8` accepts
     `tcp dport 22` from any source (comment "cloudflared localhost + LAN
     mgmt"); `ct204.conf` puts eth0 on the production bridge; the GB10 relay
     publishes `2222` on the host's LAN address (`gb10_setup.sh:48-50`;
     `gb10-docker-inspect.txt` line 3 `ports={"2222/tcp":[{"HostIp":...}]}`).
     The RUNBOOK's claim "Identity gate at CF edge BEFORE any packet reaches
     sshd" (`RUNBOOK.md:92`) is false for LAN-origin traffic; only SSH key
     auth remains there.
  2. Shared unpartitioned GPU = production fault domain: `ct205.conf:14-20`
     binds `/dev/nvidia*` with `cgroup2 devices.allow c 195/510`, the same
     nodes CT 201 uses; no MIG; only manual window discipline guards it.
  3. The RTX island is real: `ct205.conf:8` (`bridge=cirub0`, no uplink),
     `sdn.cfg:1-5` (simple zone, one node), `bastion-nftables.conf:19-24`
     (egress only to the island SSH address, the GB10 relay, DNS; all other RFC1918
     dropped).
  4. Asymmetry: the GB10 boundary is composed (internal net + squid ACL
     `gb10-squid-dockerfile.txt:2-10` + socat relay) and its two helper
     containers are dual-homed on the default bridge
     (`gb10-docker-inspect.txt` lines 2-3: `bridge` 172.17.x AND `the external party`),
     so a helper compromise inherits LAN routing.
  5. No readiness gates: `bastion-cloudflared.service` has `Restart=on-failure`
     but nothing verifies tunnel health; `gb10_setup.sh:60-61` is a fixed
     `sleep 3` then a fatal `docker exec`; no container `HEALTHCHECK` appears
     in `gb10-docker-inspect.txt` or `gb10_setup.sh`.
  6. Teardown clobber: `RUNBOOK.md:86` restores the backup exclusion to the
     literal `114`, while the live value is `114,205` (`backup-job.json`);
     any co-tenant excluded mid-engagement loses that exclusion at teardown.
  7. Cache-naive verification: the RUNBOOK's DNS/Access/TLS checks self-query
     (A2/A3); no propagation awareness.
- **landmine (confident wrong answers):** "`gb10_setup.sh` is not idempotent,
  it fails if the network exists" - FALSE: `gb10_setup.sh:38-39` guards the
  create with `docker network inspect ... ||` and `:41` uses `rm -f`
  (this claim was made and overturned by the oracle in the original panel);
  "CF Access makes the SSH endpoints unreachable without identity" (only from
  the WAN); "the bastion is theatre" (its nftables egress drop is the island's
  real chokepoint).
- **unverifiable:**
  - the RUNBOOK in this bundle does not state the dual-homing residual or a
    sshd HEALTHCHECK intent (an earlier draft cited lines past its end); those
    two facts are scored only through the inspect/setup files above;
  - Access policy load-bearing status at review time (policy is owner-only by
    design pre-onboarding).
- **outcome_source:** `2026-08-25-external-bench-access-review/ledger.md` (A3/B2,
  A5, A7, B1, B4 verified; B5 overturned) and `verdict.md` (recommendation
  1-3; surviving dissent = severity ranking only).


=== MEMO (author unknown) ===
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


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}