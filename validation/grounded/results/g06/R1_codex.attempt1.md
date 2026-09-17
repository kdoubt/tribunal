1. **ATTACK**

- **Peer-1:** The fallback-directory risk is real, but `nofail` does not itself establish container startup behavior or directory writability. The proposal supplies options, not service dependencies (`artifact/unified-nfs-design.md:44–45`). Its falsifier also conflates retry behavior with startup ordering: `hard` does not establish a container-to-mount dependency. `nofail` can preserve production boot while explicit dependencies and readiness checks gate sandbox consumers. See [systemd.mount(5)](https://man7.org/linux/man-pages/man5/systemd.mount.5.html) and [nfs(5)](https://man7.org/linux/man-pages/man5/nfs.5.html). Confidence 0.85 is excessive for the asserted inevitable outcome.

- **Peer-2:** “Writes fail unless 0777” is unsupported. `all_squash` maps incoming identities, including shifted UIDs, to the anonymous identity at the server; shifted UID ≈101000 does not automatically remain the server authorization identity. See [exports(5)](https://man7.org/linux/man-pages/man5/exports.5.html). Client traversal permissions, server ownership, ACLs and actual UID mappings still require testing. The proposal’s claim that binds “run as that uid” is indeed inadequate (`artifact/unified-nfs-design.md:46–47`), but neither guaranteed EPERM nor mandatory world-writability follows. The proposed touch probe is useful; its outcome is unknown.

- **Peer-3:** The potential severity is justified; the asserted path is unproven. CT 206’s network is unspecified (`artifact/unified-nfs-design.md:27–28`), and `bridge=vmbr0` alone establishes neither effective NAS RPC reachability nor successful backup access. Conversely, one failed mount is insufficient evidence of containment. Require routing/firewall inspection and harmless RPC/export-access tests, including helper paths. The wildcard, unsquashed backup exports are documented (`artifact/synology-exports.txt:2–3`), but “full read/write of all backups” remains conditional. The existing helper-compromise route also matters (`artifact/RUNBOOK.md:121–127`).

- **Peer-4:** Local watchdog files establish a necessary boundary, not A5 satisfaction. The contract promises independent recovery (`artifact/CONTRACT.md:14–16,48–59`); it does not show that recovery avoids NFS-dependent logging, cleanup, process waits or restart dependencies. Even a stale heartbeat only triggers recovery—it does not prove recovery completes. The falsifier is too narrow: a NAS-loss test that stalls production restoration disproves the claim without relocating any file. Confidence 0.8 is unjustified without scripts or fault-injection results.

2. **CONCEDE**

Preserve mount readiness gates and protection against local-directory fallback. Preserve explicit UID/permission validation; binds do not automatically align identities. CT 206 must have specified, verified isolation because the existing backup exports make unintended NAS access catastrophic. Keep lock, budget state, window flags and heartbeats local exactly as documented (`artifact/CONTRACT.md:21–29`).

3. **REVISE**

**Own-5:** Replace the unconditional `nofail` recommendation with: “Use `hard,vers=3,proto=tcp,_netdev`; retain `nofail` only with explicit dependencies and readiness gates that prevent sandbox consumers using absent storage while allowing production boot independently.” Confidence **0.94 → 0.95**.

**Own-8:** No revision; **0.93 retained**. Local control files support the architecture, but only NAS-loss tests establish timely, healthy production restoration.

Other claims retain their stated uncertainty and acceptance probes.

4. **VERDICT INPUT**

Q1: **redesign** — trusted local queue authority, separately controlled shared workspace and immutable models; retain RTX-host placement with explicit containment.

Q2: **build-with-changes** — enforce dedicated-export restrictions and verify every sandbox/helper access boundary.

Q3: **redesign** — require independent recovery, mount gates and NAS-loss acceptance tests.

Highest severity: **conditional backup compromise through unintended NAS RPC access**; the strongest established design defect remains party-writable authoritative queue state.
