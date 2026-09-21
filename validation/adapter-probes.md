# Adapter probes - the evidence behind the adapter guidance

Every operational rule in `adapters/` should be traceable to something
observed, not assumed. This page publishes the probes behind the rules that
were changed after a real run, so a reader can re-run them instead of taking
the adapter's word for it.

**Scope and status.** These are *operational* facts about specific CLI
versions on one host, dated because vendor behaviour changes. They are **not**
a benchmark and **not** a "which model is better" comparison - `CONTRIBUTING.md`
declines that, and seats are deliberately interchangeable. Where two CLIs
differ below, the finding is that adapters must not *assume* one shape, not
that either CLI is superior. This page is **not** lift evidence and changes
nothing in [`../README.md`](../README.md)'s no-lift position; for run
aggregates see [`FIELD-RECORD.md`](FIELD-RECORD.md).

**Anonymity.** These probes carry no panel content: the artifact read is this
repo's own `core/CONTRACT.md`, and the arithmetic probe is the smoke test from
the adapters. Nothing here touches a private brief, a client artifact, or a
run's claims, so the transcripts are reproduced verbatim.

Captured 2026-09-21T02:33Z · `codex-cli 0.153.2` · `grok 1.0.13 (5e9a58528b76)`
· origin: the 2026-09-20 panel archived by the maintainer, whose own read-test
produced a false positive.

## Probe 1-3: a seat's read path may *be* shell

`core/CONTRACT.md:21` is the known span; the expected reply is its text.

| # | Command (cwd = a non-git dir holding `CONTRACT.md`) | Observed |
|---|---|---|
| P1 | `codex exec -s read-only --skip-git-repo-check "Read CONTRACT.md … reply with line 21 verbatim … **Use your built-in file-read tool, not shell commands.**"` | *"I don't have a built-in file-read tool available here, so I can't read line 21 without using shell commands."* — **read refused** |
| P2 | same, **tool instruction removed** | `Seats MUST be genuinely heterogeneous processes - different vendors or at` — **correct** |
| P3 | `grok --permission-mode plan -p` with **P1's exact constrained prompt** | `Seats MUST be genuinely heterogeneous processes - different vendors or at` — **correct** |

**What it shows.** P1 vs P2 isolates the instruction: same CLI, same file, same
span, one sentence removed. P3 shows the constrained prompt is not wrong in
general - it succeeds on a CLI that has a native read tool. So the defect is the
*assumption* that every seat has a non-shell read path. Under `-s read-only`,
Codex's read path is shell inside the sandbox; instructing that seat away from
shell removes its only way to read.

**Why it mattered.** The read-test in `adapters/claude-code/SKILL.md` exists to
detect a host that intercepts reads, and it reads a refusal as that
interception. P1 therefore produces a false positive: the operator diagnoses a
broken host and starts fixing something that is not broken. This happened to the
maintainer on 2026-09-20 before the probe isolated it.

**Rules this supports.** `adapters/shell/README.md` "Silent seat killers" 1 and
the `SKILL.md` read-test bullet: ask for the span, do not prescribe the tool;
the read-only mode is the control.

## Probe 4-5: `--skip-git-repo-check` and the trusted directory

| # | Command | Observed |
|---|---|---|
| P4 | `codex exec -s read-only "What is 8347 multiplied by 26? …"`, cwd = **non-git dir** | `Not inside a trusted directory and --skip-git-repo-check was not specified.` · **exit 0** · no answer |
| P5 | the same command, cwd = **a git repo** | `217022` — correct |

**What it shows.** The failure is conditional on the working directory, not on
the flag alone: inside a git repo the flag is a no-op, so a panel run from a
project root never sees this. It bites where a panel is driven from a scratch or
panel directory. The shape is worth naming because it is *silent*: exit status 0
with plausible-looking stderr and no position - the smoke test catches it only
because the expected answer is absent, which is exactly why the adapters require
a **verifiable** smoke test rather than "reply OK".

**Rules this supports.** The `SKILL.md` seat-command example now carries
`--skip-git-repo-check`, matching `adapters/shell/README.md:17` and `:124`,
which have shipped it since their examples block.

## Reproducing

```sh
mkdir /tmp/probe && cd /tmp/probe && cp /path/to/tribunal/core/CONTRACT.md .
# P1 / P2 / P3 / P4 above, verbatim; then P5 from any git repo.
```

Expect divergence over time: these are dated vendor behaviours. A probe that no
longer reproduces is a reason to re-date or retire the rule it supports, not to
assume the rule is wrong.
