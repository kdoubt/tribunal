# Decision G07 - a benchmark job queue for an untrusted external party on shared production GPUs

## Artifact(s)

All under `./artifact/` (read them directly; cite `file:line`):

- `queue-design.md` - THE PROPOSAL under review (Design A window-batch; rejected B/C).
- `RUNBOOK.md` - the existing access architecture this extends (bastion,
  islands, window scripts, security posture, known residual).
- `window-rtx.sh`, `window-gb10.sh` - the existing health-gated window
  open/close scripts the proposal builds on (confirm VRAM freed on open; prod
  HEALTHY not just active on close).

Context (USER-FACT): gated remote GPU access already exists for an untrusted
external party ("the external party", a hardware LLM-inference benchmarking lab) to two GPUs
shared with production, with no hardware partition (no MIG), so it is strictly
prod-XOR-the external party per GPU. Access is CF-Access-gated SSH into an isolated sandbox
per GPU (RTX -> LXC CT 205 on a no-uplink island; GB10 -> a docker container
on an internal network). Production on the RTX is `vllm.service` (a ~92 GB
model, minutes-scale cold start) plus `vllm-embed.service`; on the GB10, three
vLLM containers. The goal is a QUEUE so the external party can submit benchmark jobs that
run one at a time on the full GPU, unattended, without an operator manually
opening and closing a window per job, and without letting the external party
disrupt or DoS production.

## Question under review

- **Q1 Architecture.** Is Design A (window-batch: operator-gated window, an
  arbiter drains the external party's FIFO queue one job at a time, prod down once per
  window not per job) the right shape? Is per-job preemption (B) correctly
  rejected given minutes-scale cold start? Is rejecting reuse of the estate's
  NATS spine (C) for an external party correct, or is a subject-scoped
  separate NATS user actually safer than a local spool? Propose a better
  middle path if one exists.
- **Q2 Security / trust boundary.** the external party controls only job specs (a `cmd`
  run as user `the external party` inside his sandbox) and submission timing. Enumerate
  every way he could still harm production or escape: DoS the arbiter or
  prod, fill spool disk, craft a job that outlives its `timeout`, exhaust the
  sandbox to affect the host, or influence the preemption decision. Name the
  highest-severity real hole. Is "the operator opens the window, not the external party"
  sufficient to keep preemption off the untrusted party?
- **Q3 Systems / operational.** Will this break prod or fail to recover? See
  A1-A5.

## Operational readiness (mandatory, A1-A5)

- **A1** The arbiter starts/stops prod via the window scripts, which
  health-gate. Does the arbiter verify a job's sandbox is ready, bound each
  job with a timeout, and handle a job that hangs? Per-job AND per-window
  caps? What retries on a failed prod restart?
- **A2** After `window close` the script waits for prod HEALTHY. What if it
  never goes healthy? Does the arbiter leave prod DOWN while it moves on? Any
  state (queue dir, "window open" flag, current-job marker) that two actors
  write or that a reader could observe mid-transition?
- **A3** How does the arbiter know prod is really back (not just `active`)
  before it reports the window closed, and that the GPU is really free before
  it starts a job, given the GB10 reports VRAM `used=[N/A]`?
- **A4** Every store/actor that must change, who else writes it, symptom if
  skipped: the window flag, the queue spool, prod services/containers, the
  GPU, the max-downtime budget counter.
- **A5** If the arbiter or its host crashes or reboots mid-drain, does it (a)
  double-run a job, (b) leave prod DOWN indefinitely, or (c) resume cleanly?
  Is there a dead-man that restores prod if the arbiter dies? Does a reboot
  re-open a window that was closed?

## Precedent (contested evidence)

- The estate has a cross-node dead-man watchdog (ping + executor-halt ->
  email, 2-minute timers), a pattern to copy; known gap: death of its own host
  silences alerts.
- Co-resident vLLM crashloop when GPU memory fractions summed above 1.0;
  "active is not healthy".
- A read-modify-write by two cross-process writers with no lock caused a lost
  update; applies to any shared flag/counter here.
- A reboot during an hourly backup locked a live-trading CT for 20 h; check
  cluster tasks before any disruptive op.
- The estate's NATS spine has a per-role user model and a hard custody
  boundary (confidential content never transits it); this is the backbone
  Design C would reuse.
- The existing window scripts already health-gate, and the RTX one refuses to
  run during a backup.

## Decision criteria (owner-supplied)

(1) production is never left DOWN unattended and is never DoS'd by the
external party; (2) the external party gets the full GPU for real, unattended, one job at a
time; (3) crash/reboot recovery is safe (fail toward prod-UP); (4) the
simplest thing that achieves 1-3. Tie-break: any path that can leave prod DOWN
unattended, or let the external party trigger or extend preemption, outranks all convenience.

## Constraints

- No MIG; prod-XOR-the external party only. the external party untrusted, sandbox-confined, no host/root,
  no prod control.
- The arbiter runs trusted-side (the RTX host for the RTX; the GB10 host as
  its login user, who is in the docker group with no passwordless sudo). It
  may reuse `window-*.sh`.
- The ~92 GB model's cold start is minutes; per-job flapping is presumed
  pathological (challenge if wrong).

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
