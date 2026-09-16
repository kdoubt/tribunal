# Decision G06 - a unified NFS-backed single-session workspace for an untrusted external party

## Artifact(s)

All under `./artifact/` (read them directly; cite `file:line`):

- `unified-nfs-design.md` - THE PROPOSAL under review.
- `RUNBOOK.md` - current bench-access and isolation posture (island, bastion, CF Access).
- `CONTRACT.md` - the existing queue design (spool, at-most-once, dead-man) this extends.
- `synology-exports.txt` - the live NAS export list (note existing `no_root_squash`).
- `gb10-nfs-precedent.txt` - the working NAS-NFS-on-GB10 mount to copy.

Context (USER-FACT): an untrusted external party ("the external party") has gated,
queue-only access to two GPUs (an RTX in an LXC on a no-uplink L2 island =
CT 205; a GB10 docker container on an `--internal` net), both shared with
production via a window-batch queue with an independent dead-man. The two
per-GPU code-server sessions are to be replaced with ONE unified code-server,
backed by a shared Synology NFS "warm" tier so large model files (tens of GB)
live on the NAS and are reused across jobs. The NAS is a shared production
appliance that also holds Proxmox backups and other tenants' exports. All three
hosts (the arbiter host, the RTX host, the GB10 host) can already reach it.

## Question under review

- **Q1 Architecture.** Is "one NFS share = workspace + models + queue spool,
  host-mounted then bind-mounted into each sandbox" the right shape? Is
  co-mingling the external party's RW workspace with the spool and models on one share
  sound, or should they be separate shares/tiers? Is the RTX host the right
  home for the unified code-server, or a blast-radius mistake?
- **Q2 Security.** Does host-mount + bind-mount truly preserve the RTX island
  and the GB10 internal-net isolation, or is there a path (the NFS server,
  other exports, `no_root_squash`, uid/all_squash mapping, symlink/`crossmnt`
  escape, traversal) by which the external party reaches other NAS data (prod backups,
  other tenants) or another host? Name the highest-severity real hole. Is
  exporting a share of a shared prod NAS into an untrusted party's
  environment defensible, and under exactly what export options?
- **Q3 Systems/operational.** A1-A5 below. NFS becomes a hard dependency for
  jobs, the arbiter's spool, and the external party's editor.

## Operational readiness (mandatory, A1-A5)

- **A1** Mount options (`soft` vs `hard`, `nofail`, `_netdev`, `timeo`,
  `retrans`): what is correct so a NAS blip degrades gracefully rather than
  hanging jobs, the arbiter, or code-server? Is there a readiness predicate
  that the mount is live before a window opens or a job runs?
- **A2** Is the queue's atomic-rename claim + at-most-once (CONTRACT.md) still
  correct over NFSv3 with multiple writers (arbiter + job in the sandbox):
  close-to-open consistency, attribute caching, `nolock`, stale filehandles,
  rename atomicity across clients?
- **A3** How to verify the export is correctly restricted (only the three host
  IPs, squashed) without trusting a single `showmount`? What proves the external party's
  sandbox cannot reach an unintended export?
- **A4** Every store/actor that must change, who else writes it, symptom if
  skipped: the NAS export config, each host's fstab/mount, the bind mounts
  into CT 205, the GB10 containers, and CT 206, the spool location move,
  uid/gid mapping.
- **A5** NAS reboot or mount drop mid-window: does the arbiter fail toward
  prod-UP and not corrupt the spool? Boot ordering? Teardown that unmounts
  and removes the export?

## Precedent (contested evidence)

- The GB10 already NFS-mounts a NAS share read-only (`gb10-nfs-precedent.txt`):
  `vers=3, soft, timeo=150, retrans=3, _netdev, nofail, nolock, ro`.
- Live NAS exports (`synology-exports.txt`) use `no_root_squash` and
  whole-subnet/`*` clients on the backup exports: the anti-pattern not to copy.
- Island CT 205 isolation is "LAN unreachable by construction"; a mount must
  not become a new route.
- Shared mutable state with two cross-process writers and no lock caused a
  lost update. "Active is not healthy": a mount listed in `mount` output does
  not mean it serves.
- The queue dead-man restores prod on window overrun or arbiter death and must
  still hold if NFS is what died.

## Decision criteria (owner-supplied)

(1) no path from the untrusted party to other NAS data or another host; (2)
an NFS/NAS failure never leaves production down and never corrupts the queue;
(3) large models usable across both GPUs without per-job shipping; (4) the
simplest shape that achieves 1-3. Tie-break: any cross-tenant/data-exposure
path, or any NAS failure that leaves prod down or double-runs a job, outranks
all convenience.

## Constraints

- Cannot re-VLAN; the RTX island stays no-uplink; the GB10 sandbox stays internal-net.
- The NAS is shared prod: export changes must not touch existing exports.
- code-server host = the RTX host (owner's choice); the arbiter stays on the
  arbiter host; dead-man per GPU host.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <artifact file:line, verbatim span, or the exact probe that would settle it; else ASSUMPTION / SPECULATIVE (name it) / EXTERNAL (source)>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Plus **VERDICT INPUT**: one line per question (build-as-proposed /
build-with-changes / redesign) and the single highest-severity issue overall.
Maximum 1200 words.
