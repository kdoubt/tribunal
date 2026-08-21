# Blind-order mapping + all judge passes (revealed post-judging)

Judges saw memos as "Memo X" / "Memo Y" with no arm labels. Order was fixed
before each pass and is revealed here for auditability. Pass 2 & 3 use the
**flipped** order vs pass 1, to catch position bias.

## Order per pass

| decision | pass-1 (Claude-1) | pass-2/3 (Claude-2, Grok) |
|----|----|----|
| P1 | X=solo, Y=panel | X=panel, Y=solo |
| P2 | X=panel, Y=solo | X=solo, Y=panel |
| P3 | X=solo, Y=panel | X=panel, Y=solo |

## Judge choices (decoded to arm)

| decision | Claude judge-1 (independent) | Claude judge-2 (independent, flipped) | Grok judge (CONFLICTED - its own work is in the panel memo) |
|----|----|----|----|
| P1 | **panel** (12-11) | **panel** (11-11, chose on discrimination) | solo (11-9) |
| P2 | **solo** (11-10) | **solo** (11-9) | panel (11-10) |
| P3 | **solo** (12-10) | **solo** (12-11) | panel (12-11) |

## Aggregate rubric score per judge (the robust metric)

Summing the /12 memo totals across the three decisions:

| judge | solo | panel | winner (points) | forced-choice |
|----|----|----|----|----|
| Claude pass 1 (independent) | 34 | 32 | **solo** | solo 2 / panel 1 |
| Claude pass 2 (independent, flipped) | 34 | 31 | **solo** | solo 2 / tie 1 |
| gpt-oss-120b (independent 4th vendor) | 32 | 32 | **tie** | solo 1 / panel 1 / tie 1 |
| Grok (CONFLICTED) | 32 | 32 | **tie** | solo 1 / panel 2 |

gpt-oss per-decision (pass-1 order: P1 X=solo Y=panel; P2 X=panel Y=solo; P3 X=solo Y=panel):
P1 → panel (10 vs 11), P2 → solo (10 vs 9), P3 → tie (12 vs 12).

- **Aggregate: the panel never out-scored solo under any judge.** Solo won under
  both independent Claude passes and tied under the conflicted Grok judge.
- The **only** thing that moved was the *forced choice* of the conflicted judge
  (Grok, whose own text is in the panel memos) - and on a tied aggregate.
- So this is **not** "the result flips with the judge." It is: solo ≥ panel on
  points for every judge; a conflicted judge tie-broke toward the panel 2/3.
  Grok is not blindly pro-panel (it forced-chose solo on P1), which keeps its
  read informative but does not make it an independent second judge.
