Framings considered: topology simplicity; containment after session compromise; operational lifecycle safety. I choose containment plus lifecycle safety because the decision criteria require both, and the documented window closure restores production while retaining bench GPU access (`artifact/RUNBOOK.md:67–75`).

1. **CLAIM:** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
   
   **EVIDENCE:** `artifact/ct205.conf:8` attaches RTX only to `cirub0`; `artifact/sdn.cfg:1–5` defines its simple zone. `artifact/bastion-nftables.conf:14,19–24` drops forwarding and restricts private IPv4 output. Conversely, `artifact/gb10-docker-inspect.txt:2–3` shows two dual-network intermediaries. Bastion output defaults to accept and its destination deny is IPv4-only (`artifact/bastion-nftables.conf:16,24`). **SPECULATIVE:** usable LAN IPv6 is not evidenced.
   
   **CONFIDENCE:** 0.95
   
   **FALSIFIER:** Effective network policy demonstrates that every bench path traverses equivalent restrictions, including IPv6 and GB10 host access.

2. **CLAIM:** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
   
   **EVIDENCE:** `artifact/cloudflare-objects.json:5,18–20,28–32` configures the tunnel and owner-email allow policy. `artifact/bastion-nftables.conf:8` accepts SSH without source/interface restriction; `artifact/ct204.conf:8` supplies its LAN address. `artifact/gb10-docker-inspect.txt:3` publishes LAN port 2222. `artifact/gb10_setup.sh:48–50` supplies no source restriction. `artifact/RUNBOOK.md:42–46` installs the same party’s credentials along these paths. **SPECULATIVE:** upstream controls might block direct LAN entry; their configuration is absent. Before onboarding, require negative direct-entry tests and fresh allowed/disallowed identity tests.
   
   **CONFIDENCE:** 0.94
   
   **FALSIFIER:** Direct LAN SSH is blocked by demonstrated effective controls while the approved Cloudflare route succeeds.

3. **CLAIM:** The highest-severity demonstrated design hole is retained guest GPU access when production resumes, allowing benchmark workloads to compete with production outside the accepted window.
   
   **EVIDENCE:** `artifact/ct205.conf:14–20` grants persistent shared-device access; `artifact/gb10_setup.sh:53–55` grants all GPUs. Closure only restarts RTX production and reduces GB10 memory before restarting its production containers (`artifact/RUNBOOK.md:69–75`); neither closure stops the bench environment or revokes devices. The runbook explicitly acknowledges shared-driver crash impact (`artifact/RUNBOOK.md:95–96`). Require bench shutdown and verified GPU release before production restart.
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** An enforced closure mechanism terminates guest GPU workloads and removes their ability to restart before production restoration.

