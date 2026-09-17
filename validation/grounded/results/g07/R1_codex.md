1. **ATTACK**

- **Peer-1:** The categorical NATS rejection exceeds the evidence. `artifact/queue-design.md:52–56` states the proposal’s rationale, not a demonstrated security comparison. The confidentiality rule at `artifact/queue-design.md:17–18` does not establish protection against external resource exhaustion. Neither broker resource isolation nor spool admission controls are specified. A trusted spool is a reasonable default, but “strictly safer” remains unproved. The falsifier also unfairly requires NATS to have *no* additional coupling cost; a security advantage could justify added complexity.

- **Peer-2:** The missing recovery mechanism is established; its **highest-severity ranking** is under-argued. Even a functioning independent closer cannot establish safe restoration through the existing scripts: neither terminates the benchmark sandbox before restarting production (`artifact/window-rtx.sh:30–41`; `artifact/window-gb10.sh:35–48`). Hostile GPU activity can therefore defeat restoration without an arbiter crash. Also, daemon death and host reboot are different cases: the artifacts do not establish every production boot/restart policy. The strongest supported conclusion is that unattended recovery is unspecified, with a concrete daemon-death failure path.

- **Peer-3:** The claim identifies a credible failure mode, not a demonstrated extension exploit. The proposal explicitly requires an absolute duration cap and leaves remaining jobs queued (`artifact/queue-design.md:43–47`); replenishing the FIFO or lying about `est_seconds` does not inherently defeat that rule. Sharing an arbiter establishes dependence on its responsiveness, not attacker control over its timer. A blocked-read exploit needs an implementation or probe. A separate closer is valuable, but calling unchanged `close` still leaves benchmark GPU access intact.

- **Peer-4:** Several risks are bundled under stronger certainty than their evidence supports. GB10 host storage placement does not establish absent quotas or a shared production filesystem (`artifact/RUNBOOK.md:38`). A 115g cap establishes limited headroom, not proven host starvation (`artifact/window-gb10.sh:30–31`). The RTX disk claim correctly remains an assumption. Passwordless sandbox sudo supports the cleanup concern, not host-root escape (`artifact/RUNBOOK.md:23–28`). Finally, one successful driver-reset test cannot falsify the broader shared-driver risk.

2. **CONCEDE**

Trusted admission into immutable, bounded storage is a sound middle path. Subject permissions alone do not establish broker resource isolation.

An independent expiry/recovery mechanism is mandatory; operator-only opening does not guarantee timely restoration. Enforce deadlines independently of queue depth, estimates, and untrusted filesystem operations.

Detached or privileged sandbox descendants, storage exhaustion, host resource pressure, and shared-driver faults all require explicit treatment. The driver residual is directly documented (`artifact/RUNBOOK.md:103–105`). Recovery must fence benchmark execution before restoring production.

3. **REVISE**

**Own-2 — new text:** “Operator-controlled opening is insufficient because neither close script terminates the benchmark sandbox or revokes its GPU access before restarting production; this permits hostile activity to obstruct restoration, while the documented shared-driver residual prevents these artifacts from establishing the strict no-DoS guarantee.”

**Confidence: 0.99 → 0.98.** This narrows “unattainable” to what the reviewed architecture establishes. Missing GPU fencing remains my highest-severity issue; missing dead-man recovery is independently release-blocking. Own-1 and Own-3 through Own-8 remain unchanged.

4. **VERDICT INPUT**

Q1: **build-with-changes** — retain A, reject B, use bounded trusted admission; transport superiority remains unproved.

Q2: **redesign** — highest-severity overall: benchmark GPU access persists through close, enabling hostile interference with production restoration.

Q3: **build-with-changes** — require fenced recovery, independent expiry, durable state ownership, bounded operations, and crash/reboot validation.
