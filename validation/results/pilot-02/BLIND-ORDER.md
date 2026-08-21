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

- **Independent judge (Claude, both passes): panel 1 / solo 2 - stable under order flip.**
- **Conflicted judge (Grok): solo 1 / panel 2 - inverts every decision.**
- The two judge *vendors* are anti-correlated on all three decisions. Grok is
  not blindly pro-panel (it picked solo on P1), but it is conflicted, so its
  inversion cannot be cleanly attributed to judgment vs. bias.
