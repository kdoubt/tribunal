I'll read the cited artifacts and verify each disputed claim against the frozen brief before writing the cross-examination.1. **ATTACK**

**Peer-1.** Shape/B-reject is already settled; the live part is transport. “Artifacts do not establish local-spool > isolated NATS” is a true *meta* claim, but **0.92 plus “either spool or broker”** is not a position. `queue-design.md:52–56` rejects *reusing* `jobs.batch-infer.>`, not “any JetStream account.” Criterion (4) plus custody (`:17–18`) still pick a host-owned drop dir over a new account, ACLs, and arbiter-on-spine coupling. In-sandbox `/scratch/queue/incoming` (`:33–34`) is unsafe *regardless* of NATS; “either” leaves that hole. The “twice” nit (`:24–27`) is right (per-job is **one** restart, not two) and does not revive B. Falsifier is hand-wavy (“configurations”).

**Peer-2.** Bounds/ingest gap is real (`:30–36,61–63`). **0.98** is high for an absence claim; they list surfaces, not a ranked hole, and do not show host escape. `RUNBOOK.md:109–115` is a **pre-existing helper residual** (needs GB10 root; the external party is in `--internal` `the external party-bench`). It is not a Design A submission bug unless they show a path from `cmd` to squid/socat. “Not demonstrated host escapes” undercuts treating this as the security headline.

**Peer-3.** Pointers are exact: close never stops CT 205 / `the external party-bench` (`window-rtx.sh:30–41`, `window-gb10.sh:35–48`); driver residual is `RUNBOOK.md:103–105`. Two issues are **conflated**: (i) leftover the external party GPU consumers at prod-start (crashloop precedent; GB10 only `docker update` to 20g, result discarded); (ii) a driver crash that already happened. (i) is Design A’s close-path hole and is fixable (stop sandbox / fence GPU **before** start). (ii) is the existing shared-driver residual. **0.99** that (ii) is *the* highest hole, and that “never DoS” is unattainable, overreaches: they never compare to A5 (no dead-man; open already stopped prod). A closer that only runs these scripts still fails (i).

**Peer-4.** Thresholds and fail-open backup are real (`window-rtx.sh:11,19–20,26`; `window-gb10.sh:13,29–31`). **0.97** slightly overclaims “exclusive full-GPU”: `<4000 MiB` / `exited+≥60G` are heuristics, not a uniqueness proof. They omit helper hangs (no ssh/curl deadline) and arbiter policy on close `exit 1` — that is the A2/A3 failure mode, not the probe itself.

2. **CONCEDE**

- A is the right shape; B stays rejected (N restarts vs one per window).
- In-sandbox spool is not shown safer than an isolated NATS *account*; admission must be trusted-side, bounded, copy-off the external party’s FS; `est_seconds` untrusted.
- Close does not revoke GPU; leftover bench + shared driver can block HEALTHY restore.
- Open/close probes are not exclusive-GPU and not fail-closed (`docker update` unchecked; backup `2>/dev/null|grep`).
- Independent wall-clock closer + host-enforced sandbox stop before prod start.

3. **REVISE**

- **Own-1:** no revision. “Twice” is source-sloppy; N vs 1 cold-starts still pathological (0.85).
- **Own-2:** *Rejecting reuse of `jobs.batch-infer` (C) is correct; middle path is operator-gated windows with trusted-side admission the external party cannot write. Artifacts do not prove a drop dir strictly safer than a separate NATS account; in-sandbox spool is unsafe either way.* **0.80→0.72**
- **Own-3:** *Operator-open blocks starting preemption, not extending it (drain/`est_seconds` in the same arbiter) and not GPU occupancy after close (scripts never stop the sandbox).* **0.85→0.90**
- **Own-4:** *Highest hole is restore-unsafe: no dead-man after `open`, and `close` starts prod without fencing the external party off the GPU — arbiter death leaves prod DOWN; a close that runs can still leave prod not HEALTHY unattended.* **0.90→0.88** (rank contested, both mechanisms cited).
- **Own-5:** no revision (A1 still unmet, 0.85).
- **Own-6:** *Close `exit 1` with no retry plus non-exclusive/fail-open probes (4 GiB, 60G, unchecked `docker update`, backup grep) means the arbiter can mark restored while GPU is not exclusive and prod is not HEALTHY.* **0.88→0.90**
- **Own-7:** no revision (0.82); Peer-2’s ingest list is the missing table.
- **Own-8:** no revision of the harm list; RTX host-disk-fill stays ASSUMPTION. GPU leftover at close outranks spool-fill.

4. **VERDICT INPUT**

Q1 build-with-changes (A, not B; trusted-side admission, not in-sandbox spool or prod NATS). Q2 build-with-changes (independent closer; stop/fence sandbox before prod start; bound ingest). Q3 build-with-changes (dead-man fail-toward close; retry/page on non-200; one-writer state). Highest: restore-unsafe — no dead-man and close does not revoke GPU.
