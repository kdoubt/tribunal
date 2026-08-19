## Position

Treat the panel as a structured dispute, not a group chat. Its value comes from independent error processes and explicit adjudication—not from producing agreeable prose.

## 1. Ideal round structure

Use three rounds by default, with an optional fourth:

1. **Independent analysis**
   Each panelist receives the same frozen task packet and independently produces:
   - conclusion;
   - key assumptions;
   - evidence or reasoning;
   - confidence;
   - likely failure point;
   - what evidence would change its view.

   Do not expose another panelist’s answer yet. This preserves independence.

2. **Targeted rebuttal**
   Relay each answer to the other panelist with a narrow instruction: identify the strongest claim, the most consequential error, any unsupported assumption, and one point that should survive synthesis. Require engagement with claim IDs, not a fresh essay.

3. **Revision under challenge**
   Each panelist submits a revised position and explicitly labels:
   - retained claims;
   - changed claims;
   - unresolved disagreements;
   - confidence changes and why.

4. **Adjudication, only when needed**
   Invoke a tie-break round if disagreement affects the decision and can plausibly be resolved through a specific test, source check, calculation, or critic. Do not add another generic debate round.

Stop when conclusions stabilize, remaining disagreement is decision-irrelevant, or the next round would merely restate positions. Cap ordinary panels at three substantive rounds; endless convergence is usually cosmetic.

## 2. Why heterogeneity helps

Cross-vendor panels are valuable when models differ in training, tools, safety tuning, reasoning style, and characteristic blind spots. The gain is not “more intelligence by voting.” It is a lower chance of correlated failure.

They are especially useful for:

- ambiguous architecture or policy choices;
- high-impact reviews;
- adversarial threat modeling;
- specifications with hidden assumptions;
- questions where plausible but incompatible interpretations exist;
- evaluating outputs from one panelist’s native ecosystem.

They are not worth it for routine transformations, easily tested code, simple retrieval, low-stakes drafting, or tasks dominated by one authoritative source. If correctness can be established cheaply with a compiler, test, schema, calculator, or primary document, use that oracle before convening a debate.

## 3. Failure modes and mitigations

- **Sycophancy:** Later panelists defer to confident language.  
  Require independent Round 1, explicit objections, and confidence before cross-exposure.

- **Convergence by politeness:** Models manufacture consensus without resolving substance.  
  Require a disagreement ledger containing exact claims, decision impact, and resolution status.

- **Context poisoning:** One hallucinated premise becomes shared truth.  
  Preserve provenance; mark every relayed item as user fact, panelist claim, inference, or verified evidence. Never promote repetition into evidence.

- **Prompt drift:** Debate shifts from the user’s question to an interesting side issue.  
  Attach an immutable task statement and decision criteria to every round. Reject responses that do not map back to them.

- **Verbosity collapse:** Context fills with repeated essays, degrading attention.  
  Use stable claim IDs, delta-only revisions, strict word budgets, and a compact state table maintained by the orchestrator.

- **False independence:** Both panelists inherit the orchestrator’s framing bias.  
  Use a neutral shared packet and ask each to list alternative framings before choosing one.

- **Majority illusion:** Two similar answers are treated as proof.  
  Never count votes without checking independence and evidence quality.

- **Authority laundering:** The synthesizer states uncertain panel claims as facts.  
  Preserve uncertainty and cite which claims were verified versus merely agreed upon.

## 4. Relay and verdict rules

Relay decisive passages verbatim, with claim IDs and attribution. Summarize only for compression, and label summaries explicitly; preserve links, qualifications, confidence, and original text for disputed claims. A summary written by the orchestrator must never become the sole object of rebuttal.

Attributed debate is the default because it improves traceability. Use blinded relay when prestige, vendor identity, or prior confidence could bias evaluation; reveal attribution after judgments are recorded.

The final output should be adjudication, not forced consensus. It should contain:

- recommended decision;
- strongest supporting reasons;
- strongest surviving objection;
- resolved and unresolved disagreements;
- confidence and key assumptions;
- verification performed;
- next action or test.

If the evidence does not resolve the dispute, say so and choose using explicit decision criteria rather than pretending agreement.

## 5. Non-obvious practical tip

Do not relay full transcripts every round. Maintain a **canonical claim ledger** outside the panelists’ prose: claim ID, author, status, evidence, rebuttal, confidence, and decision relevance. Send each panelist only the frozen task packet plus the claims they must address.

Most implementations treat context as memory. It is better treated as contested evidence: curated, provenance-tagged, and aggressively deduplicated.
