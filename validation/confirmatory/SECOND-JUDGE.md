# Second-judge addendum - 2026-09-22

**Designated second-judge supplementary result - not completion of the primary
endpoint.** On 2026-09-22 the designated non-OpenAI second judge
(Alibaba Qwen family, `qwen/qwen3.8-27b`, served by the Groq gateway; temperature 0;
the published inputs, the published blinding record and the verbatim published
rubric prompt; one call per decision) scored the same ten ambiguous decisions
(`c02`, `c11`-`c19`) as the primary judge.

**Forced-choice agreement with the published primary judge: 9 of 10.** The single
divergence is `c11`, where the primary judge recorded `tie` and the second judge
recorded `solo`. Both judges' forced-choice records show **panel 0**.

This **does not** compute, complete or substitute for the pre-registered primary
endpoint - mean `A_panel - A_2seat` paired across all decisions - which
[`RESULTS.md`](RESULTS.md) records was **not measured as pre-registered**, because
`c09` ended in a split to which the pre-registration assigns no numeric value.

It **does not declare lift**: [`PROTOCOL.md`](PROTOCOL.md)'s gate is conjunctive and
was already closed by the primary judge's result, whatever a second judge returns.

It **does not refute lift with new force** either. A second designated scoring of the
same ten-decision comparison is a robustness check. It is not a reason the decision
rule was ever "incomplete", and it is not "decisive in a way internal re-running
cannot be" - both of those were overclaims removed in v1.1.9, and this page does not
reintroduce them.

The README's **"has not demonstrated" / no-lift** position is unchanged by this run.

## Why it was run

`PROTOCOL.md` ("Judge") states that a non-OpenAI second judge "**is the designated
upgrade** and will be added when a gateway key is available". A gateway key became
available. This is that step, executed in the pre-registered form. Until now the
designated second judge had never been run.

## What it means, stated minimally

The ambiguous-arm preference for the solo memo is **not an artefact of the single
judge that produced it**: a judge from a different model family, given the same
inputs and the same rubric, reaches the same forced choice on 9 of 10 decisions and
never prefers the panel. That is the whole of the claim.

## Method

- **Inputs:** `decisions/cNN-brief.md`, `results-raw/cNN/solo-codex.md` and
  `results-raw/cNN/panel.md` - the published files, unmodified.
- **Blinding:** the published order record in
  [`results-raw/AUDIT.md`](results-raw/AUDIT.md) section 1, applied unchanged. No
  re-randomisation.
- **Prompt:** the verbatim rubric prompt published in `results-raw/AUDIT.md`
  section 3, with `$BRIEF`, `$X` and `$Y` substituted and sent as one message per
  decision, matching section 4's description of the original run.
- **Settings:** `temperature: 0`, `max_tokens: 4096`, one call per decision.
- **Endpoint:** `POST https://api.groq.com/openai/v1/chat/completions`. Named for
  reproducibility: this page reports what was *run*, so it pins the model and host,
  unlike [`REPLICATING-THE-JUDGE.md`](REPLICATING-THE-JUDGE.md), which deliberately
  pins neither because a recipe's aliases rot.
- **Procedure:** the published recipe in
  [`REPLICATING-THE-JUDGE.md`](REPLICATING-THE-JUDGE.md), followed as written. It
  worked unmodified; the only friction was a gateway that rejects a default
  programmatic user agent, which is a client detail and not a protocol matter.

Raw per-decision scores, forced choices and the model's verbatim replies are
published in [`results-second-judge/scores.json`](results-second-judge/scores.json),
so the aggregate can be recomputed without re-running the judge.

## Limitations

- **One run, one model, one gateway.** No repeat sampling, so nothing here speaks to
  the second judge's own variance.
- **The rubric ceilings on the solo arm.** See `RESULTS.md` Limitations. The
  aggregate difference between the two judges is driven almost entirely by how
  harshly each marks the panel arm; the comparison arm has very little room to move.
  Read the forced-choice agreement, not the score gap, as the informative quantity.
- **Same ten decisions, same memos.** This is a re-score of one fixed comparison, not
  new evidence about panels in general.
- **Not a model comparison.** Which judge is "better" is not measured here and is
  outside this repository's scope.

## Exploratory, non-designated

A model outside any `PROTOCOL.md`-designated family - TypeSafe's **Jev**
(`jev-1.13.0`, `POST https://api.typesafe.ai/v1/systemone`) - was also run over the
same ten decisions on 2026-09-22, using the same published inputs, blinding record
and rubric criteria.

It satisfies **no pre-registered condition**: `PROTOCOL.md` designates the Meta Llama
and Alibaba Qwen families for the second judge, and Jev is neither. It is therefore
**not a third independent confirmation**, and its scores are deliberately **not
tabulated** beside the two judges - three agreeing judges would otherwise read as
three independent confirmations when only one was designated.

It is named rather than described anonymously for the same reason the second judge's
model and host are pinned above: a disclosure a reader cannot verify is not much of a
disclosure. What is withheld is the weight its numbers would carry, not its identity.
