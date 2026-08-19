I’ll read the existing panel skills so the position is grounded in how these reviews actually run, then give an independent take on the five design questions.**Position (Grok seat):** treat the panel as a *claim-stress test*, not a salon. Heterogeneity is only valuable if Round 0 is independent, later rounds are claim-scoped, and the orchestrator is forbidden from becoming a third silent vote.

## 1. Round structure

**Default: 2 rounds. Cap at 3. Never 4.**

- **R0 — Isolated findings (parallel, no cross-talk).** Same artifact, same question, no other-model text. This is the only uncontaminated signal. Require ranked claims: `claim / evidence / severity / falsifier`. Cap length (e.g. 6 claims). If they agree here, stop. Agreement *after* seeing each other is cheap.
- **R1 — Targeted rebuttal.** Relay only *disputed* claims, numbered, with the original evidence. Task: concede, refute with counter-evidence, or reframe with a cheaper test. Ban “I largely agree, however…”.
- **R2 — Decision residual (optional).** Only if a ship/kill/architecture call still turns on a factual or causal disagreement. One question: *what would change the call, and can we check it in the repo now?* If the leftover is taste, stop and escalate to a human.

**Stop when:** (a) R0 already agrees, (b) remaining deltas are severity/taste not truth, (c) a claim is conceded or empirically checkable, (d) a round adds no *new* claim IDs. Do not run “until consensus.” Consensus is a failure mode, not a goal.

## 2. When heterogeneity is worth it

Self-critique catches local bugs, missing tests, and style. A Codex+Grok panel is worth the latency only when you need **uncorrelated priors**:
- security/auth boundaries, irreversible migrations, “is this even the right design,” ambiguous product tradeoffs, or anything one lab’s models are trained to be polite about.
- Codex tends conservative/tool-shaped; Grok tends blunt and less agreement-seeking. That *difference* is the product. Persona-prompting one model as both is costume, not diversity.

**Skip the panel when:** the artifact is smaller than the overhead; the question is grep-able; nits/formatting; the orchestrator can verify in one pass; or you already have a failing test. Three frontier CLIs on a 20-line rename is ritual.

**Rule of thumb:** if a wrong answer costs more than ~15 minutes of human time or a production incident, panel. Otherwise one frontier + tests.

## 3. Failure modes and mitigations

| Failure | Mitigation |
|---|---|
| **Sycophancy** (“excellent point”) | Ban praise. Score rounds on *overturned claims*, not eloquence. |
| **Convergence-by-politeness** | Stop-on-stable-dissent. Synthesis must keep ≥1 live objection. |
| **Context poisoning** | Orchestrator may not paraphrase claims. Quote or drop. |
| **Prompt drift** | Frozen system prompt per seat; each round is original brief + numbered claims only, not the full chat. |
| **Verbosity collapse / inflation** | Hard claim cap; reject answers without evidence pointers. |
| **Shared-corpus false consensus** | Prefer repo-checkable claims. Famous-architecture agreement is weak. |
| **Identity theater** | Roles are *incentives* (skeptic vs builder), not “be Grok-like.” |
| **Orchestrator last-word bias** | Verdict template is mechanical; no new arguments in synthesis. |

## 4. Relay and verdict rules

**Relay:**
- **Verbatim** for the claim sentence, evidence quotes, and severity. Summarize *only* non-claim prose, and mark it `ORCH-SUMMARY`.
- **Attributed, never blind.** You need the source to discount known biases (Codex over-indexes on neatness; Grok over-indexes on punch). Blind panels also invite “as another reviewer noted…” hallucinations.
- **No editorial glue.** Do not add “the stronger argument is…”. Do not merge two claims into one.
- **Check before relay.** If a seat cites a file/API, the orchestrator verifies or stamps `UNVERIFIED`. Relaying a false citation is poisoning.

**Verdict:** **adjudication, not consensus.** Output three buckets:
1. **Independent agreement** (same claim in R0, before cross-talk) — high weight.
2. **Surviving dissent** — named, with whose claim and the cheapest check.
3. **Orchestrator call** — one paragraph, mode-tagged (ship / don’t / decide-after-check), *using only bucketed claims*. If R0 conflicted and R1 didn’t resolve, the human decides; the panel must not paper over it.

Consensus language (“the panel believes”) is banned unless bucket 1 is non-empty.

## 5. Trenches tip

**People poison R0 by “helpfully” sharing context.** The usual mistake is one transcript: Claude thinks, then asks Codex, then pastes both into Grok. That is a relay chain, not a panel — Grok is reviewing Claude’s frame.

Spawn seats in parallel with the *raw artifact* and a frozen question. If you must add orchestrator notes, put them in a clearly marked appendix the seats are told to ignore unless a claim needs them. The second most common error is letting R1 see the *whole* rival essay; rebuttal quality collapses into tone-matching. Number the claims and send only those.

Independence first. Cross-talk second, and only on leftovers. The orchestrator is a switchboard, not a chair.