4. **CLAIM:** GB10’s internal network does not establish isolation from services on the GB10 host itself.
   
   **EVIDENCE:** `artifact/gb10-docker-inspect.txt:4` records gateway `172.30.99.1`; `artifact/gb10_setup.sh:38–39` creates an internal network without isolated gateway mode. **EXTERNAL:** [Docker’s gateway-mode documentation](https://docs.docker.com/engine/network/port-publishing/#gateway-modes) states that internal-network containers can access host services bound to the bridge address or wildcard addresses. The only direct negative probe targets another LAN address (`artifact/gb10_setup.sh:69`). **SPECULATIVE:** exploitable host listeners, host escape, and another tenant’s data exposure remain unproved; host INPUT policy and listener inventory are unavailable.
   
   **CONFIDENCE:** 0.90
   
   **FALSIFIER:** Effective host INPUT filtering blocks bench-originated host-service access, verified against actual wildcard/bridge listeners.

5. **CLAIM:** A1 readiness is incomplete: restart settings exist for some dependencies, but end-to-end readiness and bounded recovery are not established.
   
   **EVIDENCE:** Cloudflared uses notification readiness, a 15-second startup timeout, and five-second failure restart (`artifact/bastion-cloudflared.service:7–11`), but no separate tunnel-health probe is supplied. Squid, socat, and GB10 sshd use `unless-stopped` (`artifact/gb10_setup.sh:43–58`); setup then sleeps three seconds (`:60`). GPU probe failure is ignored (`:68`), and HTTP status codes are printed without assertions (`:70–71`). Tinyproxy’s 600-second timeout is a service setting, not startup readiness (`artifact/bastion-services.conf:12`). Restart/readiness policies for tinyproxy, dnsmasq, bastion sshd, and RTX sshd are absent from their supplied snippets (`artifact/bastion-services.conf:1–21`; `artifact/ct205-services.conf:1–10`).
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** Additional deployment units and execution records demonstrate bounded retries plus authenticated SSH, DNS, proxy, and tunnel readiness checks.

6. **CLAIM:** A2–A3 validation does not establish convergence or cache-independent DNS/Access/TLS correctness.
   
   **EVIDENCE:** `artifact/RUNBOOK.md:16–18` describes a CNAME but supplies no DNS response, TTL, certificate, or Access test transcript; its explicit reachability assertion is `:30`. `artifact/gb10_setup.sh:43–60` starts containers before completing network attachments, then uses a fixed sleep. **ASSUMPTION:** DNS caching, Access propagation, and concurrent Docker firewall changes can produce transient results; their occurrence here is unproved. Require repeated fresh-client allow/deny tests, authoritative and recursive DNS checks, TLS verification, and inspection of effective Docker rules after convergence.
   
   **CONFIDENCE:** 0.96
   
   **FALSIFIER:** Timestamped fresh-session validation establishes those properties after propagation and attachment completion.

7. **CLAIM:** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
   
   **EVIDENCE:** Backup exclusions are `114,205` (`artifact/backup-job.json:1`), while teardown overwrites them with `114` (`artifact/RUNBOOK.md:86`): omission leaves stale state; overwriting can erase intervening exclusions. SDN spans two cluster files (`artifact/sdn.cfg:1–5`), with deletion/application required (`artifact/RUNBOOK.md:81–83`): omission leaves configuration behind. GB10 explicitly avoids host firewall changes (`artifact/gb10_setup.sh:5`); DOCKER-USER contents are unavailable. **EXTERNAL:** [Docker documents](https://docs.docker.com/engine/network/firewall-iptables/) Docker-managed rules and administrator restrictions in DOCKER-USER. **ASSUMPTION:** administrators/automation are other backup/SDN/firewall writers; identities are unknown. CT201’s actual device configuration is unavailable: no stable position on required node repair; **SPECULATIVE** stale nodes could prevent GPU recovery. Shared-device and driver dependencies are acknowledged at `artifact/RUNBOOK.md:25–28`.
   
   **CONFIDENCE:** 0.89
   
   **FALSIFIER:** Serialized, ownership-aware updates preserve concurrent changes, with CT201 device and inference recovery verified.

8. **CLAIM:** A5 lists the intended teardown inventory but does not provide reliable idempotent resume or executable complete teardown.
   
   **EVIDENCE:** `artifact/RUNBOOK.md:80–86` names CF app/policy/DNS/tunnel, both CTs, SDN vnet/zone, three GB10 containers, network, image, home tree, and backup restoration. However, `:81` passes two IDs to each `pct` command; **EXTERNAL:** [Proxmox syntax](https://pve.proxmox.com/pve-docs-9-beta/pct.1.html) accepts one. Setup trusts any existing network and suppresses removal errors (`artifact/gb10_setup.sh:38–41`), then destructively recreates containers. GB10 is still marked unbuilt (`artifact/RUNBOOK.md:32`) despite live inspection. **SPECULATIVE:** CT204 backup copies may survive; it is not excluded (`artifact/backup-job.json:1`).
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** Corrected procedures successfully resume partial builds and repeatedly remove all engagement state, with retained backups explicitly reconciled.

**VERDICT INPUT**

Q1: **harden-first** — useful architecture, incomplete chokepoint (claims 1, 4).  
Q2: **harden-first** — identity bypass paths and unresolved host-service exposure (claims 2, 4).  
Q3: **don’t** ship the current lifecycle procedure (claims 3, 5–8).  
Highest severity overall: **production resumes without revoking guest GPU access** (claim 3).
