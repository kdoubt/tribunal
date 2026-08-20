# Flywheel data - de-identified run records (design + tool; intake not yet open)

Tribunal improves from evidence: which panel setups caught real problems,
which dissent proved right, which rounds were wasted. That evidence lives in
each user's own `retro.md` archive. This directory holds the **tooling and
schema** for pooling a de-identified slice of it someday - and a plain account
of why that has to be done carefully.

> **Read this first - it is de-identified, not anonymous.** A contribution
> would be a **public GitHub PR**: your account, the commit author, and the
> timestamps are attached and permanent. `flywheel-export` de-identifies the
> *run content* (strips secrets, free text, paths, dates, seat labels, and
> every claim/question); it does **not** hide *you*. Anyone who can read a PR
> can see who sent it. So this is "we can't tie a record back to your actual
> decision," not "we don't know who contributed."

## Status: the exporter is usable now; the intake is NOT open

You can run `flywheel-export` today as a **personal** tool - to see your own
runs as clean metadata, or to reduce a run before quoting it somewhere. But
there is **no `contributions/` folder and no submission channel yet**, on
purpose. Two conditions must hold before one opens (both from the adversarial
anonymity panel archived with this change):

1. **A reason to collect.** Aggregate analysis needs a corpus that doesn't
   exist yet (one public run). Privacy machinery for zero data is premature.
2. **A safe intake gate**, because a public PR folder has failure modes the
   exporter can't fix:
   - **Irreversibility.** A careless or hand-written PR that pastes raw
     `retro.md`, extra keys, or client text is public the moment it's opened
     and stays in git history *even if rejected*. Review can't un-ring that.
   - **Re-identification by joins.** Even clean records can fingerprint: exact
     counts + a rare `seat_vendors` mix is near-unique in a small pool and
     trivially linked to anyone who also blogs or posts a transcript of the
     same run.

   Before opening, the gate must include: **CI that allow-lists only the exact
   schema keys/enums and auto-fails any extra field or raw file**; a PR
   template that is literally "paste `flywheel-export` stdout, nothing else";
   **coarsened counts** (bucket to `0` / `1` / `2-3` / `4+`) and small-cell
   suppression (publish a cell only at k>=5); and a prominent, standing notice
   that **merged data is not erasable**.

## What the exporter emits (the whole schema)

One JSON object per completed run. Numbers that aren't a clean small integer
become `null`; categoricals are validated to a fixed vocabulary (unknown ->
`"other"`, raw text never echoed):

| Field | Type | Values |
|---|---|---|
| `schema_version` | int | `1` |
| `verdict_mode` | enum/null | `ship` \| `dont` \| `decide-after-check` \| `human-call` \| `other` |
| `rounds_run` | enum/null | `R0` \| `R0+R1` \| `R0+R1+R2` |
| `seat_vendors` | string[] | vendor families only: `codex` \| `grok` \| `claude` \| `gemini` \| `local` \| `other` |
| `seat_count`, `seat_deaths` | int/null | counts |
| `claims_total`, `agreed_r0`, `disputed`, `conceded`, `overturned`, `verified`, `dropped`, `surviving_dissent`, `resolved_by_oracle`, `resolved_by_debate`, `citations_dropped`, `citations_unverified` | int/null | counts (exact today - a known limitation; **bucketed before any intake opens**) |
| `verdict_held` | enum/null | `yes` \| `no` \| `no-signal` |
| `dissent_proved_right` | bool/null | did surviving dissent prove right? (a boolean - never *which vendor*, which is content `CONTRIBUTING.md` declines) |
| `had_template_delta` | bool/null | whether the run produced a template delta (the *text* is never included) |

Records are sorted by content before printing, so output order does not
reflect the order (or dates) of your panels.

## Never emitted

`verdict_date` / `outcome_date`, `cost_notes`, `friction`, `outcome`,
`missed_entirely`, the `template_delta` **text**, seat **labels/models/effort**,
Brier scores (a float is a precise fingerprint), file paths, and every claim,
question, and brief. The exporter never reads these into output - verified by
an injection test (a retro stuffed with secrets in every field emitted only
vocabulary values). That guarantee is about *content leakage*; it is separate
from, and does not solve, the re-identification and PR-identity issues above.

## Identity note

Tribunal is `runtime: none` in the sense that matters: **no hosted or
application runtime, no backend, no telemetry**. `scout` and `flywheel-export`
are optional local helpers that print to stdout and nothing else. This
directory adds no service - if an intake ever opens it will be files in git,
gated by CI, submitted by choice.
