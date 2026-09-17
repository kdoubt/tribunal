Plausible framings are perimeter authentication, containment after guest compromise, and production lifecycle safety. I choose containment plus lifecycle safety: the decision criteria require isolation and recoverability even after access is granted.

All artifact paths below are relative to `artifact/`. No live infrastructure probes were executed.

**1. CLAIM:** The bastion is a real RTX traffic chokepoint, but GB10’s different isolation model leaves a potential direct path to host services outside Squid.
**EVIDENCE:** `ct205.conf:8` attaches RTX only to `cirub0`; `bastion-nftables.conf:14,19-24` drops forwarding and restricts private destinations. Conversely, `gb10-docker-inspect.txt:4` assigns the internal bridge gateway `172.30.99.1`; `gb10_setup.sh:38-39` supplies no isolated gateway mode. **EXTERNAL:** [Docker gateway documentation](https://docs.docker.com/engine/network/port-publishing/#gateway-modes) states that internal-network containers can reach host services bound to the bridge address or wildcard addresses. **SPECULATIVE:** successful service access depends on unprovided host listeners/firewall rules. No stable position on host escape or tenant-data disclosure.
**CONFIDENCE:** 0.88
**FALSIFIER:** Host inspection and guest-origin connection tests demonstrate complete denial of bridge-to-host services.

**2. CLAIM:** Window closure leaves guest GPU access enabled while restarting production, creating a concrete co-tenant contention path.
**EVIDENCE:** `RUNBOOK.md:69-75` restarts production without stopping CT205 or `the external party-bench`; `ct205.conf:14-20` grants GPU device access; `gb10_setup.sh:53-58` grants `--gpus all` and automatic restart. `RUNBOOK.md:95-96` explicitly acknowledges shared GPU failure impact. Neither close procedure requires terminating guest GPU processes.
**CONFIDENCE:** 0.97
**FALSIFIER:** A demonstrated close interlock terminates guest GPU processes and revokes device access before production restarts.

**3. CLAIM:** Access is load-bearing for the advertised tunnel entrance, but the supplied configuration does not make it mandatory for every SSH entrance.
**EVIDENCE:** `cloudflare-objects.json:5-6,17-20,28-32` pairs tunnel SSH with an owner-only Access allow policy. However, `bastion-nftables.conf:8` accepts SSH without source/interface restriction, and `gb10-docker-inspect.txt:3` publishes SSH on the LAN address. `RUNBOOK.md:42-46` requires adding email and keys; actual `authorized_keys` contents are not supplied, so their emptiness is unverified. **SPECULATIVE:** upstream filtering could restrict direct LAN entry.
**CONFIDENCE:** 0.93
**FALSIFIER:** Effective origin controls require the approved ingress path, and direct LAN SSH attempts fail independently of SSH-key validity.

**4. CLAIM:** A1 readiness is incomplete: restart configuration exists for some dependencies, but application readiness is largely unproved.
**EVIDENCE:** `bastion-cloudflared.service:3-11` provides network ordering, notify readiness, a 15-second startup timeout, and five-second failure restarts; it supplies no end-to-end tunnel-health predicate. `gb10_setup.sh:43-64` gives Squid, socat, and GB10 sshd `unless-stopped`, then uses a fixed three-second sleep without readiness retries. `gb10_setup.sh:67-71` prints status and HTTP codes; it does not assert expected codes. `bastion-services.conf:1-21` and `ct205-services.conf:1-10` supply no restart/readiness definitions for bastion sshd, RTX sshd, tinyproxy, or dnsmasq. Their effective unit policies remain unknown.
**CONFIDENCE:** 0.98
**FALSIFIER:** Effective units and bounded checks establish connector/tunnel health, both benchmark SSH logins, bastion SSH, proxy forwarding/denial, and DNS answers before access opens.

**5. CLAIM:** A2/A3 validation does not establish convergence or cache-independent DNS, Access, and TLS correctness.
**EVIDENCE:** `cloudflare-objects.json:1-42` contains configuration, not DNS records or validation results. `gb10_setup.sh:43-58` starts services before completing network attachments and endpoint creation; `:60,69-71` performs one-shot checks. Line 69 labels every failed connection—including refusal or startup failure—“blocked (good).” `RUNBOOK.md:30` reports reachability without probe transcripts. **ASSUMPTION:** DNS caches and Access propagation can make immediate observations stale; Docker rule-ordering behavior requires live inspection.
**CONFIDENCE:** 0.96
**FALSIFIER:** Recorded authoritative/fresh-resolver DNS checks, fresh unauthenticated/allowed/disallowed Access sessions, hostname TLS verification, and repeated post-convergence connectivity tests.

**6. CLAIM:** A4 shared-state handling lacks a documented preservation protocol for concurrent writers.
**EVIDENCE:**

- **Backup exclusions:** `backup-job.json:1` has `114,205`; `RUNBOOK.md:86` restores literal `114`. **ASSUMPTION:** another administrator/automation may add exclusions; replacement can erase them, while skipping removal leaves a stale exclusion.
- **SDN:** `sdn.cfg:1-5` identifies shared vnet/zone stores; `RUNBOOK.md:81-83` requires deletion and apply. Skipping those steps leaves configured objects. Other writers are unidentified.
- **Docker firewall/DOCKER-USER:** `gb10_setup.sh:5,38-58` avoids explicit host-rule edits but creates networks/publishing. **EXTERNAL:** [Docker firewall documentation](https://docs.docker.com/engine/network/firewall-iptables/) identifies Docker-managed rules and the preceding user-policy chain. Host administrator/automation ownership is **ASSUMPTION**; missing enforcement can leave unrestricted published ingress.
- **CT201 GPU nodes:** `ct205.conf:16-20` binds host nodes; `RUNBOOK.md:25-28` specifies sharing. No CT201 node mutation, ownership snapshot, or restoration is evidenced. **SPECULATIVE:** conflicting permission changes could prevent production GPU access.

**CONFIDENCE:** 0.94
**FALSIFIER:** Ownership records and coordinated, object-scoped updates preserve unrelated state and verify production GPU access.

**7. CLAIM:** A5 GB10 reruns replace containers but do not safely reconcile existing state.
**EVIDENCE:** `gb10_setup.sh:38-41` accepts any existing network named `cirubench` without checking isolation/subnet, then suppresses container-removal errors. `:53-58` assumes fixed addresses; `:56-57,62-64` reuse writable host directories. Thus container replacement does not clear guest persistence in those mounts.
**CONFIDENCE:** 0.97
**FALSIFIER:** Resume validation rejects incompatible networks, handles failed removals explicitly, and applies a documented clean-versus-preserve policy to mounted state.

**8. CLAIM:** A5 teardown names the requested object inventory, but its CT command is likely invalid and successful, repeatable completion is unproved.
**EVIDENCE:** `RUNBOOK.md:80-86` names CF app/policy/DNS/tunnel, both CTs, SDN vnet/zone, all three GB10 containers, network, image, home tree, and backup restoration. Line 81 uses `pct stop 204 205 && pct destroy 204 205`. **EXTERNAL:** the [Proxmox command definitions](https://lists.proxmox.com/pipermail/pve-devel/2015-October/017569.html) accept one positional `vmid` per command. **SPECULATIVE:** deployed-version rejection remains untested.
**CONFIDENCE:** 0.94
**FALSIFIER:** Deployed `pct` help supports that syntax, and two teardown executions verify every listed object absent without damaging unrelated state.

**VERDICT INPUT**

Q1: **harden-first** — retain the chokepoint, close GB10 host-access uncertainty (1).  
Q2: **harden-first** — enforce origin restrictions and validate containment (1, 3).  
Q3: **don’t** — do not open access with the current window-close procedure (2).  
Highest severity overall: **guest GPU access survives production restart**, exposing co-tenants to contention and shared-driver failure (2).
