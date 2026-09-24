# Replicating the judge - a free, bring-your-own-key second scoring

**What this re-score does, precisely.** It re-runs the *ambiguous-arm*
comparison - solo memo vs panel memo on `c02` and `c11`-`c19` - under a judge
outside the OpenAI lineage, testing whether the published preference (aggregate
solo 118 vs panel 103; forced choice solo 9 / tie 1 / panel 0) survives a
different judge.

**What it does not do.** It does not complete the study's pre-registered primary
endpoint. That endpoint is a *different quantity* - mean `A_panel - A_2seat`
paired across all decisions - which `RESULTS.md` (Limitations) records was never
computed in pre-registered form, c09 having ended in a split to which the
pre-registration assigns no numeric value. Nor can this re-score *declare* lift:
`PROTOCOL.md` gates that conjunctively on both judges, and the primary judge
already has the panel losing this comparison, so that gate is closed whatever a
second judge returns. A reversal here would establish **judge sensitivity** of
the 118-vs-103 result - which is worth knowing and worth filing - not registered
lift.

The designated non-OpenAI second judge **has since been run once** by the maintainer
(2026-09-22, Alibaba Qwen family - see [`SECOND-JUDGE.md`](SECOND-JUDGE.md)). That does
not close this page: a single maintainer-run scoring is not independent replication, and
a judge's own run-to-run variance is unmeasured. What this recipe asks for is a re-score
by someone who is not the maintainer. It costs $0 beyond an API key you already own or a
local model runner.

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

## Worked first step - copy, paste, run

Do `c02` first to prove your endpoint and prompt assembly, then repeat the loop
for `c11`-`c19`. Nothing here is a runner: it is one request you can paste.

```sh
# Your endpoint. Any OpenAI-compatible chat-completions URL: a hosted gateway's
# free tier, or a local vLLM/Ollama server. Pick a current Llama- or Qwen-family
# model - PROTOCOL.md designates the family, not a pinned alias.
export ENDPOINT="https://your-gateway.example/v1/chat/completions"
export API_KEY="sk-..."   # omit the Authorization header below for a local endpoint
export MODEL="<a current Llama- or Qwen-family model your endpoint serves>"

cd validation/confirmatory

# Build the request exactly as the original run did (AUDIT.md section 4): the
# published prompt is a TEMPLATE - $BRIEF, $X and $Y are substituted into it and
# sent as ONE message per decision, temperature 0, max_tokens 30000.
#
# Blind the memos BEFORE judging and record the mapping (AUDIT.md section 1):
# here X = solo, Y = panel. Counterbalance or randomize across decisions and
# write down which way round each went - you need it to decode.
#
# stdlib python3 only - no jq, nothing to install.
python3 - c02 > /tmp/c02-req.json <<'REQ'
import json, re, sys, os, pathlib
d = sys.argv[1]
audit = pathlib.Path("results-raw/AUDIT.md").read_text()
tmpl = re.search(r"^You are an impartial senior.*?(?=^```$)", audit, re.S | re.M).group(0).rstrip()
prompt = (tmpl.replace("$BRIEF", pathlib.Path(f"decisions/{d}-brief.md").read_text())
              .replace("$X",     pathlib.Path(f"results-raw/{d}/solo-codex.md").read_text())
              .replace("$Y",     pathlib.Path(f"results-raw/{d}/panel.md").read_text()))
assert "$BRIEF" not in prompt and "$X" not in prompt and "$Y" not in prompt, "template not fully substituted"
json.dump({"model": os.environ["MODEL"], "temperature": 0, "max_tokens": 30000,
           "messages": [{"role": "user", "content": prompt}]}, sys.stdout)
REQ

curl -sS "$ENDPOINT" -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_KEY" -d @/tmp/c02-req.json
```

Temperature 0. Record the raw reply, decode X/Y with your mapping, and compare
against the original judge's decoded table in
[`results-raw/AUDIT.md`](results-raw/AUDIT.md) (section 2): aggregate solo 118 vs
panel 103, forced choice solo 9 / tie 1 / panel 0. **All ten decisions are needed
for the aggregate** - `c02` alone is a practice run, not a result.

If your endpoint rejects the request, the usual causes are a model alias your
gateway no longer serves, a missing `Content-Type`, or a local server that wants
no `Authorization` header at all.

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
per-decision scores and blind mapping, and totals. What your result settles:
whether the published ambiguous-arm preference is stable across judges, or is an
artifact of the one judge that produced it. It does not complete the primary
endpoint and cannot declare lift (see the top of this page) - but it is evidence
no amount of *internal* re-running can supply, because the value is precisely
that the judge is not ours. Contributions of this shape are explicitly eligible
(see [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md), "Issues / PRs").
