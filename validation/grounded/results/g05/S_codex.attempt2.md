Plausible framings are ingress authentication, containment after guest compromise, and safe production lifecycle. I choose containment plus lifecycle: the stated objective requires surviving an authenticated guest’s actions, and the runbook acknowledges shared GPU failure impact (`artifact/RUNBOOK.md:95–96`).

**1. CLAIM:** The highest-severity demonstrated hole is unenforced GPU exclusivity: closing either window leaves benchmark workloads able to compete with restarted production.
**EVIDENCE:** `artifact/ct205.conf:14–20` grants shared GPU devices; `artifact/gb10_setup.sh:53–58` grants `--gpus all` with `--restart unless-stopped`; `artifact/RUNBOOK.md:69–75` restarts production without stopping either benchmark guest; `artifact/RUNBOOK.md:95–96` explicitly acknowledges shared-driver impact.
**CONFIDENCE:** 0.95
**FALSIFIER:** A demonstrated close-window interlock terminates benchmark processes and removes their GPU access before production restarts, including after host reboot.

**2. CLAIM:** The bastion provides meaningful RTX filtering, but GB10’s different isolation model creates an additional host-access boundary that the supplied configuration does not establish as closed.
**EVIDENCE:** RTX island: `artifact/ct205.conf:8`, `artifact/sdn.cfg:1–5`; bastion forwarding drop and private-destination filtering: `artifact/bastion-nftables.conf:14–24`. GB10’s dual-homed proxy/relay and bridge gateway `172.30.99.1`: `artifact/gb10-docker-inspect.txt:2–4`. **EXTERNAL:** Docker documents that an internal network ordinarily permits access to host services listening on its bridge address or wildcard addresses ([Docker gateway modes](https://docs.docker.com/engine/network/port-publishing/#gateway-modes)). **SPECULATIVE:** Reachable host services depend on unsupplied host listeners/firewall rules. No stable position on demonstrated host escape or another tenant’s data disclosure.
**CONFIDENCE:** 0.89
**FALSIFIER:** Host INPUT rules demonstrably block guest-to-host services while preserving the intended proxy/relay paths.

**3. CLAIM:** Cloudflare identity is load-bearing for the advertised tunnel route, but it is not a universal SSH gate.
**EVIDENCE:** Hostname-to-local-SSH mapping and owner-only allow policy: `artifact/cloudflare-objects.json:5–6,25–38`; unrestricted source acceptance for bastion SSH: `artifact/bastion-nftables.conf:8`; LAN-published GB10 SSH relay: `artifact/gb10-docker-inspect.txt:3`. Onboarding adds ordinary SSH keys at all endpoints (`artifact/RUNBOOK.md:42–46`). Thus the configured LAN paths do not traverse Access. **SPECULATIVE:** Upstream controls might block those paths; their configuration is absent. Empty current authorized_keys files and perimeter WAN exposure cannot be independently verified from this bundle.
**CONFIDENCE:** 0.93
**FALSIFIER:** Effective network controls reject direct LAN SSH to both entry points, and fresh authorized/unauthorized identity tests demonstrate the intended edge gate.

**4. CLAIM:** A1 readiness is incomplete: only cloudflared has an explicit startup-notification contract, while the remaining dependencies lack demonstrated end-to-end readiness and bounded retry.
**EVIDENCE:** Connector: `artifact/bastion-cloudflared.service:7–11` specifies `Type=notify`, 15-second startup timeout, failure restart, five-second delay; tunnel-route health is not tested there. Squid, socat and GB10 sshd: `artifact/gb10_setup.sh:43–60` specifies restart policies followed by fixed `sleep 3`, without health predicates. Squid’s functional curl is one-shot (`:70`); LAN checks print results and tolerate failures (`:69,71`). Tinyproxy/dnsmasq configurations provide no service restart/readiness definitions (`artifact/bastion-services.conf:8–21`); bastion and RTX sshd snippets likewise omit them (`artifact/bastion-services.conf:1–7`, `artifact/ct205-services.conf:1–4`). Their actual service units were not supplied.
**CONFIDENCE:** 0.97
**FALSIFIER:** Supplied service definitions and bounded probes establish DNS resolution, proxy allow/deny behavior, authenticated SSH through each route, and tunnel recovery.

**5. CLAIM:** A2–A3 convergence and cache-independent DNS/Access/TLS validation are not established by the recorded verification.
**EVIDENCE:** `artifact/RUNBOOK.md:16–18` lists intended CNAME/Access objects but supplies no resolution or authentication transcript; `artifact/cloudflare-objects.json:1–42` contains no DNS record or TLS observation. `artifact/gb10_setup.sh:60–71` checks once after three seconds, tests one LAN destination, and does not assert expected HTTP status codes. **ASSUMPTION:** DNS caches, Access propagation/session reuse, and concurrent Docker firewall reconciliation could produce transient or stale outcomes; no such event is demonstrated.
**CONFIDENCE:** 0.94
**FALSIFIER:** Repeated tests using authoritative and fresh recursive DNS, fresh allowed/denied Access sessions, hostname/SNI TLS verification, and packet checks after Docker restart converge to asserted results.

**6. CLAIM:** A4 requires coordinated handling of four shared stores, but writer ownership and safe reconciliation are insufficiently documented.
**EVIDENCE:**

- Backup exclusions: `artifact/backup-job.json:1` has `114,205`; `artifact/RUNBOOK.md:86` prescribes replacement with `114`. **ASSUMPTION:** Other administrators/automation also write this job; stale replacement can erase their exclusions, while skipping initial exclusion admits benchmark backup churn (`artifact/RUNBOOK.md:31`).
- SDN: `artifact/sdn.cfg:1–5`; removal/application at `artifact/RUNBOOK.md:81–83`. **ASSUMPTION:** Other cluster administrators write these maps; skipped application leaves stale runtime topology.
- Docker firewall/DOCKER-USER: no ruleset or explicit edit is supplied; the script claims “no host iptables/systemd changes” (`artifact/gb10_setup.sh:5`). **EXTERNAL:** Docker nevertheless creates publication firewall rules ([Docker port publishing](https://docs.docker.com/engine/network/port-publishing/)). Other writers and ordering remain **SPECULATIVE**; skipping an intended restriction leaves its boundary unenforced.
- CT201 GPU nodes: shared-device precedent is asserted at `artifact/RUNBOOK.md:25–28`; CT201 configuration/device state is absent. **ASSUMPTION:** Driver/device-management processes are competing writers; stale bindings can prevent production GPU recovery. No evidence requires modifying CT201’s nodes during this build.

**CONFIDENCE:** 0.88
**FALSIFIER:** An ownership/reconciliation record covers all four stores and demonstrates preserved unrelated state plus production GPU recovery.

**7. CLAIM:** A5 GB10 reruns replace containers rather than safely resume validated state.
**EVIDENCE:** `artifact/gb10_setup.sh:38–41` accepts any existing network named `cirubench`, without validating isolation/subnet, then removes all three containers; `:6,43–64` provides fail-fast execution without rollback; `:62–64` rewrites persistent home initialization. CF/Proxmox creation procedures are not supplied, so their double-create behavior remains **SPECULATIVE**.
**CONFIDENCE:** 0.96
**FALSIFIER:** Reconciliation validates existing objects and fault-injection demonstrates safe recovery at every creation boundary.

**8. CLAIM:** A5 teardown inventories the requested objects but contains a concrete CT-removal command defect and does not establish idempotent completion.
**EVIDENCE:** `artifact/RUNBOOK.md:80–86` covers CF app/policy/DNS/tunnel, both CTs, SDN vnet/zone, three GB10 containers, network, directory/home, image, and backup restoration. However, `:81` uses `pct stop 204 205 && pct destroy 204 205`. **EXTERNAL:** Proxmox defines each command with one `vmid` ([`PVE::CLI::pct`, command definitions](https://github.com/proxmox/pve-container/blob/master/src/PVE/CLI/pct.pm)). No absence checks or retry handling accompany the inventory.
**CONFIDENCE:** 0.98
**FALSIFIER:** The installed CLI accepts that exact syntax, and teardown succeeds twice with verified object absence and preserved unrelated exclusions.

**VERDICT INPUT**

Q1: **harden-first** — meaningful chokepoint, incomplete GB10 host boundary (2–3).  
Q2: **don’t** — withhold external onboarding until GPU exclusivity and bypass boundaries are enforced (1–3).  
Q3: **don’t** — readiness, reconciliation and teardown remain incomplete (4–8).  
Highest severity: **benchmark GPU access survives window closure alongside restarted production** (1).
