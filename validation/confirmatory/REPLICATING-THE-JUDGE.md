# Replicating the judge - a free, bring-your-own-key second scoring

The confirmatory study's pre-registered decision rule is literally incomplete
until someone runs it: lift is declared only if the primary endpoint "is
positive and holds under **both** the primary judge and the designated
non-OpenAI second judge" ([`PROTOCOL.md`](PROTOCOL.md), "Pre-registered
primary endpoint & decision rule") - and the second judge was never run (see
[`RESULTS.md`](RESULTS.md), Limitations). This page is the recipe for running
it yourself, at $0, with no infrastructure beyond an API key you already own
or a local model runner. Either outcome - corroborating the no-lift finding
or overturning it - is exactly the independent replication this repo says it
wants.

> **This page is about the judge only. Free-tier models are NOT Tribunal
> debate seats.** A debate seat must be frontier-class and genuinely
> heterogeneous (`core/CONTRACT.md`, Roles; `core/METHODOLOGY.md`, "Model
> selection - where tiering is safe and where it isn't"): a cheap seat adds
> correlated incapacity, not diversity, and pairing same-lineage models is
> identity theater. Evaluation *judging*, by contrast, is one of the
> mechanical stages METHODOLOGY explicitly licenses for any capable
> cheap/fast configuration. Staffing seats is out of scope here.

## What you need

- **A judge model outside the OpenAI lineage** - the protocol designates the
  Meta Llama or Alibaba Qwen families ([`PROTOCOL.md`](PROTOCOL.md), Judge).
  Any OpenAI-compatible chat-completions endpoint works: a hosted gateway's
  free tier (several exist; e.g. Groq's) or a local runner (vLLM, Ollama).
  Pick whatever current, strongest Llama- or Qwen-family model your endpoint
  serves - this page deliberately pins no model names, because gateway
  rosters change without notice and a pinned alias rots.
- **Your own key** (or a local endpoint needing none). This repo never ships
  credentials and never proxies API access - bring your own.
- The published inputs, all in this repository (see below).

## The recipe

For each of the ten ambiguous decisions (`c02`, `c11`-`c19`):

1. **Inputs:** the brief at `decisions/cNN-brief.md`; the two memos at
   `results-raw/cNN/solo-codex.md` and `results-raw/cNN/panel.md`.
2. **Prompt:** the verbatim judge prompt published in
   [`results-raw/AUDIT.md`](results-raw/AUDIT.md) (section 3), with your
   endpoint substituted. Temperature 0.
3. **Blind it:** assign the two memos to "Memo X" / "Memo Y" per decision -
   randomize (or counterbalance) the order and **record the mapping before
   judging**, as the original run did (`results-raw/AUDIT.md`, section 1).
4. **Decode and total:** apply your recorded mapping to the scores; compare
   against the original judge's decoded table in `AUDIT.md` (section 2):
   aggregate solo 118 vs panel 103, forced choice solo 9 / tie 1 / panel 0.

The same procedure re-scores the pilot-02 memos (`../results/pilot-02/`,
which publishes its own blind-order record).

## Reporting

Open an issue or PR with: judge model + endpoint type (hosted/local), your
per-decision scores and blind mapping, and totals. Per the pre-registered
rule, "no lift" stands unless the panel wins on aggregate under **both**
judges - so a second-judge result is decisive in a way no amount of internal
re-running can be. Contributions of this shape are the repo's most-wanted
(see [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)).
