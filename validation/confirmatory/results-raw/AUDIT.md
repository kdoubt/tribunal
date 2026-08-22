# Ambiguous-arm audit record (blind mapping, judge prompt, reconciliation)

An external re-review of v1.0.2 correctly noted that the raw judge log
([`ambiguous-judge-gptoss.md`](ambiguous-judge-gptoss.md)) scores only "Memo X"
and "Memo Y" - without the X/Y-to-arm mapping, the published "solo 118 vs panel
103" could not be independently verified from the repo. This file publishes the
operator-side run records, recovered verbatim after that review (v1.0.3).

## 1. Blind order record (verbatim, written before judging)

The runner wrote this file *before* invoking the judge on each decision:

```
c02 order: X=panel Y=solo
c11 order: X=solo Y=panel
c12 order: X=panel Y=solo
c13 order: X=solo Y=panel
c14 order: X=panel Y=solo
c15 order: X=solo Y=panel
c16 order: X=panel Y=solo
c17 order: X=solo Y=panel
c18 order: X=panel Y=solo
c19 order: X=solo Y=panel
```

**Deviation, disclosed:** the protocol says order is "randomized per decision";
the runner *alternated* X/Y per decision instead (counterbalanced exactly 5/5).
Alternation is deterministic, not random - but it cannot systematically favor
one arm across the set, and it was recorded before judging.

**Which solo memo:** the solo memo judged was **`solo-codex.md`, fixed, for all
10 decisions** - not the protocol's per-decision higher-scored solo (that pick
would itself have required extra judge passes). This is a deviation from the
pre-registered `A_solo` definition on this arm. Note its direction: a *fixed
single vendor* is a weaker baseline than an ex-post best-of-two, so this
deviation cannot manufacture the no-lift result - the panel lost to a baseline
weaker than the pre-registered one. But it does mean "solo" in the ambiguous
tables means "Codex solo", and Grok's solo memos (published per decision as
`solo-grok.md`) were never judged.

## 2. Decoded scores (mapping applied to the raw judge log)

| decision | X was | Y was | solo score | panel score | forced choice |
|---|---|---|---|---|---|
| c02 | panel | solo | 12 | 11 | solo |
| c11 | solo | panel | 12 | 12 | tie |
| c12 | panel | solo | 12 | 10 | solo |
| c13 | solo | panel | 12 | 10 | solo |
| c14 | panel | solo | 10 | 9 | solo |
| c15 | solo | panel | 12 | 9 | solo |
| c16 | panel | solo | 12 | 11 | solo |
| c17 | solo | panel | 12 | 11 | solo |
| c18 | panel | solo | 12 | 11 | solo |
| c19 | solo | panel | 12 | 9 | solo |
| **total** | | | **118** | **103** | **solo 9 / tie 1 / panel 0** |

This reconciles exactly with the published headline (aggregate 118 vs 103;
forced choice solo 9, tie 1, panel 0). It also means the solo memo was the
higher-scoring memo in every one of the 9 non-tie decisions - which is what the
mapping is needed to verify.

## 3. The judge invocation (verbatim prompt)

Model: `gpt-oss-120b` on a local vLLM server (OpenAI-compatible
`/v1/chat/completions`; endpoint host redacted - it is a LAN address),
`temperature: 0`, `max_tokens: 30000`, one call per decision. The prompt, with
`$BRIEF` = the decision's `decisions/cNN-brief.md`, `$X`/`$Y` = the two memos
per the order record above:

```
You are an impartial senior technical reviewer. Below is an engineering
DECISION BRIEF, then TWO independent decision memos (Memo X and Memo Y)
written by different processes you know nothing about. Judge ONLY the text.
Do not assume either is 'an AI' or 'a panel'.

Score EACH memo 0-3 per criterion (0=absent, 3=excellent):
1. RISKS surfaced - real tradeoffs/failure modes (3 = load-bearing ones)
2. DISCRIMINATION - names a concrete, cheap test/criterion that would actually settle the call (3 = sharp, pre-registerable)
3. HONESTY & DECISIVENESS - commits to a call AND marks genuine uncertainty; penalize BOTH waffling and false confidence
4. FREEDOM FROM false objections / padding (3 = tight)

Output EXACTLY:
Memo X: r1=_ r2=_ r3=_ r4=_ total=_/12 - <one line>
Memo Y: r1=_ r2=_ r3=_ r4=_ total=_/12 - <one line>
CHOICE: X | Y | tie - <one sentence why>

=== BRIEF ===
$BRIEF

=== MEMO X ===
$X

=== MEMO Y ===
$Y
```

## 4. Known-answer arm scoring (for completeness)

The known-answer arm used no judge model. `decision_correct` was applied by the
**operator**, comparing each memo's headline call against the sealed
`correct_call` in `sealed/cNN-truth.md`. The protocol assigned rubric
application to "the judge model + the mechanical oracle, explicitly not the
orchestrator", and no live oracle was invoked (see RESULTS, Limitations) - so
this too was executed short of the pre-registration. The calls being compared
are one-word (YES/NO-style) headlines, so there is little scorer latitude - with
one real exception, c09, whose contestable classification is discussed in
[`../RESULTS.md`](../RESULTS.md).
